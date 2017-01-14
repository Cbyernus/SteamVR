Let's say you want to write a new OpenVR driver called "mydriver". To do that you will need to following the steps below.

1. Put all the files for your driver under a directory. Let's say that directory is "<install dir>/mydriver"
2. Add a new DLL to "mydriver/bin/win64/driver_mydriver.dll" (For Linux and OSX use the appropriate platform directory and the appropriate shared library extension.)
3. Implement the [driver factory function](https://github.com/ValveSoftware/openvr/wiki/Driver-Factory-Function) in that DLL.
4. Add an implementation of [`vr::IClientTrackedDeviceProvider`](https://github.com/ValveSoftware/openvr/wiki/IClientTrackedDeviceProvider_Overview) to the DLL and return it from the factory. This provider will be phased out in the next SDK udpate, but for now you need it.
5. Add an implementation of [`vr::IServerTrackedDeviceProvider`](https://github.com/ValveSoftware/openvr/wiki/IServerTrackedDeviceProvider_Overview) and have that return implementations of [`vr::ITrackedDeviceServerDriver`](https://github.com/ValveSoftware/openvr/wiki/vr::ITrackedDeviceServerDriver-Overview) for each tracked device.
6. Add a [driver manifest file](https://github.com/ValveSoftware/openvr/wiki/DriverManifest) to "<installdir>/mydriver"
7. Run: vrpathreg adddriver "<installdir>/mydriver"

