---
title: Subsurface Scattering in Ray Tracing Pipeline of UE4
date: 2020-12-09 20:48:12
tags:
---



### Ray Tracing管线与Rasterization管线的差异

两种管线SSS实现的主要差异在于`ScreenShadowMaskTexture`的渲染方式不同，在Ray Tracing管线中，使用`RayTracingShadowMaskTexture`的内容写入到`ScreenShadowMaskTexture`中，具体语句在*LightRendering.cpp* 中的`FDeferredShadingSceneRenderer::RenderLights()`中：

```c++
GraphBuilder.QueueTextureExtraction(RayTracingShadowMaskTexture, &ScreenShadowMaskTexture);
```

追踪`RayTracingShadowMaskTexture`，发现其在`RenderRayTracingShadows()`中渲染。在*RayTracingShadows.cpp*中找到`FDeferredShadingSceneRenderer::RenderRayTracingShadows()`，可以在其中找到使用的shader类为`FOcclusionRGS`，进一步找到绑定的shader文件为*RayTracingOcclusionRGS.usf*。且`RWOcclusionMaskUAV`为最终传出的`RayTracingShadowMaskTexture`。

```c++
IMPLEMENT_GLOBAL_SHADER(FOcclusionRGS, "/Engine/Private/RayTracing/RayTracingOcclusionRGS.usf", "OcclusionRGS", SF_RayGen);
...
PassParameters->RWOcclusionMaskUAV = OutShadowMaskUAV;
```

在*RayTracingOcclusionRGS.usf*中，`RWOcclusionMaskUAV`的写入分为3种分支，其中不做降噪的分支如下，关于降噪分支会在后文提及：

```glsl
const float ShadowFadeFraction = 1;
float SSSTransmission = Occlusion.TransmissionDistance;
float FadedShadow = lerp(1.0f, Square(Shadow), ShadowFadeFraction);
float FadedSSSShadow = lerp(1.0f, Square(SSSTransmission), ShadowFadeFraction);

float4 OutColor = EncodeLightAttenuation(half4(FadedShadow, FadedSSSShadow, FadedShadow, FadedSSSShadow));
RWOcclusionMaskUAV[PixelCoord] = OutColor;
```

而在UE4.25版本中，`TransmissionDistance`直接被写为`Visibility`，而`Visibility`则是阴影项`Shadow`的计算结果。这说明光追管线中没有计算SSS着色模型的次表面阴影项，而是暂时使用阴影项来代替，这直接导致了次表面散射效果的缺失。

```glsl
Out.TransmissionDistance = (LocalSamplesPerPixel > 0) ? Out.Visibility / LocalSamplesPerPixel : Out.Visibility;
```

<br>



### Ray Tracing管线的修复

对于基于厚度的SSS算法，Ray Tracing管线相对于Rasterization管线的优势在于可以更加精确的计算出物体上一个像素被自身遮挡的厚度以及光线到达物体表面的辐射量，我们将其分别定义为Thickness和Transmittance，于是我们就着手计算。

```glsl
// The thickness of itself
// "Thickness = -1" stand for it is not shielded by itself, but shielded by others;
// "Thickness = 0" stand for it is on the surface exposed to light directly, not shielded by anything.
float Thickness = 0;
// Transmittance of the object, in the range of 0-1, is calculated with opacity of each object.
// When "Transmittance = 0" light will be completely blocked.
float Transmittance = 1;
```

画一张简单的示意图，以便覆盖所有可能的情况：

![img](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202012/09/175733-521194.png)

可以发现，我们需要拿到的数据有`StartingPointID`、`ClosestHitID`、`Opacity`、`HitT`等。



<br>

#### 扩展Payload

UE4中定义了几种Payload，但是其中其实用到的只有`FMaterialClosestHitPayload`和`FPackedMaterialClosestHitPayload`，两者均继承自最基本的`FMinimalPayload`，后者是前者的打包压缩版，目的是节约Payload的体积，增强光追的性能。`FMaterialClosestHitPayload`的主要内容如下：

```glsl
										// Unpacked  Packed
										// offset    bytes
	// float FMinimalPayload::HitT		// 0         4       32bits
#if USE_RAYTRACED_TEXTURE_RAYCONE_LOD
	FRayCone RayCone;					// 4         8       64bits
#endif // USE_RAYTRACED_TEXTURE_RAYCONE_LOD
	float3 Radiance;					// 12        6       48bits
	float3 WorldNormal;					// 24        6       48bits
	float3 BaseColor;					// 36        6       48bits
	float3 DiffuseColor;				// 48        0       (derived)
	float3 SpecularColor;				// 60        0       (derived)
	float Opacity;						// 72        2       16bits
	float Metallic;						// 76        1       8bits
	float Specular;						// 80        1       8bits
	float Roughness;					// 84        2       16bits
	float Ior;							// 88        2       16bits
	uint ShadingModelID;				// 92        2       4bits
	uint BlendingMode;					// 96        0       4bits (packed with ShadingModelID)
	uint PrimitiveLightingChannelMask;	// 100       0       3bits (packed with ShadingModelID)
	float4 CustomData;					// 104       4       32bits
	float GBufferAO;					// 120       0       (removed)
	float3 IndirectIrradiance;			// 124       0       48bits -- gbuffer only has float payload and there are truncation HLSL warnings
	float3 WorldPos;					// 136       0       (derived)
	uint Flags;							// 148       0       5bits (packed with ShadingModelID)
	float3 WorldTangent;				// 152       6       48bits
	float Anisotropy;					// 164       2       16bits (packed with WorldTangent)
										// 168 total
```

其中并没有`InstanceID`，于是我们需要扩展这个payload，如下：

```glsl
#ifdef USE_INSTANCE_ID
	uint InstanceID;						// 32bits
#endif
```

同时也需要扩展`FPackedMaterialClosestHitPayload`以及打包与解包的函数。

当这些做完之后可能会发现，编译后UE4会出现不明原因Crash，而且再也打不开了，这是为什么呢？

原因在于UE4其实在C++中指定了这个Payload的最大Size，当超过这个Size时，编译Shader，UE4就会直接Crash。

此时在*Engine/Source/Runtime/Renderer/Private/RayTracing*文件夹下的*RayTracingAmbientOcclusion.cpp*、*RayTracingMaterialHitShaders.cpp*、*RayTracingShadows.cpp*、*RaytracingSkylight.cpp*这四个文件中将`// sizeof(FPackedMaterialClosestHitPayload)`注释处的`Initializer.MaxPayloadSizeInBytes`修改为所需的新大小，即可解决这一问题。

**可以发现的很重要的一点是，UE4为不同需求使用了不同的`RayGenerationShader`(`RayTracingDebugMainRGS`/`OcclusionRGS`/`RayTracingPrimaryRaysRGS`)，但是对于不同材质其实都使用的是同一个`ClosestHitShader`，即`MaterialCHS`，同时也都使用同一个Payload，即`FPackedMaterialClosestHitPayload`。**

<br>

#### 在UE4 Ray Tracing Debug View中显示InstanceID

为了验证UE4中每个物体的`InstanceID`不同，同时方便调试，我们先将`InstanceID`加入Ray Tracing Debug View。

首先在*RayTracingDebug.usf*添加一个`case`，根据`InstanceID`的值显示颜色：

```glsl
case RAY_TRACING_DEBUG_VIZ_INSTANCE_ID:
    Result = float4((Payload.InstanceID % 10) / 10.0f, (Payload.InstanceID % 5) / 5.0f, (Payload.InstanceID % 3) / 3.0f, 1.0f);
    break;
```

同时在*Shaders/Shared/RaytracingDebugDefinitions.h*中需要定义`RAY_TRACING_DEBUG_VIZ_INSTANCE_ID`

引擎中需要修改的文件有：

*Runtime/Renderer/Private/RayTracing/RayTracingDebug.cpp*，*Editor/UnrealEd/Private/RayTracingDebugVisualizationMenuCommands.cpp*

此时重新编译UE4，就可以从Ray Tracing Debug View中选择InstanceID，显示效果如下：

![img](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202012/09/181825-496602.png)

<br>



### 关于降噪



<br>

### 厚度计算流程

![image-20211020180843810](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202110/20/180844-88438.png)



<br>