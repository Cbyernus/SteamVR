# DirectX12 support in SteamVR

SteamVR supports submission of eye buffers originating from the DirectX12 graphics API.  

## Image description

When submitting a DirectX12 image as an overlay or eye texture into SteamVR, use **TextureType_DirectX12** for its **ETextureType**. The handle to pass into the **Texture_t** structure should be a pointer to a **D3D12TextureData_t** structure containing the DirectX12 resources needed for the runtime to process it:

```c++
struct D3D12TextureData_t
{
	ID3D12Resource *m_pResource;
	ID3D12CommandQueue *m_pCommandQueue;
	uint32_t m_nNodeMask;
};
```
* m_pResource - pointer to **ID3D12Resource** created with **D3D12_RESOURCE_DIMENSION_TEXTURE2D**.
* m_pCommandQueue - pointer to **ID3D12CommandQueue** on which SteamVR will execute the transfer of the image.
* m_nNodeMask - in the case of multi-adapter, this is a bitmask of the set of nodes.  If not using multi-adapter, set this to 0.

## Image Resource State

The D3D12 **m_pResource** should be transitioned to the resource state **D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE** prior to passing it ot the SteamVR runtime.  The resource will remain in the **D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE** state when the submission work has finished executing.

## Explicit Timing
DirectX12 applications can enable explicit timing mode by calling **IVRCompositor::SetExplicitTimingMode**. In DirectX12, the purpose of explicit timing mode is to get a more accurate GPU timestamp for when the frame begins.

More accurate GPU timestamp for the start of the frame is achieved by the application calling **IVRCompositor::SubmitExplicitTimingData** immediately before its first submission to the **ID3D12CommandQueue** for the frame. This is more accurate because normally this GPU timestamp is recorded during WaitGetPoses. In D3D11, WaitGetPoses queues a GPU timestamp write, but it does not actually get submitted to the GPU until the application performs it next flush. By using SubmitExplicitTimingData, the timestamp is recorded at the same place for DirectX12 as it is for D3D11, resulting in a more accurate GPU time measurement for the frame.

## Example Code

An example of using DirectX12 with SteamVR on Windows (Visual Studio 2015+) can be found in the SDK [[https://github.com/ValveSoftware/openvr/tree/master/samples/hellovr_dx12]].  
