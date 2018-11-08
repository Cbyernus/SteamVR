![](https://steamcdn-a.akamaihd.net/steamcommunity/public/images/clans/5519564/c4de76fcc6edd2380e753fbb7f9333ed76424f9f.png)

# Overview
The Skeletal Input system allows drivers for each type of controller to provide applications with an animated skeleton of the user's hand to the best level of fidelity that its sensors are able to detect.  Developers can then reduce their need to create custom hand animations for each controller they plan to support, and can instead use the animation provided by the current driver in conjunction with their own game-specific animations.  Skeletal data can be bound to input actions, just like other input.  The animation can be requested to be as accurate as possible (which will often mean that the range of motion of the hand will conform to the shape of the controller), or to represent the full range of motion of an empty human hand; in some cases these two options will effectively be the same.  The animation can also be provided as an additive offset from the tightest grip the user is able to make when grasping the controller, so that it can be layered on top of animations that the developer has made.  Finally, functionality is provided to compress and decompress the skeletal pose for the current frame to allow for more efficient transfer to remote clients in a networked multi-user application.  

# Getting Started
### Add an Action and Controller Binding
To use the Skeletal Input API, first add a "skeletal action" entry for each hand to the actions section of your application's [action manifest file](https://github.com/ValveSoftware/openvr/wiki/Action-manifest).  Make sure that the 'type' is skeleton, and that it has an additional 'skeleton' field to define which skeleton the action should use.  The choices for skeleton are either "/skeleton/hand/left" or "/skeleton/hand/right".  For example:

```JSON
{
"name": "/actions/demo/in/lefthand_anim",
"type": "skeleton",
"skeleton": "/skeleton/hand/left"
}
```

Next, you need to add a binding for this action to each controller.  You can do this through the SteamVR binding UI (preferred).  

### Setup the Action
Call `vr::VRInput()->GetActionSetHandle()` as described in the [SteamVR Input](https://github.com/ValveSoftware/openvr/wiki/SteamVR-Input) documentation to get a handle to the skeletal action that you defined in the manifest.  This will later be used to retrieve the skeletal pose associated with the action.  

### Retrieving the Pose
Each frame your application should already be calling `vr::VRInput()->UpdateActionState()` to update the status of all the actions that have been defined in your action manifest.  This will update the skeletal input too.  To retrieve the pose, first call `vr::VRInput()->GetSkeletalActionData()` with the 'VRActionHandle_t' for your skeletal action and a pointer to a `InputSkeletalActionData_t` object.  If `InputSkeletalActionData_t.bActive` is true, that means there is bone transform data that is ready.  `InputSkeletalActionData_t.boneCount` will contain the number of bones in the skeleton; you will need an array of at least this many `vr::VRBoneTransform_t` to hold the bone transforms.  

To retrieve the bone transforms, call `vr::VRInput()->GetSkeletalBoneData()` with the options for the format you want to receive the data in and buffer to place the transform in.  

### Retrieving a Compressed Pose
Instead of using `vr::VRInput()->GetSkeletalBoneData()` to retrieve the full bone transforms, you can instead call `vr::VRInput()->GetSkeletalBoneDataCompressed()` to retrieve a compressed binary blob suitable for networking.  Note that variable compression is used, so the size of the buffer will fluctuate as the hand pose changes.  Once you have transmitted the compressed buffer to a remote client, the bone transforms can be retrieved by passing the buffer to `vr::VRInput()->DecompressSkeletalBoneData()`.  Note that the remote client may not have the same type of controller as the client that sent the buffer, so the compression and decompression is designed to be used independent of any particular driver.  

### Placing the Skeleton in the World
The root bone transform provided by the API will always be in relation to an offset on the input device, and not a position in the VR space.  To get the location in VR space to place the skeleton so that it matches the location of the user's hand, you must retrieve it from the API the same way that you retrieve the locations of the controllers themselves, using `vr::VRInput()->GetPoseActionData()`, and passing in the value `InputSkeletalActionData_t.activeOrigin` that you received from calling `GetSkeletalActionData()`.  

# Skeleton Definition
Detailed information on the skeleton used by the Skeletal Input system can be found here: [Hand Skeleton](https://github.com/ValveSoftware/openvr/wiki/Hand-Skeleton)

# API Documentation

## Enums
### EVRSkeletalTransformSpace
Options for the coordinate space that bone transforms should be provided in
* `VRSkeletalTransformSpace_Model`: The bone transforms have been fully concatenated with their parent bones, and are all in same the coordinate space as the model which they would animate
* `VRSkeletalTransformSpace_Parent`: The bone transforms are in the local coordinate space of their parent bone


### EVRSkeletalMotionRange
* `VRSkeletalMotionRange_WithController`: The range of motion of the skeleton takes into account any physical limits imposed by the controller itself.  This will tend to be the most accurate pose compared to the user's actual hand pose, but might not allow a closed fist for example
* `VRSkeletalMotionRange_WithoutController`: Retarget the range of motion provided by the input device to make the hand appear to move as if it was not holding a controller.  eg: map "hand grasping controller" to "closed fist"


### EVRSkeletalReferencePose
* `VRSkeletalReferencePose_BindPose`
* `VRSkeletalReferencePose_OpenHand`
* `VRSkeletalReferencePose_Fist`
* `VRSkeletalReferencePose_GripLimit`


### EVRFinger
* `VRFinger_Thumb`
* `VRFinger_Index`
* `VRFinger_Middle`
* `VRFinger_Ring`
* `VRFinger_Pinky`

### EVRFingerSplay
* `VRFingerSplay_Thumb_Index`
* `VRFingerSplay_Index_Middle`
* `VRFingerSplay_Middle_Ring`
* `VRFingerSplay_Ring_Pinky`

### EVRSkeletalTrackingLevel
* `VRSkeletalTracking_Estimated`: body part location can’t be directly determined by the device. Any skeletal pose provided by the device is estimated by assuming the position required to active buttons, triggers, joysticks, or other input sensors. E.g. Vive Controller, Gamepad
* `VRSkeletalTracking_Partial`: body part location can be measured directly but with fewer degrees of freedom than the actual body part. Certain body part positions may be unmeasured by the device and estimated from other input data. E.g. Knuckles, gloves that only measure finger curl
* `VRSkeletalTracking_Full`: body part location can be measured directly throughout the entire range of motion of the body part. E.g. Mocap suit for the full body, gloves that measure rotation of each finger segment

## Structs

### InputSkeletalActionData_t
* `bool bActive`: Whether or not this action is bound to an input source that is present in the system and is in an action set that is active from the last UpdateActionState call.
* `VRInputValueHandle_t activeOrigin`: The input source that this action state was generated by.  If this action is bound to multiple inputs, this will be the input that changed most recently.


### VRSkeletalSummaryData_t
Contains summary information about the current skeletal pose
* `float flFingerCurl[ VRFinger_Count ]`: The amount that each finger is 'curled' inwards towards the palm.  In the case of the thumb, this represents how much the thumb is wrapped around the fist.  0 means straight, 1 means fully curled
* `float flFingerSplay[ VRFingerSplay_Count ]`: The amount that each pair of adjacent fingers are separated.  0 means the digits are touching, 1 means they are fully separated.


## Functions

### GetSkeletalActionData
```
EVRInputError GetSkeletalActionData( VRActionHandle_t action, InputSkeletalActionData_t *pActionData, uint32_t unActionDataSize, VRInputValueHandle_t ulRestrictToDevice )
```
Reads the state of the skeletal action.  

* `action` - The handle of the skeletal action to check the status of
* `pActionData` - Pointer to the InputSkeletalActionData_t struct that the function should fill in
* `unActionDataSize` - The size in bytes of the InputSkeletalActionData_t struct
* `ulRestrictToDevice` - If the action can be bound to more than one device, this should be the device to restrict the query to


### GetSkeletalBoneData
```
EVRInputError GetSkeletalBoneData( VRActionHandle_t action, EVRSkeletalTransformSpace eTransformSpace, EVRSkeletalMotionRange eMotionRange, VRBoneTransform_t *pTransformArray, uint32_t unTransformArrayCount, VRInputValueHandle_t ulRestrictToDevice )
```
Retrieve the bone transform data.  The bones will be in SteamVR's units and coordinate system; see [Hand Skeleton](https://github.com/ValveSoftware/openvr/wiki/Hand-Skeleton) for more info. 

* `action` - The handle of the skeletal action to retrieve the bone transforms for
* `eTransformSpace` - The coordinate space that each bone transform should be returned in.  
* `eMotionRange` - The desired range of motion of the animation.  Can be either:
* `pTransformArray` - Pointer to the array of `vr::VRBoneTransform_t` that the function should put the bone transforms in
* `unTransformArrayCount` - The number of elements in `pTransformArray`
* `ulRestrictToDevice` - If the action can be bound to more than one device, this should be the device to restrict the query to

### GetSkeletalBoneDataCompressed
```
EVRInputError GetSkeletalBoneDataCompressed( VRActionHandle_t action, EVRSkeletalTransformSpace eTransformSpace, EVRSkeletalMotionRange eMotionRange, void *pvCompressedData, uint32_t unCompressedSize, uint32_t *punRequiredCompressedSize, VRInputValueHandle_t ulRestrictToDevice )
```
Retrieve the bone transforms as a compressed binary blob.

* `action` - The handle of the skeletal action to retrieve the bone data for
* `eTransformSpace' - The coordinate space that each bone transform should be returned in. See `GetSkeletalBoneData()` for details
* `eMotionRange` - The desired range of motion of the animation.  See `GetSkeletalBoneData()` for details
* `pvCompressedData` - Pointer to a buffer where the compressed data should be placed
* `unCompressedSize` - The size of the buffer pointed to by pvCompressedData
* `punRequiredCompressedSize` - The size of the compressed data.  If this is less than unCompressedSize the function will return an error and no transforms will be written
* `ulRestrictToDevice` - If the action can be bound to more than one device, this should be the device to restrict the query to

### DecompressSkeletalBoneData
```
EVRInputError DecompressSkeletalBoneData( void *pvCompressedBuffer, uint32_t unCompressedBufferSize, EVRSkeletalTransformSpace *peTransformSpace, VRBoneTransform_t *pTransformArray, uint32_t unTransformArrayCount )
```
Turn a compressed blob back into bone transforms

* `pvCompressedBuffer` - Pointer to a buffer holding the compressed data
* `unCompressedBufferSize` - The size of the buffer pointed to by pvCompressedBuffer
* `peTransformSpace` - Pointer to EVRSkeletalTransformSpace enum.  The function will write the coordinate space of the compressed buffer here
* `pTransformArray` - Pointer to the array of `vr::VRBoneTransform_t` that the function should put the bone transforms in
* `unTransformArrayCount` - The number of elements in `pTransformArray`.  If there are not enough elements, the function will return an error




