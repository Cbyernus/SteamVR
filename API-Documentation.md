# Overview
The OpenVR API provides a game with a way to interact with Virtual Reality displays without relying on a specific hardware vendor's SDK. It can be updated independently of the game to add support for new hardware or software updates. 

This means that you can access the position of the headset using only one API call and it works for all headset brands (Oculus, Mixed Reality, Vive, etc)!

The API is implemented as a set of C++ interface classes full of pure virtual functions. When an application initializes the system it will return the interface that matches the header in the SDK used by that application. Once a version of an interface is published, it will be supported in all future versions, so the application will not need to update to a new SDK to move forward to new hardware and other features.

## Concrete practical example
OpenVR is divided into 2 layers: application and driver.
OpenVR for application talks to SteamVR. SteamVR then talks to OpenVR driver. So OpenVR application works at a higher level than an OpenVR driver.

### OpenVR Application
One real-world example of an application is a game engine like Unity. Unity uses OpenVR to get the position of the headset and apply that to the main camera without the need to care which brand of headset you are using. Unity then sends the camera image to OpenVR. OpenVR does some operation on the image and then display to the real headset monitor.

For a simple example, you can also build a small application that just prints the position of the headset to the console. Most of the API documentation in this wiki is about building OpenVR application. An application usually starts with calling `vr::VR_Init()` function from `openvr.h`.

### OpenVR Driver
An OpenVR driver is a software that introduces VR devices into SteamVR system. You would want to write a driver when you create new hardware (or virtual hardware) like a headset or controller. To write a driver, you should start with writing a `HmdDriverFactory()` function and include  `openvr_driver.h`.

# Initialization and Cleanup

Because the OpenVR API causes the game to connect to any attached VR hardware, it is not initialized automatically. To initialize the API and get access to the vr::IVRSystem interface call the vr::VR_Init function. To close down your connection to the hardware and release your vr::IVRSystem interface, call vr::VR_Shutdown.

`vr::IVRSystem *vr::VR_Init( vr::`[`HmdError`](https://github.com/ValveSoftware/openvr/wiki/HmdError)` *peError, vr::EVRApplicationType eApplicationType )`

eApplicationType must be one of:
* `VRApplication_Scene` - A 3D application that will be drawing an environment.
* `VRApplication_Overlay` - An application that only interacts with overlays or the dashboard.
* `VRApplication_Background` - The application will not start SteamVR. If it is not already running the call with VR_Init will fail with `VRInitError_Init_NoServerForBackgroundApp`.
* `VRApplication_Utility` - The application will start up even if no hardware is present. Only the IVRSettings and IVRApplications interfaces are guaranteed to work.  This application type is appropriate for things like installers.

The call will return a vr::IVRSystem pointer that allows the game to call other OpenVR API methods. If something fails the call will return NULL and peError will be set to an error code that indicates what the problem was.
peError - The error code that occurred or vr::VRInitError_None if there was no error. See [`vr::HmdError`](https://github.com/ValveSoftware/openvr/wiki/HmdError) for possible error codes.


`void vr::VR_Shutdown()`

Shuts down the connection to the VR hardware and cleans up the OpenVR API. The vr::IVRSystem pointer returned by vr::VR_Init will be invalid after this call is made. 

# Interfaces

The API is broken down into six primary interfaces in the vr namespace:
* [IVRSystem](https://github.com/ValveSoftware/openvr/wiki/IVRSystem_Overview) - Main interface for display, distortion, tracking, controller, and event access.
* [IVRChaperone](https://github.com/ValveSoftware/openvr/wiki/IVRChaperone_Overview) - Provides access to chaperone soft and hard bounds.
* [IVRCompositor](https://github.com/ValveSoftware/openvr/wiki/IVRCompositor_Overview) - Allows an application to render 3D content through the VR compositor.
* [IVROverlay](https://github.com/ValveSoftware/openvr/wiki/IVROverlay_Overview) - Allows an application to render 2D content through the VR compositor.
* [IVRRenderModels](https://github.com/ValveSoftware/openvr/wiki/IVRRenderModels_Overview) - Allows an application access to render models.
* [IVRScreenshots](https://github.com/ValveSoftware/openvr/wiki/IVRScreenshots_Overview) - Allows an application to request and submit screenshots.
* [IVRInput](https://github.com/ValveSoftware/openvr/wiki/SteamVR-Input) - Allows an applications to define and query invokeable actions (and action sets) so that users may create, edit, and share custom bindings with any supported device.

# Other Functions

`bool vr::VR_IsHmdPresent()`

Returns true if the system believes that an HMD is present on the system. This function is much faster than initializing all of OpenVR just to check for an HMD. Use it when you have a piece of UI that you want to enable only for users with an HMD.

This function will return true in situations where vr::VR_Init() will return NULL. It is a quick way to eliminate users that have no VR hardware, but there are some startup conditions that can only be detected by starting the system.


`bool vr::VR_IsRuntimeInstalled()`

Returns true if the OpenVR runtime is installed on the system.


`char *vr::VR_RuntimePath()`

Returns where the OpenVR runtime is installed.


`const char *VR_GetVRInitErrorAsSymbol( vr::EVRInitError error );`

This function returns the vr::EVRInitError enum value as a string. It can be called any time, regardless of whether the VR system is started up.


`void *VR_GetGenericInterface( const char *pchInterfaceVersion, vr::`[`EVRInitError `](https://github.com/ValveSoftware/openvr/wiki/HmdError)` *peError )`

Requests an interface by name from OpenVR. It will return NULL and pass back an error in peError if the interface can't be found. It will always return NULL if vr::VR_Init() has not been called successfully.


`bool VR_IsInterfaceVersionValid( const char *pchInterfaceVersion )`

Returns true if the interface name and version is supported by the installed runtime.