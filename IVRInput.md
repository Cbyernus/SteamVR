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



		/** Returns a handle for an action set. This handle is used for all performance-sensitive calls. */
		virtual EVRInputError GetActionSetHandle( const char *pchActionSetName, VRActionSetHandle_t *pHandle ) = 0;

		/** Returns a handle for an action. This handle is used for all performance-sensitive calls. */
		virtual EVRInputError GetActionHandle( const char *pchActionName, VRActionHandle_t *pHandle ) = 0;

		/** Returns a handle for any path in the input system. E.g. /user/hand/right */
		virtual EVRInputError GetInputSourceHandle( const char *pchInputSourcePath, VRInputValueHandle_t  *pHandle ) = 0;

		// --------------- Reading action state ------------------- //

		/** Reads the current state into all actions. After this call, the results of Get*Action calls 
		* will be the same until the next call to UpdateActionState. */
		virtual EVRInputError UpdateActionState( VR_ARRAY_COUNT( unSetCount ) VRActiveActionSet_t *pSets, uint32_t unSizeOfVRSelectedActionSet_t, uint32_t unSetCount ) = 0;

		/** Reads the state of a digital action given its handle. This will return VRInputError_WrongType if the type of
		* action is something other than digital */
		virtual EVRInputError GetDigitalActionData( VRActionHandle_t action, InputDigitalActionData_t *pActionData, uint32_t unActionDataSize ) = 0;

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

