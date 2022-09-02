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
