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

The extensions to enable need to match the SteamVR runtime software and will change as SteamVR gets updated, so your application needs to make sure to call this at runtime and use the output result to enable extensions at instance and device creation time. **This is important, as just copying the result of these calls once into your application will cause it to break in the future when SteamVR gets updated with a newer version of its Vulkan submission code**.

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

Vulkan requires that only a single thread access a VkQueue at a time.  When any Vulkan Texture_t is passed to the runtime (through **IVRCompositor::Submit** or otherwise), the runtime will schedule work to the Vulkan queue represented by **m_pQueue**. As such, no other thread of your application should try to access that queue until the call returns.

In addition to **IVRCompositor::Submit**, the following functions may also access the queue:
 * **IVRCompositor::PostPresentHandoff**
 * **IVRCompositor::WaitGetPoses** (does not need to access the queue if using Explicit Timing and PostPresentHandoff, see [Explicit Timing](#explicit-timing))
 * **IVRCompositor::SubmitExplicitTimingData** (see [Explicit Timing](#explicit-timing))


## Image layout

Any Vulkan image represented by **m_nImage** should be in the **VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL** layout when passed to the SteamVR runtime, and will still be in that state when the submission work has finished executing.

## Image usage flags

Any Vulkan image represented by **m_nImage** must have been created with at least the following VkImageUsageFlags: **VK_IMAGE_USAGE_TRANSFER_SRC_BIT** | **VK_IMAGE_USAGE_SAMPLED_BIT**.

## Image Formats

The following image formats are currently supported for the **m_nFormat** of Vulkan Texture_t (through **IVRCompositor::Submit** or otherwise):

```c++
VK_FORMAT_R8G8B8A8_UNORM
VK_FORMAT_R8G8B8A8_SRGB
VK_FORMAT_B8G8R8A8_UNORM
VK_FORMAT_B8G8R8A8_SRGB
VK_FORMAT_R32G32B32A32_SFLOAT
VK_FORMAT_R32G32B32_SFLOAT
VK_FORMAT_R16G16B16A16_SFLOAT
VK_FORMAT_A2R10G10B10_UINT_PACK32
```

## Explicit Timing
Vulkan applications can enable explicit timing mode by calling **IVRCompositor::SetExplicitTimingMode**.  There are two purposes for SetExplicitTimingMode:
1. To get a more accurate GPU timestamp for when the frame begins.
2. (Optional) To avoid having WaitGetPoses access the Vulkan queue so that the queue can be accessed from another thread while WaitGetPoses is executing.

More accurate GPU timestamp for the start of the frame is achieved by the application calling **IVRCompositor::SubmitExplicitTimingData** immediately before its first submission to the Vulkan queue for the frame.  This is more accurate because normally this GPU timestamp is recorded during WaitGetPoses.  In D3D11, WaitGetPoses queues a GPU timestamp write, but it does not actually get submitted to the GPU until the application performs its next flush.  By using SubmitExplicitTimingData, the timestamp is recorded at the same place for Vulkan as it is for D3D11, resulting in a more accurate GPU time measurement for the frame.

Avoiding WaitGetPoses accessing the Vulkan queue can be achieved using SetExplicitTimingMode as well.  If this is desired, the application must call PostPresentHandoff itself prior to WaitGetPoses.  If SetExplicitTimingMode is true and the application calls PostPresentHandoff, then WaitGetPoses is guaranteed not to access the queue.  Note that PostPresentHandoff and SubmitExplicitTimingData will access the queue, so only WaitGetPoses becomes safe for accessing the queue from another thread.

## Example Code
An example of using Vulkan with SteamVR on Windows/Linux can be found in the SDK [[https://github.com/ValveSoftware/openvr/tree/master/samples/hellovr_vulkan]].  
