This is an overview of the steps to write a new OpenVR driver. In the examples, the driver is simply called "mydriver".

It is advisable to use the built-in SteamVR drivers as references. For example, the Oculus driver is maintained by Valve and demonstrates nearly all of the below features (excluding driver registration). The built-in drivers can be found in your Steam install directory in the sub-path 'Steam\steamapps\common\SteamVR\drivers'.

1. Put all the files for your driver under a directory. Let's say that directory is "<install dir>/mydriver"
2. Add a new DLL to "mydriver/bin/win64/driver_mydriver.dll" (For Linux use the appropriate platform directory and the appropriate shared library extension.)
3. Implement the [driver factory function](https://github.com/ValveSoftware/openvr/wiki/Driver-Factory-Function) in that DLL.
4. ~~Add an implementation of [`vr::IClientTrackedDeviceProvider`](https://github.com/ValveSoftware/openvr/wiki/IClientTrackedDeviceProvider_Overview) to the DLL and return it from the factory. This provider will be phased out in the next SDK udpate, but for now you need it.~~ This was removed in [OpenVR version 1.0.6](https://github.com/ValveSoftware/openvr/commit/70acfe9262290ddb789588a7390e5fc60bb20080#diff-614ced34b3fbb27d875cdae21a8a16e6).
5. Add an implementation of [`vr::IServerTrackedDeviceProvider`](https://github.com/ValveSoftware/openvr/wiki/IServerTrackedDeviceProvider_Overview) and have that return implementations of [`vr::ITrackedDeviceServerDriver`](https://github.com/ValveSoftware/openvr/wiki/vr::ITrackedDeviceServerDriver-Overview) for each tracked device.
6. Add a [driver manifest file](https://github.com/ValveSoftware/openvr/wiki/DriverManifest) to "<installdir>/mydriver"
7. Add and remove your driver to SteamVR's config file using vrpathreg. vrpathreg.exe is available in the user's SteamVR install directory. Invoke it with the adddriver command like so: 
vrpathreg adddriver "<installdir>/mydriver". 
Support the clean removal of your driver by using the 'removedriver' argument, again with your driver path like so: 
vrpathreg removedriver "<installdir>/mydriver".
* SteamVR's install directory can be located on Windows using the registry. On Windows 10, use the standard uninstall key: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Steam App 250820
* If you are working directly with Valve to deploy your driver, this step should occur in your installer/uninstaller, not in your driver.

For drivers with status icons, you can add icons files to 'mydriver/resources/icons'. These metadata to map hardware status to the appropriate icon belongs in 'mydriver/resources/driver.vrresources'.

For drivers with custom artwork in the input binding UI, add SVG files to the same folder, 'mydriver/resources/icons'.

For drivers with localized strings, add localization for all languages to 'mydriver/resources/localization/localization.json'. This will affect the strings that are displayed in the UI related to your hardware, for example, in the input binding UI. Generally, you only need to provide translations for the names of your hardware and input paths. If localization is working for some, but not all of your devices, then you may need to explicitly tell OpenVR to use a certain resource namespace. To do this, add a line to the input profile for the device that is not working, "resource_root" : "<root name>", where the <root name> is replaced with the driver's name as written in the .vrdrivermanifest. 

For devices with buttons, triggers, joysticks, and other kinds of input controls, please refer to the [`vr::IVRDriverInput`](https://github.com/ValveSoftware/openvr/wiki/IVRDriverInput-Overview) API.

For devices that wish to provide animation data through the Skeletal Input system, please refer to the [Skeletal Input Driver](https://github.com/ValveSoftware/openvr/wiki/Creating-a-Skeletal-Input-Driver) documentation.

The source code for a sample driver is available [here](https://github.com/ValveSoftware/openvr/tree/master/samples/driver_sample). The sample implements these interfaces and includes an example HMD/controller.

Other driver documentation:
* [Device Sleep States](https://github.com/ValveSoftware/openvr/wiki/Device-sleep-states)
* [Driver Direct Mode](https://github.com/ValveSoftware/openvr/wiki/Driver-direct-mode)

