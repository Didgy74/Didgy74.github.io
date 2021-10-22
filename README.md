
# Nils Petter Skålerud, aka "Didgy"

My portfolio. Under construction. Very WIP.


# [Blog posts](blogposts)

I primarily write blog posts about my GUI solution. In these blog posts I will talk about some of my rationale regarding the design of my GUI toolkit, for which I have put a lot of time into. I'll be talking about the basic problems I want the toolkit to solve, as well as the concepts I employ to solve them and how I arrived at these concepts.


# Projects

### DEngine

Keywords: Game engine. Mobile. GUI. Vulkan. Level editor. 3D application.

A game engine project where I experiment with a level editor designed for tablets. In addition to this I focus on desktop support, as well as very high performance code.

Highlights of this include a fully homemade retained-mode GUI toolkit designed for automatic layouting, superior cross-platform portability and equal mobile/desktop support. Another highlight is the Vulkan renderer written from the ground up with the intent of running efficiently on mobile GPUs.  

Repository: [github.com/Didgy74/DEngine](https://github.com/Didgy74/DEngine)

![Screenshot of DEngine](https://raw.githubusercontent.com/Didgy74/Didgy74.github.io/main/DEngine/Screenshot.jpg)

---

### Texas

Keywords: Texture loading. KTX. DDS. PNG. WebP. User-provided memory/resources.

A texture/image loading library for C++ designed to support loading image fileformats designed for 3D applications (i.e KTX or DDS) while also supporting regular web image fileformats (i.e PNG). This library lets you load these types under one file-agnostic interface. This is a niche not supported by existing libraries such as GLM and stb_image.

The longterm goal would be to make this a production-ready library for extracting imagedata from file into memory regardless of fileformat. The library will not do any processing on the imagedata.

This was my first attempt at making a library meant to be used by others, not just me. I've made an effort to write both documentation as well as code examples. The interface is designed to be portable, it works without using C++ exceptions, lets the user provide their own memory for the loader to use, and also lets the user pass in either raw data or a polymorphic stream object. The library also avoids the C++ STL. Furthermore, the library is designed to be quite modular in that it allows you to disable loading-functions that might run slower at the benefit of an easier loading process, You can also turn off support for any fileformats to make the library more lightweight to compile.

Later on I want to start setting up automated build tests as well as comparing outputs up against other established libraries to guarantee correct file-reading. I also want to run benchmarks against these libraries.

Repository: [github.com/Didgy74/Texas](https://github.com/Didgy74/Texas)
