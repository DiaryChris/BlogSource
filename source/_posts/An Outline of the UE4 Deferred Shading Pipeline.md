---
title: An Outline of the UE4 Deferred Shading Pipeline
date: 2020-11-16 00:38:12
tags:
---



这几天研究了UE4的延迟渲染管线，一直进度很慢，最终发现了一个很清晰的系列文章，所以依据这一系列文章的思路总结一篇**UE4延迟渲染管线概览**。

参考文章：https://medium.com/@lordned

此文章基于UE4的较老版本，所以本文会基于4.25版本的源码对于一些改动进行修改。

<br>

------

## Introduction

UE4中存在三种主要的渲染管线，分别为*deferred shading pipeline*，*forward shading pipeline*和用于移动端的*tile-based deferred rendering pipeline*。

本文只涵盖*deferred shading pipeline*

<br>

------

## Shaders and Vertex Data

### Shaders

首先，`FShader`是所有Shader的基类。

UE4主要有2种Shader，`FGlobalShader`和`FMaterialShader`。其中`FGlobalShader`仅存在一个全局着色器实例，这意味每个实例不能拥有各自的参数，只具有全局参数。`FMaterialShader`是所有需要材质参数的Shader的基类，允许`SetParameters`函数从C++侧改变HLSL中的材质参数。

其中值得注意的是继承自`FMaterialShader`的`FMeshMaterialShader`，它允许在渲染每个Mesh前设定材质参数，是所有需要材质和顶点工厂参数的着色器的基类。

将C++中的Shader类与HLSL中的Function进行绑定的语句为：

```c++
IMPLEMENT_MATERIAL_SHADER_TYPE(TemplatePrefix, ShaderClass, SourceFilename, FunctionName, Frequency)
```
或者包装过`IMPLEMENT_SHADER_TYPE`的这种语句：

```c++
IMPLEMENT_GLOBAL_SHADER(ShaderClass,SourceFilename,FunctionName,Frequency)
```

其中`Frequency`参数从Vertex, Hull, Domain, Geometry, Pixel, Compute中指定了着色器的类型。

例如：

```c++
IMPLEMENT_MATERIAL_SHADER_TYPE(,FDepthOnlyPS,TEXT(“/Engine/Private/DepthOnlyPixelShader.usf”),TEXT(“Main”),SF_Pixel);
```

<br>

### Caching and Compilation Environments

修改材质时，UE4将自动为Shader编译许多种可能的permutation。

此时`ShouldCompilePermutation`函数可以用于指定Permutation是否被编译。

而`ModifyCompilationEnvironment`函数则用于在编译着色器之前修改HLSL中的预处理定义。

<br>

### Vertex Factory

Vertex Factory封装顶点源数据，并且传递到到顶点着色器中。

首先，UE4使用`FPrimitiveSceneProxy`来指定Mesh所使用的Vertex Factory。`FPrimitiveSceneProxy`类似一个渲染线程版本的`UPrimitiveComponent`，由于UE4中的游戏线程和渲染线程数据不互通，所以使用`FPrimitiveSceneProxy`与`UPrimitiveComponent`连接来使渲染线程获取游戏数据。`FPrimitiveSceneProxy`可以在适当的时间查询游戏线程，并将数据从游戏线程获取到渲染线程上，以便可以对其进行处理并将其放置于GPU。

接下来将C++中的Vertex Factory与特定HLSL文件相绑定，语法为：

```c++
IMPLEMENT_VERTEX_FACTORY_TYPE(FactoryClass, ShaderFilename, bUsedWithmaterials, bSupportsStaticLighting, bSupportsDynamicLighting, bPrecisePrevWorldPos, bSupportsPositionOnly)
```

例如：

```c++
IMPLEMENT_VERTEX_FACTORY_TYPE(FLocalVertexFactory,”/Engine/Private/LocalVertexFactory.ush”,true,true,true,true,true);
```

之后，在使用这些顶点数据时，顶点着色器（VS）中均接受统一的`FVertexFactoryInput`结构体作为输入，而在各个*...VertexFactory.ush*文件中，其对`FVertexFactoryInput`定义各不相同，UE4中通过`include`不同的*...VertexFactory.ush*来达成对VS的不同输入。

<br>

------

## Rendering Dependency Graph

在原文中，UE4使用Drawing Policy来为绘制指定正确的着色器Permutation，使用Drawing Policy Factory来创建Drawing Policy并将其添加到适当的列表中。最后，通过一个很长的继承链`FDeferredShadingRenderer::Render`循环遍历各种列表并调用它们的绘制函数。

但是，在4.23版本之后，UE4逐渐采用Rendering Dependency Graph(RDG)，或称Render Graph来代替这个流程。

关于RDG，可以参见：https://docs.unrealengine.com/en-US/Programming/Rendering/RenderDependencyGraph/index.html

渲染依赖图，是一个基于图的调度系统，旨在执行渲染管线的整帧优化，利用DirectX12等现代API的优势，通过使用自动的异步计算调度以及更有效的内存管理来提高性能。大体思路就是构建一个渲染表，最后执行图表中的渲染逻辑。

主要的两个类是`FRDGBuilder`和`FRDGResource`，分别负责构建Render Graph和派生Render Graph中的资源。

当需要在RDG最终添加渲染逻辑时，通过`GraphBuilder.AddPass`传入一个硬编码的Lambda函数来实现，同时传入的还有Pass名称/参数结构体/Pass类型。其中`GraphBuilder`是`FRDGBuilder`的一个实例。

```c++
template<typename ParameterStructType, typename ExecuteLambdaType>
void AddPass
(
    FRDGEventName && Name,
    ParameterStructType * ParameterStruct,
    ERDGPassFlags Flags,
    ExecuteLambdaType && ExecuteLambda
)
```

最后调用`FRDGBuilder::Execute()`，来执行整个RDG

```
GraphBuilder.Execute();
```

<br>

------

## The Deferred Shading Pipeline

### The Deferred Shading Base Pass

之后就是延迟渲染管线的重点GPU部分，首先是Base Pass，也就是渲染到GBuffer的部分。

#### Base Pass Vertex Shader

UE4为减少代码量，使用同一个顶点着色器入口来处理多种不同的`FVertexFactoryInput`，此时，同时`include`多个*...VertexFactory.ush*显然并不能达到想要的效果，于是UE4采用了动态指定的方式。

在*BasePassVertexCommon.ush*中有一句：

```glsl
#include "/Engine/Generated/VertexFactory.ush"
```

当编译着色器时，会将其设置为正确的Vertex Factory，以使引擎知道要使用`FVertexFactoryInput`的哪种实现。

然而对于不同的Vertex Factory，需要不同的VS进行处理，此时*BasePassVertexShader.usf* 中的处理方式是调用`GetVertexFactoryIntermediates`，`VertexFactoryGetWorldPosition`，`GetMaterialVertexParameters`这些分别在不同*...VertexFactory.ush*中定义的方法，巧妙地解决了这一问题。

接下来对于VS的输出，由于管线中可能具有或不具有Tessellation阶段，所以需要不同的输出。在*BasePassVertexCommon.ush*中可以看到，UE4使用`#define`预编译时更改`FBasePassVSOutput`的含义，可以将其定义为简单的`FBasePassVSToPS`结构，也可以将其定义为`FBasePassVSToDS`供Tessellation阶段使用。

这样一来，等于UE4将不同的VS集中在一起共用了同一个入口，即*BasePassVertexShader.usf* 中的`Main`函数。

#### Base Pass Pixel Shader

在UE4中写自定义Shader通常是使用Material Graph连节点图，那么UE4内部需要首先将节点图翻译为HLSL代码。

在*MaterialTemplate.ush*中我们可以发现`FPixelMaterialInputs`结构体和很多函数体中都只有一个`%s`，这些就是字符串占位符，UE4会根据Material Graph将其替换为翻译后的代码。

Base Pass像素着色器（PS）的主入口在 *BasePassPixelShader.usf* 中的`FPixelShaderInOut_MainPS`函数，其中有以下一段：

```glsl
// Store the results in local variables and reuse instead of calling the functions multiple times.
half3 BaseColor = GetMaterialBaseColor(PixelMaterialInputs);
half  Metallic = GetMaterialMetallic(PixelMaterialInputs);
half  Specular = GetMaterialSpecular(PixelMaterialInputs);

float MaterialAO = GetMaterialAmbientOcclusion(PixelMaterialInputs);
float Roughness = GetMaterialRoughness(PixelMaterialInputs);
float Anisotropy = GetMaterialAnisotropy(PixelMaterialInputs);
uint ShadingModel = GetMaterialShadingModel(PixelMaterialInputs);

half Opacity = GetMaterialOpacity(PixelMaterialInputs);
```

其中`GetMaterialBaseColor`这些函数在*MaterialTemplate.ush*中定义，用于拿到在Material Graph中输出的信息。

然后在这个PS中会根据不同的Shading Model所需的数据做一些特殊的计算，例如SubsurfaceColor；如果启用了DBuffer Decal，那么也会对GBuffer数据做一些相关修改。

之后新建一个`FGBufferData`用于存入所有GBuffer信息：

```glsl
FGBufferData GBuffer = (FGBufferData)0; 
```

然后调用定义在*ShadingModelMaterials.ush*中的`SetGBufferForShadingModel`函数，其中全是一些根据不同Shading Model对所需`GBuffer.CustomData`所做的写入。另外有一个很重要的点是`GBuffer.ShadingModelID`是在这个函数中最终确定，因为其中有一些可能会修改最终Shading Model的分支判断，所以传入的Shading Model可能与写入GBuffer中的不同：

```glsl
// Use GBuffer.ShadingModelID after SetGBufferForShadingModel(..) because the ShadingModel input might not be the same as the output
SetGBufferForShadingModel(
	GBuffer,
	MaterialParameters,
	Opacity,
	BaseColor,
	Metallic,
	Specular,
	Roughness,
	Anisotropy,
	SubsurfaceColor,
	SubsurfaceProfile,
	GBufferDither,
	ShadingModel
	);
```

这里有一个需要注意的点是，如果你需要自定义Shading Model并且存储`GBuffer.CustomData`，需要在*BasePassCommon.ush* 中修改这一语句，添加你的自定义Shading Model：

```glsl
#define WRITES_CUSTOMDATA_TO_GBUFFER		(USES_GBUFFER && (MATERIAL_SHADINGMODEL_SUBSURFACE || MATERIAL_SHADINGMODEL_PREINTEGRATED_SKIN || MATERIAL_SHADINGMODEL_SUBSURFACE_PROFILE || MATERIAL_SHADINGMODEL_CLEAR_COAT || MATERIAL_SHADINGMODEL_TWOSIDED_FOLIAGE || MATERIAL_SHADINGMODEL_HAIR || MATERIAL_SHADINGMODEL_CLOTH || MATERIAL_SHADINGMODEL_EYE))
```

我们回到主PS中，最终调用了*DeferredShadingCommon.ush*中定义的`EncodeGBuffer`根据不同Shading Model对于GBuffer数据进行了编码：

```glsl
EncodeGBuffer(GBuffer, Out.MRT[1], Out.MRT[2], Out.MRT[3], OutGBufferD, OutGBufferE, OutGBufferF, OutVelocity, QuantizationBias);
```

最终输出了A-F与Velocity共7张GBuffer textures，Base Pass目标达成。

<br>

### Deferred Light Pixel Shader

UE4接下来将对像素进行光照计算，计算分为3个阶段，分别为Non shadow-casting lights，Indirect lighting，Shadow casting lights。

对于每个光源，UE4都会根据GBuffer与像素深度计算一个`ScreenShadowMaskTexture`，用于在屏幕空间表示场景中在其阴影中的像素。

光照计算的主入口在*DeferredLightPixelShaders.usf*中的`DeferredLightPixelMain`函数，其中计算光照的主要函数为`GetDynamicLighting`：

```glsl
const float4 Radiance = GetDynamicLighting(DerivedParams.WorldPosition, DerivedParams.CameraVector, ScreenSpaceData.GBuffer, ScreenSpaceData.AmbientOcclusion, ScreenSpaceData.GBuffer.ShadingModelID, LightData, GetPerPixelLightAttenuation(InputParams.ScreenUV), Dither, uint2(InputParams.PixelPos), RectTexture, SurfaceShadow);
```

#### Get Dynamic Lighting

我们可以在*DeferredLightingCommon.ush*中找到`GetDynamicLighting`和`GetDynamicLightingSplit`函数，其中前者调用了后者，分别计算了光照的Diffuse分量和Specular分量并且加和。

在`GetDynamicLightingSplit`函数中我们可以发现调用了`GetShadowTerms`，`IntegrateBxDF`和`LightAccumulator_AddSplit`这三个主要函数。其中`IntegrateBxDF`在*ShadingModels.ush*中定义，有一个大分支，用于计算不同Shading Model的光照。

```glsl
FDirectLighting IntegrateBxDF( FGBufferData GBuffer, half3 N, half3 V, half3 L, float Falloff, float NoL, FAreaLight AreaLight, FShadowTerms Shadow )
{
	switch( GBuffer.ShadingModelID )
	{
		case SHADINGMODELID_DEFAULT_LIT:
		case SHADINGMODELID_SINGLELAYERWATER:
		case SHADINGMODELID_THIN_TRANSLUCENT:
			return DefaultLitBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_SUBSURFACE:
			return SubsurfaceBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_PREINTEGRATED_SKIN:
			return PreintegratedSkinBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_CLEAR_COAT:
			return ClearCoatBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_SUBSURFACE_PROFILE:
			return SubsurfaceProfileBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_TWOSIDED_FOLIAGE:
			return TwoSidedBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_HAIR:
			return HairBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_CLOTH:
			return ClothBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		case SHADINGMODELID_EYE:
			return EyeBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
		default:
			return (FDirectLighting)0;
	}
}
```

其中需要传入的`Shadow`是由之前调用的`GetShadowTerms`得到：

```glsl
FShadowTerms Shadow;
Shadow.SurfaceShadow = AmbientOcclusion;
Shadow.TransmissionShadow = 1;
Shadow.TransmissionThickness = 1;
GetShadowTerms(GBuffer, LightData, WorldPosition, L, LightAttenuation, Dither, Shadow);	
```

可以看出`SurfaceShadow`和`TransmissionShadow`分量分别被初始化为`AmbientOcclusion`和`1`，即像素不在阴影中时值为`1`，如果在阴影中，则该值将小于`1`。

在`GetShadowTerms`中，`LightAttenuation`被读取用于计算，对其进行追踪就会发现，其来源于*ShadowRendering.cpp*中渲染的`ScreenShadowMaskTexture`，可以在`FSceneRenderer::RenderShadowProjections`函数中找到写入`ScreenShadowMaskTexture`的`RenderPass`：

```c++
// Normal deferred shadows render to the shadow mask
FRHIRenderPassInfo RPInfo(ScreenShadowMaskTexture->GetRenderTargetItem().TargetableTexture, ERenderTargetActions::Load_Store);
...

TransitionRenderPassTargets(RHICmdList, RPInfo);
RHICmdList.BeginRenderPass(RPInfo, TEXT("RenderShadowProjection"));
RenderShadowMask(nullptr);
RHICmdList.SetScissorRect(false, 0, 0, 0, 0);
RHICmdList.EndRenderPass();
```

该函数在*LightRendering.cpp*中的`FDeferredShadingSceneRenderer::RenderLights`函数中被循环调用：

```c++
RenderShadowProjections(RHICmdList, &LightSceneInfo, ScreenShadowMaskTexture, ScreenShadowMaskSubPixelTexture, HairDatas, bInjectedTranslucentVolume);
```

对于每个光源，都会渲染一张`ScreenShadowMaskTexture`，其中方向光渲染出的图使用`RG`通道，Point Light和Spot Light渲染出的图使用`BA`通道。

*ShadowRendering.cpp*的注释中也解释了`ScreenShadowMaskTexture`各个通道的含义：

```c++
// Light Attenuation channel assignment:
//  R:     WholeSceneShadows, non SSS
//  G:     WholeSceneShadows,     SSS
//  B: non WholeSceneShadows, non SSS
//  A: non WholeSceneShadows,     SSS
//
// SSS: SubsurfaceScattering materials
// non SSS: shadow for opaque materials
// WholeSceneShadows: directional light CSM
// non WholeSceneShadows: spotlight, per object shadows, translucency lighting, omni-directional lights
```

`GetShadowTerms`调用`DistanceFromCameraFade`之后将其与静态阴影混合并存入`Shadow`的各个分量，这一步被叫做Remapping the light attenuation buffer。

另外UE4计算了Radial Light的的光能衰减，保存在`LightMask`中。

最后定义在*LightAccumulator.ush*中的`LightAccumulator_AddSplit`被调用，可以看到用到了`Shadow`和`LightMask`：

```glsl
LightAccumulator_AddSplit( LightAccumulator, Lighting.Diffuse, Lighting.Specular, Lighting.Diffuse, LightColor * LightMask * Shadow.SurfaceShadow, bNeedsSeparateSubsurfaceLightAccumulation );
LightAccumulator_AddSplit( LightAccumulator, Lighting.Transmission, 0.0f, Lighting.Transmission, LightColor * LightMask * Shadow.TransmissionShadow, bNeedsSeparateSubsurfaceLightAccumulation );
```

可以看出，两次调用分别是计算Surface和Subsurface的光照。

最后`GetDynamicLightingSplit`返回光照累加的结果：

```glsl
return LightAccumulator_GetResultSplit(LightAccumulator);
```

#### IES light profile

我们回到`DeferredLightPixelMain`，可以发现我们得到的`Radiance`之后乘了个`Attenuation`：

```
OutColor += (Radiance * Attenuation) * OpaqueVisibility;
```

这里的`Attenuation`其实和之前的`LightAttenuation`不同，是由`ComputeLightProfileMultiplier`函数计算得到，是为了考虑IES light profile对于光照的影响。

关于IES light profile，参见：https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/IESLightProfiles/index.html

<br>

### ResolveSceneColor

由于`DeferredLightPixelMain`会对影响对象的每一个光源运行，所以UE4会累积该光照结果并将其存储在Buffer中，这个Buffer在多步之后的`ResolveSceneColor`中才被绘制。

<br>

