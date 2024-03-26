---
title: Subsurface Scattering in Raster Pipeline of UE4
date: 2020-12-09 16:17:12
tags:
---



### 寻找着色模型入口

我们首先找到UE4的Subsurface Scattering是在哪里实现的，打开RenderDoc，发现是在`StandardDeferredLighting`这一步和延迟渲染的光照一同实现的。

![image-20201109180721972](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/09/180723-456486.png)

于是我们在`Source/Runtime/Renderer`路径下搜索`StandardDeferredLighting`，定位到*LightRendering.cpp*。

在文件中查找`IMPLEMENT_GLOBAL_SHADER`，找到所用到的shader类型为`FDeferredLightPS`，shader文件为`/Engine/Private/DeferredLightPixelShaders.usf`，入口函数为`DeferredLightPixelMain`：

```c++
IMPLEMENT_GLOBAL_SHADER(FDeferredLightPS, "/Engine/Private/DeferredLightPixelShaders.usf", "DeferredLightPixelMain", SF_Pixel);
```

打开`DeferredLightPixelShaders.usf`中的`DeferredLightPixelMain`函数，发现输出为名为`OutColor`的变量，顺着`OutColor`逐个排查，找到`GetDynamicLighting`函数，跳到`DeferredLightingCommon.ush`文件，进一步找到`GetDynamicLightingSplit`函数，计算光照的函数应该是`IntegrateBxDF`：

```glsl
Lighting = IntegrateBxDF( GBuffer, N, V, Capsule, Shadow, LightData.bInverseSquared );
```

随着`IntegrateBxDF`跳入文件`ShadingModels.ush`，发现如下代码：

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

终于找到了所有着色模型的分叉点，这样以后研究着色模型就可以从这里作为入口了。

<br>

### 深入SubsurfaceBxDF

```glsl
FDirectLighting SubsurfaceBxDF( FGBufferData GBuffer, half3 N, half3 V, half3 L, float Falloff, float NoL, FAreaLight AreaLight, FShadowTerms Shadow )
{
	FDirectLighting Lighting = DefaultLitBxDF( GBuffer, N, V, L, Falloff, NoL, AreaLight, Shadow );
	
	float3 SubsurfaceColor = ExtractSubsurfaceColor(GBuffer);
	float Opacity = GBuffer.CustomData.a;

	float3 H = normalize(V + L);

	// to get an effect when you see through the material
	// hard coded pow constant
	float InScatter = pow(saturate(dot(L, -V)), 12) * lerp(3, .1f, Opacity);
	// wrap around lighting, /(PI*2) to be energy consistent (hack do get some view dependnt and light dependent effect)
	// Opacity of 0 gives no normal dependent lighting, Opacity of 1 gives strong normal contribution
	float NormalContribution = saturate(dot(N, H) * Opacity + 1 - Opacity);
	float BackScatter = GBuffer.GBufferAO * NormalContribution / (PI * 2);
	
	// lerp to never exceed 1 (energy conserving)
	Lighting.Transmission = AreaLight.FalloffColor * ( Falloff * lerp(BackScatter, 1, InScatter) ) * SubsurfaceColor;

	return Lighting;
}
```

可以看出，`SubsurfaceBxDF`中首先调用了一遍默认的延迟光照`DefaultLitBxDF`，然后从`GBuffer`中提取出`SubsurfaceColor`和`Opacity`进行进一步的次表面散射计算。

我们可以将`lerp(BackScatter, 1, InScatter)`这一项展开，在极坐标中画出其`Transmission`分布图像，其中$α$为入射角弧度，$o$为不透明度`Opacity`，下图为入射角约为$\frac\pi4$时的`Transmission`分布：

![1  < 2π  0.77  0.5  cos  α  2π  12  —cos θ> 0}  5Π/6  1.5  7TT/6  0.5  0.5 ](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/09/204909-950629.png)

可以看出，其能量大部分分布在透射方向，小部分分布在明暗分界线方向，且不透明度$o$越小，透射能量越大。

<br>

### ShadowTerm&LightAttenuation

计算完BxDF之后，在光照累加时还用到了`Shadow`，这也是影响SSS的一个关键。

```glsl
LightAccumulator_AddSplit( LightAccumulator, Lighting.Diffuse, Lighting.Specular, Lighting.Diffuse, LightColor * LightMask * Shadow.SurfaceShadow, bNeedsSeparateSubsurfaceLightAccumulation );
LightAccumulator_AddSplit( LightAccumulator, Lighting.Transmission, 0.0f, Lighting.Transmission, LightColor * LightMask * Shadow.TransmissionShadow, bNeedsSeparateSubsurfaceLightAccumulation );
```

我们看一下`Shadow`的定义：

```c++
struct FShadowTerms
{
	float	SurfaceShadow;
	float	TransmissionShadow;
	float	TransmissionThickness;
	FHairTransmittanceData HairTransmittance;
};
```

从*DeferredLightingCommon.ush*下的`GetShadowTerms`中可以找到`shadow`来源于`LightAttenuation`：

```glsl
// Remapping the light attenuation buffer (see ShadowRendering.cpp)
// Also fix up the fade between dynamic and static shadows
// to work with plane splits rather than spheres.

float DynamicShadowFraction = DistanceFromCameraFade(GBuffer.Depth, LightData, WorldPosition, View.WorldCameraOrigin);
// For a directional light, fade between static shadowing and the whole scene dynamic shadowing based on distance + per object shadows
Shadow.SurfaceShadow = lerp(LightAttenuation.x, StaticShadowing, DynamicShadowFraction);
// Fade between SSS dynamic shadowing and static shadowing based on distance
Shadow.TransmissionShadow = min(lerp(LightAttenuation.y, StaticShadowing, DynamicShadowFraction), LightAttenuation.w);

Shadow.SurfaceShadow *= LightAttenuation.z;
Shadow.TransmissionShadow *= LightAttenuation.z;

// Need this min or backscattering will leak when in shadow which cast by non perobject shadow(Only for directional light)
Shadow.TransmissionThickness = min(LightAttenuation.y, LightAttenuation.w);
```

而`LightAttenuation`是从`ScreenShadowMaskTexture`中采样得到，我们可以用RenderDoc抓取`ScreenShadowMaskTexture`查看。

当前场景的`ScreenShadowMaskTexture`：

![image-20201113151304160](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/13/151306-420319.png)

当前场景的`ScreenShadowMaskTexture`的`B`和`A`通道均为`1`：

![image-20201113145649320](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/13/145649-895860.png)

`R`通道记录了阴影信息：

![image-20201113152138594](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/13/152140-727687.png)

`G`通道记录了次表面阴影信息：

![image-20201113145752848](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/13/145753-107699.png)

将`RG`通道叠加，可以看到球上隐约绿色的部分即为次表面散射照亮区域：

![image-20201113154837234](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/13/154837-950261.png)

换一个场景更加明显，其中立方体使用的是另一套次表面散射着色模型`SubsurfaceProfile`：

![image-20201209155543621](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202012/09/155545-343561.png)

![image-20201209155523608](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202012/09/155524-978094.png)

<br>

### ScreenShadowMaskTexture写入

从RenderDoc中可以找到`ScreenShadowMaskTexture`写入的地方

![image-20201113160559787](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/13/160559-572628.png)



在*ShadowRendering.cpp*的`FSceneRenderer::RenderShadowProjections()`中：

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

该函数在*LightRendering.cpp*中的`FDeferredShadingSceneRenderer::RenderLights`函数中被每个光源调用一次：

在*ShadowRendering.cpp*的注释中还可以找到不同通道的含义：

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

继续沿着`RenderShadowMask()`找到`FProjectedShadowInfo::RenderProjection()`，其中调用了函数`BindShadowProjectionShaders()`，从名称可以看出在这里绑定了shader，进一步找到绑定的shader的类为`TShadowProjectionPS`，在下面的宏定义中可以找到所用的shader文件为*ShadowProjectionPixelShader.usf*：

```c++
#define IMPLEMENT_SHADOW_PROJECTION_PIXEL_SHADER(Quality,UseFadePlane,UseTransmission, SupportSubPixel) \
    typedef TShadowProjectionPS<Quality, UseFadePlane, false, UseTransmission, SupportSubPixel> FShadowProjectionPS##Quality##UseFadePlane##UseTransmission##SupportSubPixel; \
    IMPLEMENT_SHADER_TYPE(template<>,FShadowProjectionPS##Quality##UseFadePlane##UseTransmission##SupportSubPixel,TEXT("/Engine/Private/ShadowProjectionPixelShader.usf"),TEXT("Main"),SF_Pixel);
```

进入shader文件，找到`OutColor`的来源：

```glsl
float FadedSSSShadow = lerp(1.0f, Square(SSSTransmission), ShadowFadeFraction * PerObjectDistanceFadeFraction);
OutColor = EncodeLightAttenuation(half4(FadedShadow, FadedSSSShadow, FadedShadow, FadedSSSShadow));
```

接下来找到`SHADINGMODELID_SUBSURFACE`分支下次表面分量的来源为`ManualPCF`采样函数：

```glsl
SSSTransmission = ManualPCF(ShadowPosition.xy, Settings);
```

采样的数据来源于`CalculateOcclusion`函数计算出的`Occlusion`，其中计算方式为：

```glsl
float4 Thickness = max(Settings.SceneDepth - ShadowmapDepth, 0);
float4 Occlusion = saturate(FastExp(-Thickness * Settings.DensityMulConstant));

return ShadowmapDepth > .99f ? 1 : Occlusion;
```
其中`Settings.DensityMulConstant`的计算方式为：

```glsl
float Opacity = GBufferData.CustomData.a;
// Derive density from a heuristic using opacity, tweaked for useful falloff ranges and to give a linear depth falloff with opacity
float Density = -.05f * log(1 - min(Opacity, .999f));
...
Settings.DensityMulConstant = Density * ProjectionDepthBiasParameters.w;
```

于是可以画出`Occlusion`项的函数图像：

![SSSTransmission](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202012/09/155704-393859.png)

可以看出当`Opacity`从1下降为0，`Density`随着从无穷大下降至0；而随着`Thickness * Density`的增大，`Occlusion`从1下降至趋近于0。当`Occlusion`为1时表示没有遮蔽，为0时完全在阴影中，这与常识相符。

以上就是光栅管线中SSS相关的部分，接下来会有一篇解决RayTracing管线SSS错误问题的相关文档。


