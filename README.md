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
