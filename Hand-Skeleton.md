# Overview 

The [Skeletal Input](https://github.com/ValveSoftware/openvr/wiki/SteamVR-Skeletal-Input) system allows applications to pick which from a set of pre-defined hand skeletons to receive animation data for from SteamVR.  The definitions of these skeletons are fixed, so that developers can rely on the properties of the skeleton not changing once their app is released.  With that in mind, the choices made about how the skeleton should be configured have been made in an attempt to provide maximum compatibility with existing game engines while also providing extra features that attempt to make it future proof.  

Both left and right hand skeletons have the following properties:
* They have the same number of bones (31)
* There is only one root bone.  It is intended to not translate or rotate, so that it can be constrained to the reference pose of the controller that is animating the skeleton
* The system is designed to avoid any kind of scale; animating scaling can be complicated in real-time engines and is not supported in some cases.  The API only return translations and rotations of bones
* No twist bones
* There are several helper bones, but that are not intended to be directly involved in skinning your hand mesh


# Bone Structure
Both left and right hand skeletons have 31 bones, which are arranged as seen here

![](https://steamcdn-a.akamaihd.net/steamcommunity/public/images/clans/5519564/332a809f2c6defffb2a8a9197d5a648aa17a472b.png)

For convenience, here's an enum defining the bone indices:

```
typedef int32_t BoneIndex_t;
const BoneIndex_t INVALID_BONEINDEX = -1;
enum HandSkeletonBone : BoneIndex_t
{
	eBone_Root = 0,
	eBone_Wrist,
	eBone_Thumb0,
	eBone_Thumb1,
	eBone_Thumb2,
	eBone_Thumb3,
	eBone_IndexFinger0,
	eBone_IndexFinger1,
	eBone_IndexFinger2,
	eBone_IndexFinger3,
	eBone_IndexFinger4,
	eBone_MiddleFinger0,
	eBone_MiddleFinger1,
	eBone_MiddleFinger2,
	eBone_MiddleFinger3,
	eBone_MiddleFinger4,
	eBone_RingFinger0,
	eBone_RingFinger1,
	eBone_RingFinger2,
	eBone_RingFinger3,
	eBone_RingFinger4,
	eBone_PinkyFinger0,
	eBone_PinkyFinger1,
	eBone_PinkyFinger2,
	eBone_PinkyFinger3,
	eBone_PinkyFinger4,
	eBone_Aux_Thumb,
	eBone_Aux_IndexFinger,
	eBone_Aux_MiddleFinger,
	eBone_Aux_RingFinger,
	eBone_Aux_PinkyFinger,
	eBone_Count
};
```



The currently supported skeletons are `/skeleton/hand/left` and `/skeleton/hand/right`.  


# Hierarchy