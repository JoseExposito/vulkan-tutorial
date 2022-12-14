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

### Render passes

A render pass consists of one or more subpasses and multiple attachments like
color buffers or depth buffers.

Subpasses are subsequent rendering operations that depend on the contents of
framebuffers in previous passes.

### The Vulkan graphics pipeline

Now we can combine the concepts explained so far to create the Vulkan graphics
pipeline:

 - Shader stages: Define the functionality of the programmable stages.
 - Fixed-function state: Structures that define the fixed-function stages of the
   pipeline.
 - Pipeline layout: Uniform and push values referenced by the shader that can be
   updated at draw time.
 - Render pass: Attachments referenced by the pipeline stages and their usage.

### Command buffers

Commands in Vulkan (drawing operations, memory transfers, etc) are not executed
calling functions. Instead, they are recorded in a command buffer and sent to a
queue.

The steps to create a command buffer and record it are:

 - Create a command pool as a memory pool for command buffers
 - Create the command buffer
 - Start recording in the command buffer by calling vkBeginCommandBuffer
 - Start the render pass
 - Bind the pipeline
 - Record the render commands
 - End the render pass
 - Stop recording in the command buffer by calling vkEndCommandBuffer

### Synchronization

These are the main synchronization mechanisms available in Vulkan:

 - Binary semaphores: Used to add order between queue operations in the GPU.
   The semaphore starts unsignaled. Once it is signaled, the next queue
   operation can run. Pseudocode example:

   ```
   VkCommandBuffer A, B = ... // record command buffers
   VkSemaphore S = ... // create a semaphore

   // enqueue A, signal S when done - starts executing immediately
   vkQueueSubmit(work: A, signal: S, wait: None)

   // enqueue B, wait on S to start
   vkQueueSubmit(work: B, signal: None, wait: S)

   // The CPU does not wait, the synchronization happens in the GPU
   ```
 - Fences: Used to wait (in the CPU) for the GPU to finish.
   ```
   VkCommandBuffer A = ... // record command buffer with the transfer
   VkFence F = ... // create the fence
   // enqueue A, start work immediately, signal F when done
   vkQueueSubmit(work: A, fence: F)
   vkWaitForFence(F) // blocks execution until A has finished executing
   save_screenshot_to_disk() // can't run until the transfer has finished
   ```

### Drawing

Finally, with all the concepts learn so far, we can star drawing. The high level
steps to draw (in a very inefficient way) are:

 - Wait for the previous frame to finish: Use a fence to make the GPU wait until
   the previous frame has finished.
 - Acquire an image from the swap chain: By calling vkAcquireNextImageKHR.
 - Record a command buffer: Draws the scene onto that image. See the
   [Command buffers](#command-buffers) section.
 - Submit the recorded command buffer: Sets the synchronization semaphores and
   sends the recorded command buffer to the graphics queue by calling
   vkQueueSubmit().
 - Present the swap chain image: The last step is submitting the result back to
   the swap chain to have it eventually show up on the screen.
