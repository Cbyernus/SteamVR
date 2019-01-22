Let's say you want to write a new OpenVR driver called "mydriver". To do that you will need to following the steps below.

1. Put all the files for your driver under a directory. Let's say that directory is "<install dir>/mydriver"
2. Add a new DLL to "mydriver/bin/win64/driver_mydriver.dll" (For Linux and OSX use the appropriate platform directory and the appropriate shared library extension.)
3. Implement the [driver factory function](https://github.com/ValveSoftware/openvr/wiki/Driver-Factory-Function) in that DLL.
4. ~~Add an implementation of [`vr::IClientTrackedDeviceProvider`](https://github.com/ValveSoftware/openvr/wiki/IClientTrackedDeviceProvider_Overview) to the DLL and return it from the factory. This provider will be phased out in the next SDK udpate, but for now you need it.~~ This was removed in [OpenVR version 1.0.6](https://github.com/ValveSoftware/openvr/commit/70acfe9262290ddb789588a7390e5fc60bb20080#diff-614ced34b3fbb27d875cdae21a8a16e6).
5. Add an implementation of [`vr::IServerTrackedDeviceProvider`](https://github.com/ValveSoftware/openvr/wiki/IServerTrackedDeviceProvider_Overview) and have that return implementations of [`vr::ITrackedDeviceServerDriver`](https://github.com/ValveSoftware/openvr/wiki/vr::ITrackedDeviceServerDriver-Overview) for each tracked device.
6. Add a [driver manifest file](https://github.com/ValveSoftware/openvr/wiki/DriverManifest) to "<installdir>/mydriver"
7. Run: vrpathreg adddriver "<installdir>/mydriver"

For devices with buttons, triggers, joysticks, and other kinds of input controls, please refer to the [`vr::IVRDriverInput`](https://github.com/ValveSoftware/openvr/wiki/IVRDriverInput-Overview) API.

For devices that wish to provide animation data through the Skeletal Input system, please refer to the [Skeletal Input Driver](https://github.com/ValveSoftware/openvr/wiki/Creating-a-Skeletal-Input-Driver) documentation.  
