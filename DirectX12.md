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

## Example Code

An example of using DirectX12 with SteamVR on Windows (Visual Studio 2015+) can be found in the SDK [[https://github.com/ValveSoftware/openvr/tree/master/samples/hellovr_dx12]].  
