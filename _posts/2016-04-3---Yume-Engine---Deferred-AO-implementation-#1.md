---
layout: post
title: "Yume Engine - Deferred AO Implementation #1"
modified: 2015-11-23 00:01:26 +0200
tags: [yume,programming]
image:
  feature: Playground_2016-04-03_17-02-17.jpg
  credit: 
  creditlink: 
comments: 
share: 
---

![](http://i.imgur.com/1sa6nNJ.png)


I've finally implemented an AO algorithm on my renderer.I'll post some screenshots and the HLSL shader code.This algorithm is called Scaleable Ambient Obscurance(SAO) presented [here](http://graphics.cs.williams.edu/papers/SAOHPG12/McGuire12SAO.pdf).


GBuffer is not a special one as its constructed as Diffuse,Normals and reversed Depth target.
![](http://i.imgur.com/Xo7hJUT.jpg)

You can see the AO map generated after the Gbuffer pass and applied gaussian blur in 2 pass (vertical and horizontal)

Without blur pass

![](http://i.imgur.com/W3pxhFT.png)


The occlusion fades away with the distance from camera to discard pixels far away so we dont calculate occlusion at infinite depth such as Skybox.

Here is how to reconstruct depth from position or from normal gbuffer texture

```


float3 GetViewPosition(float2 ssPos, float depth, float3 iFrustumSize)
{
    float eye_z = depth * iFrustumSize.z;
    float2 t = ssPos * cProjInfo.xy + cProjInfo.zw;
    return float3(t.x, t.y, 1.0) * eye_z;
}


//Pixel shader
#ifdef NORMAL_MAP
  float4 normalInput = tNormalMap.Sample(sNormalMap,iScreenPos);
  float3 normal = normalInput.rgb * 2.0 - 1.0;
  float3 viewSpaceNormal = mul(normal, cView);
#else
float3 viewPos = GetViewPosition(iScreenPos, depthC, iFrustumSize);
float3 vsN = normalize(cross(ddx(viewPos), ddy(viewPos)));
#endif
```


More screens,

![](http://i.imgur.com/13qCSpd.png)

![](http://i.imgur.com/T3BJmQB.png)

![](http://i.imgur.com/k5jAloe.jpg)

![](http://i.imgur.com/ihqDO3F.jpg)


Now with [depth aware](http://rastergrid.com/blog/2010/09/efficient-gaussian-blur-with-linear-sampling) gaussian blur

![](http://i.imgur.com/ERjYsO1.png)

![](http://i.imgur.com/bpPA8k8.png)



Here is the pixel shader HLSL code.

```
void PS(float2 iTexCoord : TEXCOORD0,
        float2 iScreenPos : TEXCOORD1,
        float3 iFrustumSize : TEXCOORD2,
    out float4 oColor : OUTCOLOR0)
{
  #ifdef SSAO_PASS
  float depthC = Sample2D(DepthBuffer, iScreenPos).r;
 float originDepth = Sample2D(DepthBuffer, iScreenPos).r;
 float3 viewPos = GetViewPosition(iScreenPos, depthC, iFrustumSize);
 float fadeStart = 100.0;
 float fadeEnd = 300.0;
 if (viewPos.z > fadeEnd)
 {
   oColor = float4(1.0, 1.0, 1.0, 1.0);
   return;
 }
 float intensity = clamp((fadeEnd - viewPos.z) / (fadeEnd - fadeStart), 0.0, 1.0) * cIntensityDivR6;
 //float3 randVec = tRandomVecMap.SampleLevel(samRandomVec, iScreenPos, 0.0f);
 //float randomAngle = randVec.x;
 float randomAngle = frac(pow(iScreenPos.x, iScreenPos.y) * 43758.5453) / cGBufferInvSize.x;

#ifdef NORMAL_MAP
  float3 normal = DecodeNormal(Sample2D(NormalMap, iScreenPos));
  float3 vsN = mul(normal, cViewThree);
#else
  float3 vsN = normalize(cross(ddx(viewPos), ddy(viewPos)));
#endif
 float ssDiskRadius = cProjScale2 * cProjScale * cRadius / viewPos.z;
 float sum = 0.0;
 float radius2 = cRadius * cRadius;
 const float epsilon = 0.01;
 for (float i = 0.0; i < NUM_SAMPLES; ++i)
 {
   // Screen-space radius for the tap on a unit disk
   float alpha = (i + 0.5) / NUM_SAMPLES;
   float angle = randomAngle + alpha * (NUM_SPIRAL_TURNS * 6.28);
   float2 ssOffset = floor(alpha * ssDiskRadius * float2(cos(angle), sin(angle)));
   float2 ssP = iScreenPos + ssOffset * cGBufferInvSize;
   float depthP = Sample2D(DepthBuffer, ssP).r;
   float3 vsP = GetViewPosition(ssP, depthP, iFrustumSize);
   float3 v = vsP - viewPos;
   float vv = dot(v, v);
   float vn = dot(v, vsN);
   float f = max(radius2 - vv, 0.0);
   sum += f * f * f * max((vn - cBias) / (epsilon + vv), 0.0);
 }
 float occlusion = max(0.0, 1.0 - sum * intensity * (5.0 / NUM_SAMPLES));
 //occlusion = (clamp(1.0 - (1.0 - occlusion) * 5, 0.0, 1.0) + 0.1f) / (1.0 + 0.1f);
 oColor = float4(occlusion, SmallEncodeDepth(depthC), 1.0);
 #endif

 #ifdef COMBINE
 float occlusion = Sample2D(EmissiveMap, iScreenPos).r;
 float3 color = Sample2D(DiffMap, iScreenPos).rgb * occlusion;
 oColor = float4(color,1.0f);
 #endif

 #ifdef DEBUG_AO
 float occlusion = Sample2D(EmissiveMap, iScreenPos).r;
 oColor = float4(occlusion,occlusion,occlusion,1.0f);
 #endif
}
```

