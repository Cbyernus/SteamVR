# Input Overview

To read controller input state in OpenVR, an application provides a list of the "actions" that represent the operations a user can perform in the application. SteamVR then binds those actions to input components provided for the input devices that the user is using. Application developers can provide default bindings for any controller types they like. Users are able to modify those bindings or create new bindings for new devices. Users can also select bindings that were shared by other users.

# Getting Started

To use the IVRInput API in an application, a developer should take the following steps:

1. Check "Enable debugging options in the input binding user interface" under Developer settings in SteamVR. This will enable some options that will be used in later steps.
1. Create an [action manifest file](https://github.com/ValveSoftware/openvr/wiki/Action-manifest) to allow SteamVR to know what actions exist in the application.
2. Add a call to `vr::VRInput()->SetActionManifestPath()` to the application's initialization. This function takes an absolute path to the application's action manifest file. 
3. Add several calls to `vr::VRInput()->GetActionHandle()` to the application's initialization. These calls turn action paths (e.g. /actions/my_app/in/DoSomething) into handles that can be used at runtime.
4. Add one or more calls to `vr::VRInput()->GetActionSetHandle()` to the application's initialization. These calls turn action set paths (e.g. /actions/my_app) into handles that can be used at runtime.
5. Add a call to `vr::VRInput()->UpdateActionState()` at the top of the application's input processing code before reading any action state.
6. Add calls to `vr::VRInput()->Get*ActionData()` for each action used by the application. Make the results from those calls perform the action in the application.
7. Run the application.
7. Click "Controller Binding" under Settings and find your application as the top under the running application. Make some bindings using the UI and then click "export" at the bottom of the screen. Your bindings will be exported to <my documents>/steamvr/input/exports.
8. Move your exported binding into your application's directory. It doesn't matter where they are, but usually putting them next to the action manifest file is a good choice.
9. Edit the action manifest and add a `default_bindings` section that refers to your default binding file. 
10. Repeat 7-9 for other controller types you want to provide default bindings for.
10. Include your action manifest and all your default binding files in your app when it releases.

# API Documentation


`EVRInputError IVRInput::SetActionManifestPath( const char *pchActionManifestPath )`

Sets the path to the action manifest JSON file that is used by this application. If this information
was set on the Steam partner site, calls to this function are ignored. If the Steam partner site
setting and the path provided by this call are different, `VRInputError_MismatchedActionManifest` is returned. 
This call must be made before the first call to `IVRInput::UpdateActionState` or `IVRSystem::PollNextEvent`.

* `pchActionManifestPath` - Absolute path to the action manifest file to use for this application. See [action manifest file](https://github.com/ValveSoftware/openvr/wiki/Action-manifest) documentation for the format of these files.


`EVRInputError IVRInput::GetActionSetHandle( const char *pchActionSetName, VRActionSetHandle_t *pHandle )`

Returns a handle for an action set. This handle is used for all performance-sensitive calls.

For best performance, applications should call this function once per handle.

* `pchActionSetName` - The path to the action set. This is generally of the form /actions/action_set_name
* `pHandle` - A pointer to an action set variable to fill with the handle.

`EVRInputError IVRInput::GetActionHandle( const char *pchActionName, VRActionHandle_t *pHandle )`

Returns a handle for an action. This handle is used for all performance-sensitive calls.

For best performance, applications should call this function once per handle.

* `pchActionName` - The path to the action. This is generally of the form /actions/action_set_name/in/action_name or /actions/action_set_name/out/action_name.
* `pHandle` - A pointer to an action handle variable to fill with the handle.

`EVRInputError IVRInput::GetInputSourceHandle( const char *pchInputSourcePath, VRInputValueHandle_t  *pHandle )`

Returns a handle for an input source. This handle is used for all performance-sensitive calls.

For best performance, applications should call this function once per handle.

* `pchInputSourcePath` - The path to the input source. Common input sources are /user/head, /user/hand/left, /user/hand/right, and /user/gamepad.
* `pHandle` - A pointer to an input source handle variable to fill with the handle.

`EVRInputError IVRInput::UpdateActionState( VRActiveActionSet_t *pSets, uint32_t unSizeOfVRSelectedActionSet_t, uint32_t unSetCount )`

Reads the current state into all actions. After this call, the results of Get*ActionData calls will be the same until the next call to UpdateActionState. Applications should call this function once per frame.

* `pSets` - A pointer to an array of one or more VRActiveActionSet_t structures that the application would like to be active this frame.
* `unSizeofVRSelectedActionSet_t` - This parameter should be sizeof(VRActiveActionSet_t).
* `unSetCount` - The number of action VRActiveActionSet_t structs in the array.

```
struct VRActiveActionSet_t
{
	VRActionSetHandle_t ulActionSet;
	VRInputValueHandle_t ulRestrictedToDevice;
	VRActionSetHandle_t ulSecondaryActionSet;
};
```

This structure describes one active action set for a given frame.

* `ulActionSet` - The handle of an action set to activate.
* `ulRestrictedToDevice` - The device to restrict `ulActionSet` to. This is generally k_ulInvalidInputValueHandle. If it set, it could be the handle for /user/hand/left or /user/hand/right.
* `ulSecondaryActionSet` - If ulRestrictedToDevice is set, this is the alternate action set to use for all other devices in the system.

`EVRInputError IVRInput::GetDigitalActionData( VRActionHandle_t action, InputDigitalActionData_t *pActionData, uint32_t unActionDataSize )`

Reads the state of a digital action given its handle. This will return VRInputError_WrongType if the type of
action is something other than digital.

* `action` - The handle of the action to retrieve state for
* `pActionData` - Pointer to an InputDigitalActionData_t struct. 
* `unActionDataSize` - This must be sizeof(InputDigitalActionData_t)


```
struct InputDigitalActionData_t
{
	bool bActive;
	VRInputValueHandle_t activeOrigin;
	bool bState;
	bool bChanged;
	float fUpdateTime;
};
```

* `bActive` - This action is bound to an input source that is present in the system and is in an action set that is active.
* `activeOrigin` - The input source that this action state was generated by. If this action is bound to multiple inputs, this will be the input that changed most recently.
* `bState` - The current state of this digital action. True means the user wants to perform this action.
* `bChanged` - If this field is true, the digital action's state value has changed since the previous frame. As a result, applications can use this to detect rising edges (bState && bChanged) or falling edges (!bState && bChanged) to avoid duplicate activations on steady states.
* `fUpdateTime` - The time relative to now when the action was last changed. This can be passed to GetPoseActionData to get the pose of a user's hand at the time they pressed or released an input control.



		/** Reads the state of an analog action given its handle. This will return VRInputError_WrongType if the type of
		* action is something other than analog */
		virtual EVRInputError GetAnalogActionData( VRActionHandle_t action, InputAnalogActionData_t *pActionData, uint32_t unActionDataSize ) = 0;

		/** Reads the state of a pose action given its handle. */
		virtual EVRInputError GetPoseActionData( VRActionHandle_t action, ETrackingUniverseOrigin eOrigin, float fPredictedSecondsFromNow, InputPoseActionData_t *pActionData, uint32_t unActionDataSize ) = 0;

		/** Reads the state of a skeletal action given its handle. */
		virtual EVRInputError GetSkeletalActionData( VRActionHandle_t action, EVRSkeletalTransformSpace eBoneParent, float fPredictedSecondsFromNow, InputSkeletonActionData_t *pActionData, uint32_t unActionDataSize, VR_ARRAY_COUNT( unTransformArrayCount ) VRBoneTransform_t *pTransformArray, uint32_t unTransformArrayCount ) = 0;

		/** Reads the state of a skeletal action given its handle in a compressed form that is suitable for
		* sending over the network. The required buffer size will never exceed ( sizeof(VR_BoneTransform_t)*boneCount + 2).
		* Usually the size will be much smaller. */
		virtual EVRInputError GetSkeletalActionDataCompressed( VRActionHandle_t action, EVRSkeletalTransformSpace eBoneParent, float fPredictedSecondsFromNow, VR_OUT_BUFFER_COUNT( unCompressedSize ) void *pvCompressedData, uint32_t unCompressedSize, uint32_t *punRequiredCompressedSize ) = 0;

		/** Turns a compressed buffer from GetSkeletalActionDataCompressed and turns it back into a bone transform array. */
		virtual EVRInputError UncompressSkeletalActionData( void *pvCompressedBuffer, uint32_t unCompressedBufferSize, EVRSkeletalTransformSpace *peBoneParent, VR_ARRAY_COUNT( unTransformArrayCount ) VRBoneTransform_t *pTransformArray, uint32_t unTransformArrayCount ) = 0;

		// --------------- Haptics ------------------- //

		/** Triggers a haptic event as described by the specified action */
		virtual EVRInputError TriggerHapticVibrationAction( VRActionHandle_t action, float fStartSecondsFromNow, float fDurationSeconds, float fFrequency, float fAmplitude ) = 0;

		// --------------- Action Origins ---------------- //

		/** Retrieve origin handles for an action */
		virtual EVRInputError GetActionOrigins( VRActionSetHandle_t actionSetHandle, VRActionHandle_t digitalActionHandle, VR_ARRAY_COUNT( originOutCount ) VRInputValueHandle_t *originsOut, uint32_t originOutCount ) = 0;

		/** Retrieves the name of the origin in the current language */
		virtual EVRInputError GetOriginLocalizedName( VRInputValueHandle_t origin, VR_OUT_STRING() char *pchNameArray, uint32_t unNameArraySize ) = 0;

		/** Retrieves useful information for the origin of this action */
		virtual EVRInputError GetOriginTrackedDeviceInfo( VRInputValueHandle_t origin, InputOriginInfo_t *pOriginInfo, uint32_t unOriginInfoSize ) = 0;

		/** Shows the current binding for the action in-headset */
		virtual EVRInputError ShowActionOrigins( VRActionSetHandle_t actionSetHandle, VRActionHandle_t ulActionHandle ) = 0;

		/** Shows the current binding all the actions in the specified action sets */
		virtual EVRInputError ShowBindingsForActionSet( VR_ARRAY_COUNT( unSetCount ) VRActiveActionSet_t *pSets, uint32_t unSizeOfVRSelectedActionSet_t, uint32_t unSetCount, VRInputValueHandle_t originToHighlight ) = 0;

