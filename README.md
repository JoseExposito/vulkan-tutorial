# Vulkan tutorial

## About

I'm learning Vulkan following the tutorial available at
https://vulkan-tutorial.com/.

This repo contains the code I write while learning and this README file is a
little summary about the concepts learned.

## 01 - Hello triangle

### Vulkan instance

The Vulkan instance is the connection between the app and the Vulkan API.

In order to create it we need:

 - VkApplicationInfo: Contains information about the app such as the name,
   version, API version, etc.
 - VkInstanceCreateInfo: Contains information about the extensions used
   (instance extension, devices have their own extensions), the validation
   layers and a pointer to VkApplicationInfo.
 - vkCreateInstance(): This API creates the VkInstance handle with the provided
   VkInstanceCreateInfo.

### Validation layers

For performance reasons, Vulkan performs limited error checking. However,
additional checks such as parameter validation, memory leaks, thread safety, etc
can be enabled through validation layers.

Validation layers are set in the VkInstanceCreateInfo structure.

### Physical device

A VkPhysicalDevice is a data type to represent each hardware component available
in the system, usually GPUs.

Physical device capabilities can be queried in order to decide whether the
hardware is suitable for the app or not. Information about the device is
available in the form of:

 - Properties: Device name, type, supported Vulkan version, etc.
 - Features: Describe support for option features, such as shader support.

### Queue families

Almost every operation in Vulkan, from drawing to uploading a texture, requires
submitting a command buffer to a queue. Those commands are executed in order,
even though they might finish in a different order.

A physical device have different types of queue families depending on the
supported commands.

### Logical device and queues

A logical device is the interface to a physical device.

In order to create it we need:

 - A physical device to interface with.
 - Information about the queues to use and its queue family index.
 - The physical device features to use.

Once the logical device is created, we can obtain handles to its queues.

### Window surface

Since Vulkan is a platform agnostic API, in order to interact with the window
system we need to use the VK_KHR_surface extension.

This instance extension receives the native handlers to the window and creates a
surface Vulkan is able to draw on.

In addition, to a queue with graphics support, this extension requires an
additional queue with presentation support in order to display graphics on the
surface.

### Swap chain

Unlike in OpenGL, Vulkan does not have a default framebuffer. Instead, the swap
chain is used to manage the framebuffers that will be displayed on the window
surface.

The swap chain is a queue of images waiting to be displayed on the screen on
which the application draws before they are displayed.

Depending on the configuration, the images will be displayed in different ways.
The available properties of the swap chain are:

 - Capabilities: Min/max number of images in the swap chain, min/max width of
   the images, etc.
 - Surface format: Pixel format and color space.
 - Presentation mode: Conditions for swapping images on the screen: immediate,
   FIFO, mailbox (~triple buffering), etc.
 - Swap extend: Width and heigh of the images in the swap chain, usually equal
   to the size in pixels of the window surface.

### Graphics pipeline basics

The graphics pipeline is a sequence of operations that take the vertices and
textures and transforms them in pixels.

The operations are:

 - Input assembler: Collect the raw vertex data from buffers specified by the
   application.
 - Vertex shader: Run for each vertex transforming their position from model
   space to screen space.
 - Tessellation shader: Allow to subdivide geometry, i.e., transform 1 triangle
   in 3 triangles to increase the mesh quality.
 - Geometry shader: Run for each primitive (triangle, line, point) to discard it
   or output more primitives.
 - Rasterization stage: Discretizes primitives into fragments, discarding
   fragments behind other fragments or outside of the screen.
 - Fragment shader: Run for each fragment determining which framebuffer(s) they
   are written to and with which it color and depth.
 - Color blending: Mix fragments that map to the same pixel in the framebuffer
   based on transparency.

### SPIR-V

Unlike in OpenGL, shader code in Vulkan (and OpenCL) is specified in a bytecode
format called SPIR-V.

In order to compile GLSL to SPIR-V, Khronos provides a vendor-independent
compiler.

### Programable pipeline stages: Shaders

The programable stages of the graphics pipeline phases are: Vertex shader,
tessellation shader, geometry shader and fragment shader.

In these stages it is possible to upload your own code to the GPU in form of a
shader program. The steps to attach shaders to a pipeline are:

 - Compile the GLSL shader to SPIR-V.
 - Wrap them in a shader module, a small structure that wraps the binary data.
 - Wrap the shader module in VkPipelineShaderStageCreateInfo structure, with
   information about the stage and the shader main function.

### Pipeline fixed-function stages

There are 3 stages in the pipeline that allow to tweak their operations using
parameters but, unlike the programable stages, the way they work is predefined.

These stages are: Input assembler, rasterization and color blending.

The parameters that can be configured are:

 - Vertex input: Describes the format of the vertex data passed to the vertex
   shader.
 - Input assembly: Describes what kind of geometry will be drawn from the
   vertices (points, lines, triangles and line/triangle strips) and if primitive
   restart should be enabled (when reusing vertices using strips, for example,
   using the last vertex of a line as the first one of the next, it allows to
   break this chain using an special value).
 - Viewport: Region of the framebuffer to be rendered.
 - Scissors: Area within the viewport to be rendered.
 - Rasterizer: The rasterizer discretizes primitives into fragments. Parameters
   such as polygon mode (fill, line or point) or line width can be configured.
 - Multisampling: Configures anti-aliasing.
 - Color blending: Describes how colors returned by the fragment shader should
   be mixed.
 - Pipeline layout: Describes the layout of the shader uniforms
