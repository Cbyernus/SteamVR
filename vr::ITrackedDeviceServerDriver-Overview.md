The `vr::ITrackedDeviceServerDriver` interface represents a single physical tracked device in the system. It is implemented in driver dynamic libs. 

Once an instance of this interface is provided to vrserver it should remain valid until Cleanup is called on the `ITrackedDeviceServerProvider` object.

**`HmdError Activate( uint32_t unObjectId )`**

This is called before tracked device is returned to the application. It will always be
called before any display or tracking methods. Memory and processor use by the
`vr::ITrackedDeviceServerDriver` object should be kept to a minimum until it is activated. 
The pose listener is guaranteed to be valid until Deactivate is called, but 
should not be used after that point.


**`void Deactivate()`**

This is called when vrserver is switching to another driver.


**`const char *GetId()`**

Returns the serial number of this particular tracked device. This value is opaque to the VR system itself,
but should be unique within the driver because it will be passed back in via FindTrackedDeviceDriver.


**`void DebugRequest( const char *pchRequest, char *pchResponseBuffer, uint32_t unResponseBufferSize )`**

A VR Client has made this debug request of the driver. The set of valid requests is entirely
up to the driver and the client to figure out, as is the format of the response. Responses that
exceed the length of the supplied buffer should be truncated and null terminated.

# vr::IVRDisplayComponent Methods

**`void GetWindowBounds( int32_t *pnX, int32_t *pnY, uint32_t *pnWidth, uint32_t *pnHeight )`**

Provides the size and position that the window needs to be on the VR display. Note that the values you return from this function are probably just a composition of the rectangles returned by `GetEyeOutputViewport`.


**`bool IsDisplayOnDesktop()`**

Returns true if the display is extending the desktop.


**`bool IsDisplayRealDisplay()`**

Returns true if the display is real and not a fictional display.


**`void GetRecommendedRenderTargetSize( uint32_t *pnWidth, uint32_t *pnHeight )`**

Provides the suggested size for the intermediate render target that the distortion pulls from.
This function allows you to suggest a render resolution to the application to produce a final, crisp image, without unduly burdening the user's GPU.  A very large image will cause poor framerate and an overall bad experience. A small image will result in lower image clarity.

We suggest picking a value that counteracts the pixel compression at the center of your display. This maximizes the detail where the user looks the most, without rendering too many extra pixels. You can estimate the correct value using your own distortion measurements. At the center of the user's field of view, divide a small post-distortion pixel distance by its pre-distortion pixel distance, this will yield a good scaling value. Essentially, we are trying to answer the question, "How many pre-distortion pixels are needed to create one post-distortion pixel." 

For example, if `ComputeDistortion( EVREye_Eye_Left, 0.5f, 0.5f )` returns a green channel U,V of `(0.5, 0.5)`, and `ComputeDistortion( EVREye_Eye_Left, 0.52f, 0.50f )` returns `(0.528, 0.5)`, then the X delta was 0.02 and the U delta was 0.028. 0.028 / 0.02 = 1.4, or a scaling factor of 140%. Therefor, the pnWidth and pnHeight values returned by this function should be 140% of the display panel's actual resolution (assuming the whole panel is used). Note that the scaling may not be identical in both axis, and an exact scaling factor isn't strictly necessary.

These width and height values set the "100%" value in the user's settings menu, so it's nice to pick values that "look reasonable" to the user. Several factors will affect the actual rendered resolution. SteamVR performs a performance benchmark to estimate an appropriate tradeoff between resolution and performance. A user can adjust the resolution in the settings menu. An application can also choose to render at a different resolution. There's no guarantee that the values returned by your driver will actually be used.


**`void GetEyeOutputViewport( Hmd_Eye eEye, uint32_t *pnX, uint32_t *pnY, uint32_t *pnWidth, uint32_t *pnHeight )`**

Describes the viewport in the frame buffer to draw the output of the distortion into. Calling this function for each eye and combining all of the rectangles should yield the same area as is returned by `GetWindowBounds`.


**`void GetProjectionRaw( Hmd_Eye eEye, float *pfLeft, float *pfRight, float *pfTop, float *pfBottom )`**

The components necessary to build an application's projection matrix. The values are the tangents of the angles between the forward vector and the edges of the field of view. For example, if your HMD has an FOV of 90 degrees in the vertical and horizontal axes, then it has a 45 degree angle between the forward vector and each side. The values would be `tan(45 degrees)` which is 1.0,  and their signs would be left and top negative, such that:
```
*pfLeft = -1.0;
*pfRight = 1.0;
*pfTop = -1.0;
*pfBottom = 1.0;
```

When an application submits a frame, it may tell SteamVR about the field of view it used. If the application rendered with a larger FoV than the driver gave through `GetProjectionRaw`, then the image will be cropped to the driver's FoV. If the application rendered with a smaller FoV, then the image will be inset such that 0..1 UV always maps to the driver's `GetProjectionRaw` values. An application can optionally specify the FoV it used by submitting a frame with one of the structures that contains a projection matrix, for example, vr::VRTextureWithDepth_t and vr::VRTextureWithPoseAndDepth_t.


**`HmdMatrix34_t GetHeadFromEyePose( Hmd_Eye eEye )`**

Returns the transform from eye space to the head space, including adjustments for IPD. Eye space is the per-eye transformation of head space that provides stereo disparity. The head space is the pose from IVRServerDriverHost::TrackedDevicePoseUpdated for the HMD. 

Instead of Head * View * Projection, the sequence is Head * View * Eye^-1 * Projection. Normally View and Eye^-1 will be multiplied together and treated as View in an application.


**`DistortionCoordinates_t ComputeDistortion( Hmd_Eye eEye, float fU, float fV )`**

Returns the result of the distortion function for the specified eye and input UVs. UVs go from 0,0 in the upper left of that eye's viewport and 1,1 in the lower right of that eye's viewport.

If the inputs are returned as the outputs, meaning that for each channel X = u and Y = v, the image will not be distorted by the compositor. The rendered image from the application will be mapped to the GetEyeOutputViewport. This should be done when you are writing a Direct Mode Driver, and you are using your own compositor and distortion processing.

### Usage in the context of a VR Game:
* When SteamVR starts, the compositor will gather distortion data. ComputeDistortion will be called several thousand times to sample the distortion space.
* The user launches a game...
* Before the first frame of the game, the game will ask SteamVR what the FoV of the display is. GetProjectionRaw will be used here. Most applications will never query the FoV again.
* Before the first frame, the game will ask SteamVR what size render target is recommended. GetRecommendedRenderTargetSize will be used here. The user may override the value passed to the game through the SteamVR settings menu, and the game may chose a different render resolution for its own reasons.
* Each Frame, the game queries for updated pose information for placing cameras in game. The camera pose is created by combining poses from TrackedDevicePoseUpdated and GetHeadFromEyePose. 


# Assorted capability methods

**`TrackedDeviceDriverInfo_t GetTrackedDeviceDriverInfo()`**

Returns all the static information that will be reported to the clients.
	

# Administrative Methods

**`const char *GetModelNumber()`**

Returns the model number of this tracked device.

**`const char *GetSerialNumber()`**

Returns the serial number of this tracked device.

# IPD Methods

**`float GetIPD()`**

Gets the current IPD (Interpupillary Distance) in meters.

# Tracking Methods

**`DriverPose_t GetPose()`**

Gets the initial pose of the device. Future poses should be provided via `IServerDriverHost::TrackedDevicePoseUpdated()` function.


# Property Methods

**`bool GetBoolTrackedDeviceProperty( TrackedDeviceProperty prop, TrackedPropertyError *pError )
float GetFloatTrackedDeviceProperty( TrackedDeviceProperty prop, TrackedPropertyError *pError ) = 0;
int32_t GetInt32TrackedDeviceProperty( TrackedDeviceProperty prop, TrackedPropertyError *pError ) = 0;
uint64_t GetUint64TrackedDeviceProperty( TrackedDeviceProperty prop, TrackedPropertyError *pError ) = 0;
uint32_t GetStringTrackedDeviceProperty( TrackedDeviceProperty prop, char *pchValue, uint32_t unBufferSize, TrackedPropertyError *pError )`**

Returns a property of the appropriate type.


# Controller Methods


**`VRControllerState_t GetControllerState()`**

Returns the initial state of a controller.


**`bool TriggerHapticPulse( uint32_t unAxisId, uint16_t usPulseDurationMicroseconds )`**

Returns a uint64 property. If the property is not available this function will return 0.
