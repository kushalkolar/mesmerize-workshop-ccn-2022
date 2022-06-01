# Very fast plotting

There will be some examples in this section. Create a new environment and install `fastplotlib` from GitHub to try them out locally in a jupyter notebook:

```bash
git clone https://github.com/kushalkolar/fastplotlib.git
cd fastplotlib
pip install -e .

# try the examples
cd examples
jupyter lab
```

## Vulkan

Vulkan is a new graphics API that can leverage modern GPUs. OpenGL is really starting to show its age and can't take advantange of modern hardware.

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

### Examples

This example shows how you can render an image and update it on each rendering cycle. You can try to run it later if you're interested.

```python
import numpy as np
from wgpu.gui.auto import WgpuCanvas, run
import pygfx as gfx

# create a canvas and renderer
canvas = WgpuCanvas()
renderer = gfx.renderers.WgpuRenderer(canvas)

# create a scene
scene = gfx.Scene()

# create a camera, set position to center of image
camera = gfx.OrthographicCamera(512, 512)
camera.position.y = 256
camera.scale.y = -1
camera.position.x = 256

colormap1 = gfx.cm.plasma

# 512x512 array of random data
rand_img_data = np.random.rand(512, 512).astype(np.float32) * 255

# create an image graphic
# define Geometry with the grid set as a Texture using the random data
img_graphic = gfx.Image(
    gfx.Geometry(grid=gfx.Texture(rand_img_data, dim=2)),
    gfx.ImageBasicMaterial(clim=(0, 255), map=colormap1),
)

scene.add(img_graphic)


def animate():
    # update with new random image
    img_graphic.geometry.grid.data[:] = np.random.rand(512, 512).astype(np.float32) * 255
    img_graphic.geometry.grid.update_range((0, 0, 0), img_graphic.geometry.grid.size)

    renderer.render(scene, camera)
    canvas.request_draw()

canvas.request_draw(animate)
canvas
```

This makes it not to complicated to draw scientific plots, but it get quite cumbersome when you want subplots and customizable interactions.

For example a 2x2 gridplot of image data is ~100 LOC

```python
from wgpu.gui.auto import WgpuCanvas, run
import pygfx as gfx
import numpy as np

canvas = WgpuCanvas()
renderer = gfx.renderers.WgpuRenderer(canvas)

dims = (512, 512)  # image dimensions
# default cam position
center_cam_pos = (256, 256, 0)

# colormaps for each of the 4 images
cmaps = [gfx.cm.inferno, gfx.cm.plasma, gfx.cm.magma, gfx.cm.viridis]

# lists of everything necessary to make this plot
scenes = list()
cameras = list()
images = list()
controllers = list()
cntl_defaults = list()
viewports = list()

for i in range(4):
    # create scene for this subplot
    scene = gfx.Scene()
    scenes.append(scene)

    # create Image WorldObject
    img = gfx.Image(
        gfx.Geometry(
            grid=gfx.Texture(np.random.rand(*dims).astype(np.float32) * 255, dim=2)
        ),
        gfx.ImageBasicMaterial(clim=(0, 255), map=cmaps[i]),
    )

    # add image to list
    images.append(img)
    scene.add(img)

    # create camera, set default position, add to list
    camera = gfx.OrthographicCamera(*dims)
    camera.position.set(*center_cam_pos)
    cameras.append(camera)

    # create viewport for this image
    viewport = gfx.Viewport(renderer)
    viewports.append(viewport)

    # controller for pan & zoom
    controller = gfx.PanZoomController(camera.position.clone())
    controller.add_default_event_handlers(viewport, camera)
    controllers.append(controller)

    # # get the initial controller params so the camera can be reset later
    cntl_defaults.append(controller.save_state())


@renderer.add_event_handler("resize")
def layout(event=None):
    """
    Update the viewports when the canvas is resized
    """
    w, h = renderer.logical_size
    w2, h2 = w / 2, h / 2
    viewports[0].rect = 10, 10, w2, h2
    viewports[1].rect = w / 2 + 5, 10, w2, h2
    viewports[2].rect = 10, h / 2 + 5, w2, h2
    viewports[3].rect = w / 2 + 5, h / 2 + 5, w2, h2


reset_cameras = False


def animate():
    for img in images:
        # create new image data
        img.geometry.grid.data[:] = np.random.rand(*dims).astype(np.float32) * 255
        img.geometry.grid.update_range((0, 0, 0), img.geometry.grid.size)

    global reset_cameras

    # reset the cameras if `reset_camera` is set to True
    if reset_cameras:
        # this should work in the future
        # for camera, image in zip(cameras, images):
        #     camera.show_object(image)

        # until a way to center the controller in the scene is implemented
        for camera, controller, cntrl_default in zip(
            cameras, controllers, cntl_defaults
        ):
            controller.load_state(cntrl_default)
            controller.update_camera(camera)

        reset_cameras = False
    else:
        for camera, controller in zip(cameras, controllers):
            # if not reset, update with the pan & zoom params
            controller.update_camera(camera)

    # render the viewports
    for viewport, s, c in zip(viewports, scenes, cameras):
        viewport.render(s, c)

    renderer.flush()
    canvas.request_draw()


layout()


canvas.request_draw(animate)
canvas
```

## fastplotlib

About a month ago I started writing a *scientific* plotting library using the `pygfx` rendering engine. The goal is to abstract away all the fine details present in a rendering engine, i.e. Geometry, Texture, Cameras, Canvases etc., and provide an easy but flexible API to perform a diverse range of *scientific plotting where performance is the goal*. Note: this is, and never will be, a replacement for plotting libraries such as `matplotlib` or `seaborn` which serve other purposes. I see this as having a huge potential for large-scale imaging analysis, real-time Online closed-loop experiments, and massive insane visualizations combining large-scale imaging, behavior and downstream analysis (this is what I'm using it for right now).

Since it uses the `pygfx`, it works using a RFB in jupyter notebooks!

### Examples

Let's go to: `fastplotlib/examples`

Starting from `simple.ipynb`, this example uses the high level API to draw an image. This abstracts away the canvas, renderer, scene, cameras, geometry, etc. that are defined in `pygfx` (see first `pygfx` example).

```python
from fastplotlib import Plot
import numpy as np

plot = Plot()

data = (np.random.rand(512, 512) * 255).astype(np.float32)
plot.image(data=data,  vmin=0, vmax=255, cmap='viridis')

plot.show()
```

Grid plots are also easy to create, see `gridplot_simple.ipynb`

The examples outline more complex interactions with controllers and cameras.

We are planning to have an unofficial hackathon for `fastplotlib` tomorrow (Saturday)! Location is undecided at the momment. You're also welcome to join if you have questions or ideas on other things from the workshop.
