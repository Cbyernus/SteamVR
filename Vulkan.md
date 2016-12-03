# Vulkan support in SteamVR

SteamVR supports submission of eye buffers originating from the Vulkan graphics API. Due to the explicit nature of Vulkan, a bit of extra consideration is needed.

## Extensions

The SteamVR runtime will use the application's graphics queue to transmit textures to the SteamVR compositor. Depending on the platform, hardware and software version, it will need specific extensions enabled on the client application's instance and device. There are two Vulkan-specific methods in the IVRCompositor interface that can be used to query which extensions are required to enable:

```c++
	/** [Vulkan Only]
	* return 0. Otherwise it returns the length of the number of bytes necessary to hold this string including the trailing
	* null.  The string will be a space separated list of-required instance extensions to enable in VkCreateInstance */
	virtual uint32_t GetVulkanInstanceExtensionsRequired( VR_OUT_STRING() char *pchValue, uint32_t unBufferSize ) = 0;

	/** [Vulkan only]
	* return 0. Otherwise it returns the length of the number of bytes necessary to hold this string including the trailing
	* null.  The string will be a space separated list of required device extensions to enable in VkCreateDevice */
	virtual uint32_t GetVulkanDeviceExtensionsRequired( VkPhysicalDevice_T *pPhysicalDevice, VR_OUT_STRING() char *pchValue, uint32_t unBufferSize ) = 0;
```

The extensions to enable need to match the SteamVR runtime software and will change as SteamVR gets updated, so your application needs to make sure to call this at runtime and use the output result to enable extensions at instance and device creation time. **This is important, as just copying the result of these calls once into your application will cause it to break in the future when SteamVR gets updated with a newer version of its Vulkan submission code.**

## Image description

When submitting a Vulkan image as an overlay or eye texture into SteamVR, use **TextureType_Vulkan** for its **ETextureType**. The handle to pass into the **Texture_t** structure should be a pointer to a **VRVulkanTextureData_t** structure containing an explicit description of the image as well as the Vulkan resources needed for the runtime to process it:

```c++
struct VRVulkanTextureData_t
{
	uint64_t m_nImage; // VkImage
	VkDevice_T *m_pDevice;
	VkPhysicalDevice_T *m_pPhysicalDevice;
	VkInstance_T *m_pInstance;
	VkQueue_T *m_pQueue;
	uint32_t m_nQueueFamilyIndex;
	uint32_t m_nWidth, m_nHeight, m_nFormat, m_nSampleCount;
};
```

## Concurrency

When any Vulkan Texture_t is passed to the runtime (through **IVRCompositor::Submit** or otherwise), the runtime will schedule work to the Vulkan queue represented by **m_pQueue**. As such, no other thread of your application should try to access that queue until the call returns.

## Image layout

Any Vulkan image represented by **m_nImage** should be in the **VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL** layout when passed to the SteamVR runtime, and will still be in that state when the submission work has finished executing.