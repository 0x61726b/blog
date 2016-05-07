---
layout: post
title: "Yume Engine - Deferred Lights using tesellated spheres and stencil test"
modified: 2016-05-04 00:01:26 +0200
tags: [yume,programming,direct3d11,stencil,point lights,lights,deferred lights,light volumes,c++,graphics programming]
image:
  feature: pointlightfeature.jpg
  credit:
  creditlink:
comments:
share:
---

In this post,I am going to talk about how I implemented lights in the deferred pipeline of Yume and provide source code as much as possible.I also used a Micro-facet BRDF algorithm to give the point lights nice specular reflectance.
In a deferred pipeline,we must create a pass for each light to calculate the direct lighting in that pixel.If its a point/spot light we will render a tesellated sphere and use depth-stencil test to find out which pixels will be lit.If its a directional light we will render a full screen triangle.
The first pass is the ordinary gbuffer pass where we output the diffuse and specular albedo as well as normals and linear depth.
In the second pass,

```

for each light
  if Directional
    Set Depth test to LESS
    Set stencil test to EQUAL
    Set blend operation to BLEND_ADD
    Draw full screen triangle
  else if Point
    Set stencil test to EQUAL
    Set Depth test to GREATER_EQUAL
    Set blend operation to BLEND_ADD
    Draw a tesellated sphere

```

Talking about point lights,in the vertex shader we have to transform the volume to accomodate for the lights position in the world.

In the CPU side, we upload a transform matrix such as,

```

float range = light->GetRange();
XMMATRIX volumeTransform = light->GetPosition();
volumeTransform = DirectX::XMMatrixMultiply(DirectX::XMMatrixScaling(range,range,range),volumeTransform);

```

then in the vertex shader,

```

PS_INPUT_POS vs_df(in float3 position : POSITION)
{
  PS_INPUT_POS output;

  float4 worldPos = mul(volume_transform,float4(position,1.0f));
  output.Position = mul(vp,worldPos);

  output.ViewRay = worldPos - camera_pos;
  return output;
}


```

Since we dont output the pixel position in the GBuffer pass, we have to reconstruct the depth when calculating the light. Take a look at the next image, taken from [MJP's Reconstrucing depth article](https://mynameismjp.wordpress.com/2010/09/05/position-from-depth-3/)

![](https://mynameismjp.files.wordpress.com/2010/09/view-space-geo.png)


Using the ViewRay we can reconstruct the depth in the pixel shader like this

```

//Pixel shader of the light volume/quad

float depth = rt_lineardepth.Load(texCoord).x; //Sample the depth from the depth texture. In the vertex shader of the GBuffer pass depth is outputted as VertexOutput.LinearDepth = WorldPos - CameraPos

float3 V = -normalize(input.ViewRay.xyz);
float3 position = camera_pos + (-V * depth); //camera_pos is the world position of the camera


```

Now that we have the pixel position in the light pass we can calculate the direct lighting that pixel will receive. Here is the function that will calculate the point light at given position.

```

float3 BRDFPointLight(in float3 diffuseAlbedo,in float3 normal,in float3 position,in float3 lightColor,float3 lightPosition,float lightRange,in float shininess)
{
		float att = 1.0f;
		float3 L = lightPosition - position;
		float distance = length( L );

		att = max(0, 1.0f - ( distance / lightRange ));

		L /= distance;

		const float PI = 3.14159265f;

		float3 diffuse = lightColor * diffuseAlbedo / PI;
		float3 E = normalize(camera_pos - position); //E
		float3 H = normalize(L + E); //H

		float3 Ks = float3(0.7, 0.7, 0.7);

		float3 D = ((shininess + 2) / (2 * PI)) * (pow(max(0.01, dot(normal, H)), shininess));

		float3 F = Ks + ((1 - Ks) * (pow(1 - (max(0.0, dot(L, H))), 5)));

		float3 G = 1 / pow(max(0.0001, dot(L, H)), 2);

		float3 BRDF = (F * G * D) / 4;

		float3 light = (diffuse + BRDF) * (max(0.0, dot(normal, L)) * (lightColor * diffuseAlbedo * PI));

		return light*att;
}

```


Here are the results,


![](http://i.imgur.com/qsGRNwl.jpg)

![](http://i.imgur.com/2PVBLtN.jpg)

We can see the specular highlight more clearly here,

![](http://i.imgur.com/sJYd1Ak.jpg)

In this scene I had 60 point lights and the performance wasnt hurt that much,even there is no culling of lights that are that visible or dont affect any geometry.
