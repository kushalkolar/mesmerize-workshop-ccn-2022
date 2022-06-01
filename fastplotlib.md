# Plotting lots of stuff very fast

## Vulkan can leverage modern GPUs

https://vulkan.org/
> Wikipedia: “Vulkan is a low-overhead, cross-platform API, open standard for 3D graphics and computing… intended to offer higher performance and more efficient CPU and GPU usage compared to older OpenGL APIs. In addition to its lower CPU usage, Vulkan is designed to allow developers to better distribute work among multiple CPU cores.
https://en.wikipedia.org/wiki/Vulkan

tl;dr: Vulkan is new, fast, efficient, and can leverage modern GPU hardware.

### Vulkan is too low level for scientific plotting

~100 lines of code involving multiple low-level steps just to draw a triangle\
https://github.com/KhronosGroup/Vulkan-Samples/blob/master/samples/api/hello_triangle/hello_triangle.cpp

https://vulkan-tutorial.com/Overview

1. Instance and physical device selection
2. Logical device and queue families
3. Window surface and swap chain
4. Image views and framebuffers
5. Render passes
6. Graphics pipeline
7. Command pools and command buffers
8. Main loop

## The pygfx rendering engine!
https://github.com/pygfx/pygfx

A very new rendering engine that can use Vulkan via WGPU. Abstracts away buffer management, pipelines, and shaders :D 

To draw objects using the `pygfx` rendering engine we define their `Geometry`, `Texture` and `Material`. 
These objects are then placed in a `Scene` and can be viewed using a `Camera` that can be moved with `Controllers` that react to mouse events 
and user interaction.

**This rendering engine can be used to draw stuff for desktop applications AND within the browser using a remote frame buffer (RFB). 
It even works over the internet!! The same exact code works to draw as a desktop application or within a jupyter notebook**

When running within a notebook using [jupyter_rfb](https://github.com/vispy/jupyter_rfb), the rendering engine is non-blocking. 
This means you can continue writing code in your notebook within that iPython kernel to interact with the rendered objects and 
dynamically modify them! You can have multiple canvases and renderers in a notebook and they can interact with each other.


