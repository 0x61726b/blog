---
layout: post
title: "Compiling across platforms."
modified: 2015-09-02 12:13:18 -0700
tags: [yume,programming,graphics programming,cross platform]
image:
  feature: yume.png
comments: 
share: 
---
##Thoughts on developing YumeEngine
So last week I've started developing my long-awaited-in-mind(is there even a saying like this lol) project.I've read Frank Luna's D3D book and write a framework based on knowledge I've gained.
After that I've read YARE development thesis(its on the source list at the repo) and developed a different framework.Now I'm studying open sources engines to get more ideas about design patterns.
The most engine I studied is OGRE3D.The code is very well commented and readable,easy to understand.Props to the authors! At this time,I've built some required core code to 
understand what platform the engine running.The other times I spent is all learning about CMake and how to pack my projects with CMake for multi-platform.YumeEngine is planned to
support OpenGL,D3D 11 & 12. The code I have now can run on GNU and MSVC and generates a small console window.I'm working at D3D11 rendering structures such as Device,Adapters,RenderTargets,Views(depth stencil etc.).Once 
the rendering structures is done,I will try to implement a Bucket-style rendering algorithm a.k.a. stateless rendering.I've included the paper which I read it about on the repositry readme.
This is for today's update.I'm hoping to finish D3D11 code in 2 days then move on to OpenGL data structures.It should be fun!

###Thougts on Learning CMake

With a little help from Reddit,it took a long time to finally understand how CMake works and how I can use it.I've looked a lot of CMake projects to understand the workflow.
In YumeEngine,CMake part is very basic.Looking at OGRE CMake,its very scary..I only include my source/header files,link them,find DirectX and OpenGL,dynamic(if windows) or shared library,thats all.Ogre's CMake has a lot of packages/utilites,to be honest, I didnt bother trying to understand those.

For example,for constructing Direct3D libraries,files.It first defines all required files

{% highlight ruby pygments %}

# Configure Direct3D11 
set( HEADER_FILES 
	Include/YumeD3D11Required.h
	Include/YumeD3D11Device.h
	Include/YumeD3D11AdapterInfo.h
	Include/YumeD3D11AdapterInfoList.h
	Include/YumeD3D11Adapter.h
	....
	)

set( SOURCE_FILES 
	Src/YumeD3D11Device.cpp
	Src/YumeD3D11AdapterInfo.cpp
	Src/YumeD3D11AdapterInfoList.cpp
	Src/YumeD3D11Adapter.cpp
	...
    )

{% endhighlight %}


{% highlight ruby pygments %}

#Include DirectX headers
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include ${DirectX_INCLUDE_DIR})
link_directories(${DirectX_LIBRARY_DIR})
add_definitions(-D_USRDLL)

#Create the project as a library
add_library(YUME_DIRECT3D11 ${SOURCE_FILES} ${HEADER_FILES})
#Link libraries with the created project,and YUME which is main project.
target_link_libraries(YUME_DIRECT3D11
  YUME
  ${DirectX_LIBRARY}
)
{% endhighlight %}

Then including {% highlight ruby pygments %} ${DirectX_INCLUDE_DIR} {% endhighlight %} , which is defined above the tree here,then it simply links the created "library" 
with the {% highlight ruby pygments %}   ${DirectX_LIBRARY} {% endhighlight %}.
When we build this on Windows,it will create a **YUME_DIRECT3D11** project,which includes DirectX11 libraries,as well as our main library which is **YUME**.

On the top of the tree CMake file,we have some compiler checks,platform specific options.Most of those are copied from Ogre3D's cmake files.
For example,

{% highlight ruby pygments %}
CMAKE_DEPENDENT_OPTION(YUME_BUILD_DIRECT3D11 "Build Direct3D 11" ON "WIN32;DirectX_D3D11_FOUND" OFF)
CMAKE_DEPENDENT_OPTION(YUME_BUILD_DIRECT3D12 "Build Direct3D 12" ON "WIN32;DirectX_D3D12_FOUND" OFF)
CMAKE_DEPENDENT_OPTION(YUME_BUILD_OPENGL "Build OpenGL" ON "OPENGL_FOUND" OFF)
{% endhighlight %}

If we are using CMake GUI we can easily specify dependent options. **WIN32;DirectX_D3D11_FOUND** if we're on Windows and Dx11 is found,it will ask for whether to build it or not.
If we're on Linux,CMake will not even bother looking for Direct3D libraries,so that's nice.

Finally,after fiddling around with dependencies in Ubuntu,I was able to compile the project and execute the sample.Output windows are below!

**On Linux**
<img src="http://i.imgur.com/tkk4yx5.png"/>
**On Win32**
<img src = "http://i.imgur.com/UAkDhNA.png" />
