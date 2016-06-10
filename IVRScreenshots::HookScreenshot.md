`EVRScreenshotError HookScreenshot( const EVRScreenshotType *pSupportedTypes, int numTypes )`

Called by the running VR application to indicate that it wishes to be in charge of screenshots.  If the application does not call this, the Compositor will only support VRScreenshotType_Stereo screenshots that will be captured without notification to the running app.  Once hooked your application will receive a VREvent_RequestScreenshot event when the user presses the buttons to take a screenshot.

* `EVRScreenshotType *pSupportedTypes` - Pointer to an array of the screenshot types the application wants to support.
* `int numTypes` - The number of screenshot types in the pSupportedTypes array.
