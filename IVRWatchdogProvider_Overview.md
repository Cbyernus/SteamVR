Any driver that implements this interface must build for both 32-bit and 64-bit versions.
`steam.exe` is responsible for loading the watchdog driver (in 32-bit), not `vrserver.exe`. You have to close both Steam and SteamVR before building the DLL.

One trick to build both versions is to use **Build ⇾ Batch Build** function in Visual Studio.

Check this issue for more info: https://github.com/ValveSoftware/openvr/issues/1486