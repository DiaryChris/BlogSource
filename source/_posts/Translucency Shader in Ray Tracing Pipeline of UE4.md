---
title: Translucency Shader in Ray Tracing Pipeline of UE4
date: 2020-11-10 00:35:12
tags:
---



### Shader入口

首先，指定半透明物体shader文件的入口文件是：

`Renderer/Private/RayTracing/RayTracingTranslucency.cpp`

其中`FDeferredShadingSceneRenderer::RenderRayTracingTranslucency`函数中实例化了一个`GraphBuilder`，并且在调用`RenderRayTracingPrimaryRaysView`函数之后调用了：

```c++
GraphBuilder.Execute();
ResolveSceneColor(RHICmdList);
```

于是分配Shader的逻辑应该在`RenderRayTracingPrimaryRaysView`函数中，于是我们进入所在文件：

`Renderer/Private/RayTracing/RayTracingPrimaryRays.cpp`

```c++
auto RayGenShader = View.ShaderMap->GetShader<FRayTracingPrimaryRaysRGS>(PermutationVector);
...
GraphBuilder.AddPass(...);
```

所以使用的Ray Generation Shader（RGS）是`FRayTracingPrimaryRaysRGS`。

进一步，找到这一语句绑定了`FRayTracingPrimaryRaysRGS`使用哪个HLSL文件：

```c++
IMPLEMENT_GLOBAL_SHADER(FRayTracingPrimaryRaysRGS, "/Engine/Private/RayTracing/RayTracingPrimaryRays.usf", "RayTracingPrimaryRaysRGS", SF_RayGen);
```

于是我们可以在以下路径找到此语句：

`Shaders/Private/RayTracing/RayTracingPrimaryRays.usf`

```glsl
RAY_TRACING_ENTRY_RAYGEN(RayTracingPrimaryRaysRGS)
```

在`Shaders/Private/RayTracing/RayTracingCommon.ush`中找到这个宏定义：

```glsl
#ifndef RAY_TRACING_ENTRY_RAYGEN
#define RAY_TRACING_ENTRY_RAYGEN(name)\
[shader("raygeneration")] void name()
#endif // RAY_TRACING_ENTRY_RAYGEN
```

翻译过来也就是HLSL中将`RayTracingPrimaryRaysRGS`标记为一个RGS的语法：

```glsl
[shader("raygeneration")] void RayTracingPrimaryRaysRGS()
```

于是可以确定`RayTracingPrimaryRaysRGS()`就是Shader入口。

<br>

### RGS主体框架

下面我们正式开始梳理这个RGS中的内容：

首先是准备阶段，先拿到`DispatchThreadId`和发出光线的像素坐标：

```glsl
uint2 DispatchThreadId = DispatchRaysIndex().xy + View.ViewRectMin.xy;
uint2 PixelCoord = GetPixelCoord(DispatchThreadId, UpscaleFactor);
uint LinearIndex = PixelCoord.y * View.BufferSizeAndInvSize.x + PixelCoord.x;
```

然后根据像素坐标初始化`Ray`和`RayCone`：

```glsl
float2 InvBufferSize = View.BufferSizeAndInvSize.zw;
float2 UV = (float2(PixelCoord) + 0.5) * InvBufferSize;
```


```glsl
// Trace rays from camera origin to (Gbuffer - epsilon) to only intersect translucent objects
RayDesc Ray = CreatePrimaryRay(UV);
FRayCone RayCone = (FRayCone)0;
RayCone.SpreadAngle = View.EyeToPixelSpreadAngle;
```

之后就将进入一个循环来进行主要的Ray Tracing计算，我们将在下一段详细分析：

```glsl
for (uint RefractionRayIndex = 0; RefractionRayIndex < MaxRefractionRays; ++RefractionRayIndex)
{
    ...
}
```

最终，经过循环计算之后，累加的`PathRadiance`作为最后的主要输出存入数组：

```glsl
PathRadiance = ClampToHalfFloatRange(PathRadiance);
ColorOutput[DispatchThreadId] = float4(PathRadiance, FinalAlpha);
RayHitDistanceOutput[DispatchThreadId] = HitDistance;
```

也可以从文件开头找到，`ColorOutput`正是可读写的`RWTexture2D`类型：

```glsl
RWTexture2D<float4> ColorOutput;
RWTexture2D<float> RayHitDistanceOutput;
```



<br>

### 循环体分析

RayTracing是一个递归的过程，在每一个界面上进行分叉并进一步递归计算，我们将所有的折射写在左子树，反射写在右子树，形成一个二叉树状的结构。

![Refr ](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/09/005739-3830.png)



然而在GPU上运行的shader中函数并不能进行递归调用，此时可以自己使用栈来实现递归。

但是，UE4中并没有在此实现递归，而是使用循环来代替递归，舍去了对最终结果影响较小的分支，仅实现了主要分支的计算，我猜测其目的是大幅缩减计算量，提高效率。

首先将需要在循环中累加或者分发的数据进行初始化：

```glsl
float3 PathRadiance = 0.0;
float PathThroughput = 1.0;	
```

之后进入循环，我们可以对`PathRadiance`进行追踪，抽出循环的主干：

```glsl
for (uint RefractionRayIndex = 0; RefractionRayIndex < MaxRefractionRays; ++RefractionRayIndex)
{
    ...
    FMaterialClosestHitPayload Payload = TraceRayAndAccumulateResults(..., PathVertexRadiance);
    // Handle no hit condition
    if (Payload.IsMiss())
    {
        ...
        PathRadiance += PathThroughput * GetSkyRadiance(Ray.Direction, LastRoughness);
        ...
    }
    ...
    // Handle surface lighting
    PathRadiance += PathThroughput * PathVertexRadiance * vertexRadianceWeight;
    ...
    // Handle reflection tracing
    if (Payload.Roughness < TranslucencyMaxRoughness)
    {
        ReflectionPayload = TraceRayAndAccumulateResults(..., ReflectionRadiance)
	    if (ReflectionPayload.IsMiss())
        {
            ReflectionRadiance = GetSkyRadiance(ReflectionRay.Direction, LastRoughness);
        }
        ...
        PathRadiance += PathThroughput * ReflectionThroughput * ReflectionRadiance * vertexRadianceWeight;
        ...
    }
    ...
    // Set refraction ray for next iteration
    Ray.Origin = HitPoint;
    Ray.TMin = 0.01;
    Ray.TMax = NextMaxRayDistance;
    Ray.Direction = RefractedDirection;
    RayCone = PropagateRayCone(RayCone, SurfaceCurvature, Depth);
}
```

在一个RGS中，最关键的要点是`TraceRay`函数的调用，这里`TraceRay`被封装了几层在`TraceRayAndAccumulateResults`函数中，可以沿着`RayTracingLightingCommon.ush`和`RayTracingCommon.ush`这两个文件中找到最终还是调用了`TraceRay`。

于是我们可以总结出如下图中的结构，其中第一个TraceRay是指从摄像机向场景中发射的第一条光线，每一笔高亮表示一轮循环中计算的内容。

![image-20201109010846740](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/09/010847-244787.png)

<br>

### 反射-透射分支补全

由上图可以看出，UE4中极大地省略了树右侧分支的递归，而仅仅追踪了最左路的透射部分内容。于是，由于右子树反射后续分支的内容的缺失，会出现下图中的错误，即对于一个半透明材质的镜子，第一次的反射光线遇到另一个半透明物体时不再进行后续的追踪，仅计算其本身的漫反射，导致显示为黑色。

![image-20201107210256821](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/07/210258-83838.png)

那么如何修复这一问题呢？

我们有两种方案：

1. 使用栈在GPU中实现递归，将整个树的内容全部实现。
2. 继续沿用UE4的思路，用循环中添加子循环来补充实现一部分的树。

其中，方案1肯定更加接近真实世界结果，然而这却以效率作为代价，因为发出的光线数以指数级别增长。同时，有很多多次折射或反射的光线对最终结果贡献不大。由于实时光追对效率要求很严苛，所以我们最终选择做一个trade-off，使用方案2来节约计算量。

对于半透明物体的在镜子中的渲染而言，现在最缺乏的分支其实是反射后的折射分支，以及之后的多次折射分支。

于是我们在一个循环的内部添加一个函数`TraceRayForRecursiveRefractionFromFirstReflection`专门处理这些分支：

```glsl
TraceRayForRecursiveRefractionFromFirstReflection(ReflectionPayload, 
    ReflectionRay, 
    RandSequence, 
    PixelCoord, 
    RayCone, 
    AccumulatedOpacity, 
    PathThroughput * ReflectionThroughput,
    PathRadiance);
```



函数接受反射分支的光线及其`Payload`，然后分别向反射方向和折射方向发出光线计算结果。函数中包含一个子循环，沿着折射方向不断递归计算下去直到指定次数。这样一来就实现了下图中的部分，其中红色为我们补充实现的部分，绿圈为函数调用处。

![image-20201110002141890](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/10/002148-850948.png)

<br>

### 全反射BUG修复

实现了以上的函数后，我们发现镜子里得到了正确的结果，而镜子外半透明物体的亮度明显过亮，这又是什么原因呢？

经过排查后发现，这是UE4原有的另一个BUG导致的错误。

不难注意到，UE4原有代码是在一个循环中计算表面漫反射与反射，然后在下一个循环才计算折射分支内容，然而在每轮循环末尾计算折射方向时有这么一段代码：

```glsl
float3 T = refract(Ray.Direction, N, Eta);

if (any(T) > 0.0)
{
    RefractedDirection = T;
    PathThroughput *= 1.0 - Fr;
}
// Handle total internal reflection
else
{
    RefractedDirection = reflect(Ray.Direction, N);
}
```

这是在判断全反射，当发生全反射时`refract`函数会返回一个零向量，由于此时不发生折射，折射方向会修改为反射方向，即在下一个循环中进行全反射分支的计算。全反射的物理原理详见高中物理，而我们也可以理解为当发生全反射时，折射分量与反射分量能量合并，均流向了反射方向，全反射可以理解为一种特殊的折射。UE4这种将折射方向改为反射方向的写法并没有什么不妥，但是他们忽略了在当前循环已经计算过一遍反射分支，而下一轮循环中再计算全反射分支，会导致反射分支的能量被重复计算！这就是我们补全反射-透射分支后出现过亮现象的原因。

可能有同学会问，为什么补全分支之前亮度看上去是正常的呢？很简单，这是因为UE4没有计算反射-透射分支导致的反射分支偏暗的BUG正好使得全反射时反射分支重复计算的BUG造成的影响很弱，等于两个BUG的影响相互抵消，正好得到了看似正确的结果。

于是我们将全反射判断从循环末尾处提前到计算反射分支之前，如果发生了全反射，那么就不发射反射光线，避免重复计算。

```glsl
float3 T = refract(Ray.Direction, N, Eta);
//if total internal reflection happened, it should not dispatch reflection ray.
if (Payload.Roughness < TranslucencyMaxRoughness && any(T) > 0.0)
{
    // Trace reflection ray 
    ...
}
```

于是得到更加正确的结果：

![image-20201107210316403](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/07/210316-842028.png)

<br>

### 能量守恒分析

不难发现，在不断发射分支光线时，一定要保证能量守恒，才能得到正确的结果。UE4是怎么保证能量守恒的呢？

这时我们就需要关注上文中循环前定义的`PathThroughput`变量，其主要通过计算菲涅尔现象中的菲涅尔系数`Fr`和表面不透明度`Opacity`来控制光追过程中的能量守恒。

通过上文中的图可以看出，UE4中的实时光追将半透明表面上的能量分为三部分计算：表面漫反射分量、折射分量、反射分量（镜面反射），其中折射分量进行递归。当然这是一种近似方法，并不是完全正确的PathTracing方法，也是为了性能而做出的一种trade-off。

![image-20201110002303632](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/10/002304-521734.png)

我们可以从代码中提取出对于`PathThroughput`的改变：

```glsl

for (uint RefractionRayIndex = 0; RefractionRayIndex < MaxRefractionRays; ++RefractionRayIndex)
{
    ...
    // Handle surface lighting
    PathRadiance += PathThroughput * PathVertexRadiance * vertexRadianceWeight;
    ...
    // Handle reflection tracing
    if (Payload.Roughness < TranslucencyMaxRoughness)
    {
        ...
        PathRadiance += PathThroughput * ReflectionThroughput * ReflectionRadiance * vertexRadianceWeight;
        ...
    }
    ...
    // Handle refraction through the surface.
    float PathVertexTransmittance = Payload.BlendingMode == RAY_TRACING_BLEND_MODE_ADDITIVE ? 1.0 : 1.0 - Payload.Opacity;
    PathThroughput *= PathVertexTransmittance;
    if (PathThroughput <= 0.0)
    {
        break;
    }
    ...
    if (TranslucencyRefraction)
    {
        ...
        float Fr = FresnelDielectric(Eta, NoV, NoT);
        float3 T = refract(Ray.Direction, N, Eta);
        if (any(T) > 0.0)
        {
            RefractedDirection = T;
            PathThroughput *= 1.0 - Fr;
        }
        ...
    }
}
```

可以发现对于`PathThroughput`的改变主要集中在循环末尾计算下一轮折射分支时，这是因为`PathThroughput`是每个循环的公用变量，而循环是在模拟折射方向的递归，所以不应让反射方向的计算改变`PathThroughput`的数值。计算反射方向的Throughput时，其实是单独计算了`ReflectionThroughput`并乘上`PathThroughput`而得到：

```glsl
float NoV = saturate(dot(-Ray.Direction, Payload.WorldNormal));
const float3 ReflectionThroughput = EnvBRDF(Payload.SpecularColor, Payload.Roughness, NoV);

PathRadiance += PathThroughput * ReflectionThroughput * ReflectionRadiance * vertexRadianceWeight;
```

其中`EnvBRDF`函数跳进去其实是进行了菲涅尔相关的计算，约等于计算出了一个`Fr`。

我们还可以发现，每次累加`PathRadiance`时都有一个`vertexRadianceWeight`参与计算，在代码中发现其实就是表面不透明度`Opacity`：

```glsl
float vertexRadianceWeight = Payload.Opacity;
```

于是我们可以梳理出如下两张图，分别是`Fr`和`Opacity`对光线throughput的影响：

![image-20201110002340482](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/10/002341-192556.png)

![image-20201110002413670](https://raw.githubusercontent.com/DiaryChris/typora-image/master/typora202011/10/002414-872720.png)



欢迎交流！





