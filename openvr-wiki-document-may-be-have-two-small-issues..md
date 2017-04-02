- Unable to obtain warehouse via Git
I run this cmd to obtain:
```
git clone https://github.com/ValveSoftware/openvr.wiki.git
```
After a while, the final result is:
```
...
error: unable to create file IVRSystem::TriggerHapticPulse.md: Invalid argument
error: unable to create file vr::ITrackedDeviceServerDriver-Overview.md: Invalid argument
fatal: unable to checkout working tree
warning: Clone succeeded, but checkout failed.
You can inspect what was checked out with 'git status'
and retry the checkout with 'git checkout -f HEAD'
```

- something the matter in "vr::ITrackedDeviceServerDriver Overview" 
Some non ITrackedDeviceServerDriver interfaces are also placed there(from **Display Methods**)
