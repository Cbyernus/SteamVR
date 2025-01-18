注意：此頁面最初是提供給 Lighthouse 硬體供應商的 JSON 檔案的移植。

# JSON文件

### SteamVR™ 追蹤器

## 介紹

SteamVR™追蹤系統中的每個被追蹤對像都包含一個文件，該文件描述其感測器幾何形狀以及有關設備的其他重要數據。該文件是使用 JSON 文件格式編寫的。儘管許多檔案使用 JSON 格式，但儲存在追蹤物件中的檔案對於物件的開發和效能至關重要，因此被稱為「JSON 檔案」。 JSON 檔案以最少的感測器位置和方向開始其生命週期，但在整個設計和整合過程中不斷增強，包括 IMU 資料、鏡頭畸變資料以及 SteamVR™ 使用的其他元資料。最後，校準例程將原始感測器幾何值細化為特定物件上的精確感測器位置，並重寫 JSON 文件，從而為每個追蹤物件產生唯一的 JSON 檔案。

本文檔描述了可能儲存在 JSON 檔案中的所有變數、它們的含義以及如何指定它們。本文檔引用自其他描述設計過程中各個步驟的文件。若要了解每個變數何時、為何以及如何新增至 JSON 檔案中，請遵循物件設計和整合概述中概述的流程。

## JSON格式

JSON 代表 JavaScript 物件表示法，是一種輕量級資料交換格式。資料交換格式是格式化資料以在不同運算平台（有時是人類讀者）之間共享的方法。有關 JSON 格式的完整說明，請造訪 www.json.org。 由於 JSON 檔案表示 JSON 對象，因此它以大括號 { 開頭，並以大括號 } 結束。 JSON 檔案中儲存的不同成員成對存儲，並使用逗號分隔。每對都由一個字串標識，該字串與其值之間用冒號分隔，「名稱」：值。下面描述了追蹤物件 JSON 檔案的有效成員，並在文件末尾提供了完整的 JSON 檔案作為範例。

提示：像 Notepad++ 這樣的免費原始碼編輯器可以理解 JSON 格式，並提供語法突出顯示和程式碼折疊等便利功能。



## JSON 構件

**“manufacturer”**

表示製造商的公司名稱的字串值。

例子：

`"manufacturer" : "Valve`

**“model_number”**
表示物件型號的字串值，由製造商分配。

例子：

`"model_number" : "REF-HMD"`

**“device_class”**

可以設定為“controlle”,“generic_tracker”或“hmd”等字串值
將 device_class 設定為「controller」告訴 SteamVR™ 該追蹤設備是VR控制器。
將值設為“generic_tracke”,告訴 SteamVR™ 將追蹤設備為追蹤器。
將值設為“hmd”,告訴 SteamVR™ 將該追蹤設備與雙眼顯示器關聯起來，並使用該設備姿勢作為虛擬實境的觀點。

例子：

`"device_class" : "hmd`

**“device_vid”**

USB 供應商識別號碼。每個製造商都應向 USB.org 請求一個 VID。用於原型設計
出於目的，請使用如下所示的 Valve VID。

例子：

`"device_vid" : 10462`

**“device_pid”**

USB 產品識別號碼。每個製造商都應該為每個型號建立一個 PID。用於原型設計
出於目的，請使用如下所示的 PID。

例子：

`"device_pid" : 8960`

**“device_serial_number”**

表示設備的唯一序號的字串。
該值僅用作將 JSON 檔案與實際物件進行匹配的參考。每個物件創建自己的序列
物件處理器中序號的編號。將該序號寫入 JSON 檔案允許
開發人員將 JSON 檔案與實際物件關聯起來。維持這種聯繫尤為重要
光學和 IMU 校準後。物件產生的序號顯示為
燈塔_控制台。

例子：

`"device_serial_number" : "LHR-F8DE9EBE"`

**“lighthouse_config”**

包含三個成員數組的物件。每個陣列索引對應於被追蹤物件上的一個感測器。
指定 modelPoints、modelNormals 和 channelMap 中的值定義了
感測器、其方向及其電氣連接（連接埠號）。當 SteamVR™ 接收追蹤資料時
從特定連接埠上的感測器，它可以使用 lighthouse_config 中的資料將該訊號與
位於被追蹤物體上的特定物理感測器。 SteamVR™ 也會讀取所描述的感測器位置
在 lighthouse_config 中為裝置建立精確的感測器幾何形狀。 SteamVR™解決傳入
根據已知的感測器幾何形狀追蹤數據，以確定物體的當前姿態。

![image](https://user-images.githubusercontent.com/3059423/181827611-c08d4311-909b-4071-b5a9-802791bf8398.png)

例子： (Replace ellipsis with arrays described in each member below.)

```
"lighthouse_config" : {
 "channelMap" : [...],
 "modelNormals" : [...],
 "modelPoints" : [...]
 }
```

_“modelNormals”_

[x, y, z] 單位向量數組，指定光學感測器在物件座標中面向的方向
系統。

每個光電二極體都放置在物體朝外的表面上。想像一下下圖所示的感測器
以 -X 軸和 +Y 軸之間的 45° 角向外。

![image](https://user-images.githubusercontent.com/3059423/181828013-75a75a00-f3cd-49b9-9201-3f2394c0ed31.png)

垂直於光電二極體表面的單位向量指向與 +X 軸成 135° 的方向，與 +X 軸成 45° 的方向。
+Y 軸。因此，描述該方向的單位向量將為 [ -0.7071, 0.7071, 0.0 ]。如果我們有
將其他感測器放置在 45° 面上，我們預計會出現以下法線。

例子：

`"modelNormals" : [`
 `[ -0.7071, 0.7071, 0.0 ],`
 `[ -0.7071, 0.0, -0.7071 ],`
 `[ -0.7071, 0.0, 0.7071 ],`
 `[ -0.7071, 0.0, -0.7071 ],`
 `[ -0.7071, -0.7071, 0.0 ]`
 `]`

_“modelPoints”_

An array of [x, y, z] coordinates that specify the location of the center of the optical sensor’s photosensitive area
in the object’s coordinate system. Coordinate values are expressed in meters.

If we assigned the following dimensions to the sensor locations on the side of the HMD, we would need to
represent those dimensions in the modelPoints array as shown below.

![image](https://user-images.githubusercontent.com/3059423/181828159-306c3041-bf33-4a86-8f2a-74f364ed2338.png)

例子：
```
"modelPoints" : [
 [ -0.055, 0.030, 0.015 ],
 [ -0.050, 0.015, 0.005 ],
 [ -0.050, 0.0, 0.025 ],
 [ -0.050, -0.015, 0.005 ],
 [ -0.055, -0.030, 0.015 ]
 ]
```

_“channelMap”_

An array of port numbers. There is an element in the array for each sensor on the object. The value in the array
element corresponds to the electrical channel connected to the sensor. When SteamVR™ receives tracking
data from a channel, it can use this array to map that data to a sensor location specified in modelPoints.

If the five sensors shown on the side of the HMD in this example were connected to the electrical channels as
indicated below, we would need the channelMap array [3, 5, 7, 11, 13]. The sensor channel number is
determined by the sensor’s electrical connection to the object’s FPGA.

![image](https://user-images.githubusercontent.com/3059423/181828261-c138c653-2775-4b0e-8852-e2a00b1b6c53.png)

**Note**: _The channel numbers are zero based, and cover the range 0 - 31. Altium’s multichannel
schematic feature forces numbering to start at 1. Be careful when copying channel numbers from the
net names in Altium. Net SENSOR_X1 is most likely connected to FPGA channel number 0. Subtract
1 from the reference designator to get the correct channel number._

**“head”**

An object with members that orient the tracked object’s coordinate system to the real world. The head member
has two different meanings, depending on the value of device_class.

_HMD Use Case_

When the object is acting as an HMD, the head variable orients SteamVR™’s coordinate system to the tracked
object. The origin of SteamVR™ is a point between an average user’s pupils, oriented with +Y pointing up, +X
pointing to the user’s right, and -Z pointing out along the line of sight. If the model has a coordinate system that
does not match this, then changing the head member is required.

![image](https://user-images.githubusercontent.com/3059423/181828580-a70ae58a-49c7-411c-aa64-80c50b37b3e5.png)

For example, consider the HMD object below, with its coordinate system.

![image](https://user-images.githubusercontent.com/3059423/181828618-95fe6d31-35a1-4dc3-846c-488a29e61f64.png)

The “plus_x” value answers the question, “The head’s +X axis is pointing in which direction within the HMD’s
CAD coordinate system?” The head’s +X axis points in the direction of the HMD’s -X axis. Therefore, the value
for “plus_x” is [-1, 0, 0].

Likewise, the +Z axis of the head’s coordinate system points in the direction of the -Z axis of the HMD’s
coordinate system. This requires a “plus_z” value of [0, 0, -1] to match the orientations.

The head’s origin is centered between the pupils, but the HMD’s origin is between the HMD’s lenses. When
worn, the origin of the head is behind the origin of the HMD by 20 mm along the -Z axis. Matching the origins
requires a value of [0.0, 0.0, -0.020] for the “position” coordinates.

例子：
```
"head" : {
 "plus_x" : [ -1, 0, 0 ],
 "plus_z" : [ 0, 0, -1 ],
 "position" : [ 0.0, 0.0, -0.020 ]
}
```

_Controller_

When the object is acting as a controller, the head variable orients the render model shown in SteamVR™ to
the tracked object. The render model is created in the SteamVR™ coordinate system as described in **The**
**Render Model**. One method for determining the head variable is to export the render model as an STL file,
import that STL file into the 3D CAD space of the object. Then, measure the plus_x and plus_z normals and
origin of the render model from the origin of the object.

例子：

```
"head" : {
 "plus_x": [1, 0, 0],
 "plus_z": [0, 0.05233595624, 0.99862953475],
 "position": [0, 0.015, -0.040]
}
```

_“plus_x”_

HMD: A unit vector [x, y, z] representing the direction of the head’s +X axis in the object’s coordinate system.

Controller: A unit vector [x, y, z] that aligns the render model in SteamVR™ to the object’s coordinate system.

_“plus_z”_

HMD: A unit vector [x, y, z] representing the direction of the head’s +Z axis in the object’s coordinate system.

Controller: A unit vector [x, y, z] that aligns the render model in SteamVR™ to the object’s coordinate system.

_“position”_

HMD: A coordinate [x, y, z] located at the midpoint between the average user’s physical pupil locations. This is
typically determined by extending a vector out from the lens surface along the primary optical axis, until it
crosses the plane of the average user’s pupil depth.

Controller: A coordinate [x, y, z] that positions the render model in SteamVR™ in the object’s coordinate
system.

**Note**: _A “plus_y” member is not required, because a right-handed coordinate system is assumed._

**"imu"**

An object containing calibration and orientation data about the IMU. The plus_x, plus_z, and position members
map the IMU coordinate system into the object’s coordinate system. The acc and gyro members hold
accelerometer and gyroscope calibration data. For an example, consider the following IMU, placed on a PCB in
an HMD object.

![image](https://user-images.githubusercontent.com/3059423/181829693-b7846937-f4d4-4f92-a483-e525c91587e7.png)

The datasheet specifies the coordinate system of the IMU. However, that coordinate system is not aligned with
the coordinate system of the HMD. In fact, all three axes axes need adjustment. The IMU’s +X axis points in the
HMD’s +Y direction, requiring a plus_x value of [0, 1, 0]. The IMU’s +Z axis points in the HMD’s -Z direction,
requiring a plus_z value of [0, 0, -1]. The IMU’s Y axis is also different from the HMD, but specifying the X and Z
axes constrains the Y axis.

![image](https://user-images.githubusercontent.com/3059423/181829741-d705b84f-66f5-41d5-956c-8d2692e64f45.png)

Not only are the axes of the IMU different, but the IMU is not located at the origin of the HMD’s coordinate
system. The IMU is located in the X/Y plane, meaning that the Z offset is 0 mm, but the X offset is -40 mm and
the Y offset is +10 mm. Therefore, the required position value is [-0.040, 0.010, 0.0].

The accelerometer and gyroscope have offsets inherent in the device. Accelerometer offsets, for example, may
start as low as 0.025g. However, they increase during the assembly process, and throughout the life of the part,
to as high as 0.150g or more. Initial gyroscope offsets are typically 3°/s - 5°/s. The accelerometer and
gyroscope offsets change over temperature. Accounting for that, SteamVR™ has internal calibration
mechanisms that constantly trim out the error during use. To ensure that SteamVR™ can converge on the
actual offsets as quickly as possible, the IMU member of the JSON file holds initial calibration data for offsets
present at the time of manufacturing. Determining the correct values for these fields is easily accomplished using the software utility described in IMU Calibration. The **IMU calibration** tool outputs a snippet of JSON to
copy and paste into the IMU member of the JSON file.

IMU Calibrator Output:
```
Calibrating to gravity sphere, radius 9.8066
0.05265 accelerometer fit error (6 sample vectors x 8 subsamples per vector)
"acc_scale" : [ 0.998, 0.9982, 0.9912 ],
"acc_bias" : [ 0.04646, -0.04264, -0.2414 ],
"gyro_scale" : [ 1.0, 1.0, 1.0 ],
"gyro_bias" : [ 0.06343, 0.01029, -0.02168 ],
```

Example:
```
"imu" : {
 "acc_scale" : [ 0.998, 0.9982, 0.9912 ],
 "acc_bias" : [ 0.04646, -0.04264, -0.2414 ],
 "gyro_scale" : [ 1.0, 1.0, 1.0 ],
 "gyro_bias" : [ 0.06343, 0.01029, -0.02168 ],
 "plus_x" : [ 0, 1, 0 ],
 "plus_z" : [ 0, 0, -1 ],
 "position" : [ -0.040, 0.010, 0.0 ]
 }
```

_“plus_x”_

A unit vector [x, y, z] representing the direction of the IMU’s +X axis in the object’s coordinate system.

_“plus_z”_

A unit vector [x, y, z] representing the direction of the IMU’s +Z axis in the object’s coordinate system.

_“position”_

A coordinate [x, y, z] specifying the center of the IMU’s package in the object’s coordinate system.

**Note**: _A “plus_y” member is not required, because a right-handed coordinate system is assumed._

**"render_model"**

Holds a string value that specifies the name of the subfolder within the SteamVR™ “rendermodels” folder that
holds the default rendermodel for the object.

Example:

`"render_model" : "ref_controller"`

**“display_edid”**

Description

Example:
`"display_edid" : [ "", "" ]`

**“direct_mode_edid_vid”**

The integer display EDID vendor ID. This number must be whitelisted by NVIDIA to enable your display to
operate in direct mode. After that, this value tells SteamVR™ which display to use when displaying VR to the
HMD. Note that any text prefixes are omitted.

Example:

`"direct_mode_edid_vid" : xxxxx`

**“direct_mode_edid_pid”**

The integer display EDID product ID. This number must be whitelisted by NVIDIA to enable your display to
operate in direct mode. After that, this value tells SteamVR™ which display to use when displaying VR to the
HMD. Note that any text prefixes are omitted.

Example:

`"direct_mode_edid_vid" : xxxxx`

*"device"*

Description

Example:

```
"device" : {
 "eye_target_height_in_pixels" : 1080,
 "eye_target_width_in_pixels" : 960,
 "first_eye" : "eEYE_LEFT",
 "last_eye" : "eEYE_RIGHT",
 "num_windows" : 1,
 "persistence" : 0.01666999980807304,
 "physical_aspect_x_over_y" : 0.8000000119209290
 }
```

_“eye_target_height_in_pixels”_

This value indicates the vertical resolution of each eye’s display in the HMD. When operating out of direct
mode, SteamVR uses this value along with eye_target_width_in_pixels to determine which display to associate
with the tracked HMD device.

_“eye_target_width_in_pixels”_

This value indicates the horizontal resolution of each eye’s display in the HMD. Since HMDs split a single
display output feed into two physical displays, this value should be one half of the width of the display as it
appears in your operating system’s display settings. In this example, the HMD’s resolution is 1920x1080 split
between both eyes. When operating out of direct mode, SteamVR uses this value along with
eye_target_width_in_pixels to determine which display to associate with the tracked HMD device.

_“first_eye”_

Description

_“last_eye”_

Description

_“num_windows”_

Description

_“persistence”_

Description

_“physical_aspect_x_over_y”_

Description

**“lens_separation”**

Description

Example:

`"lens_separation" : 0.06230000033974648`

**“tracking_to_eye_transform”**

The lens calibration information is stored in the HMD JSON file in the block “tracking_to_eye_transform”. This
block is a two-element array where the two elements are the descriptions for the left and right eyes respectively.
For each eye’s description there are several sub-blocks, described below.

We assume here that the lens is circularly symmetric and is kept in a fixed position parallel to the display panel.
We will assume a pixel coordinate system with (0,0) in the top left corner of the display, the +X axis extending to
the right, and the +Y axis extending down. Let (w,h) be the horizontal and vertical resolution of the display. Note
that this coordinate system is independent for each eye, not a single coordinate system extending across both
eyes.

Example: (Replace ellipsis with values described in each member below.)
```
"tracking_to_eye_transform" : [
 {
 "distortion" : {...},
 "distortion_blue" : {...},
 "distortion_red" : {...},
 "extrinsics" : [...],
 "grow_for_undistort" : 0.0,
 "intrinsics" : [...],
 "undistort_r2_cutoff" : 1.50
 },
 {
 "distortion" : {...},
 "distortion_blue" : {...},
 "distortion_red" : {...},
 "extrinsics" : [...],
 "grow_for_undistort" : 0.0,
 "intrinsics" : [...],
 "undistort_r2_cutoff" : 1.50
 }
 ],
```

_“intrinsics”_

The “intrinsics” block is a 3x3 matrix describing the linear projection that the lens implements after distortion is
corrected. Five values are populated based on the focal length and the center of projection. These values can
be calculated as follows.

Let f be the focal length of the HMD lens, measured in pixels. Let (cx
 ,cy ) be the location on the panel (in pixel
coordinates) that the HMD lens is centered over. The “intrinsics” block 3x3 matrix is then:

![image](https://user-images.githubusercontent.com/3059423/181830628-6b147b9c-c06b-495f-b3d4-cebbc04cfe04.png)

Example:
```
"intrinsics" : [
 [ 1.250, 0.0, 0.0 ],
 [ 0.0, 1.0, 0.0 ],
 [ 0.0, 0.0, -1.0 ]
 ]
```

_“distortion”, “distortion_blue”, “distortion_red”_

The “distortion” block is information describing any non-linear distortion in the HMD lens. This includes the lens
centering information and a polynomial describing the lens distortion. For each eye, there are actually three
distortion blocks, “distortion”, “distortion_red”, and “distortion_blue”, which provide the distortion information for
each of the green, red, and blue color channels respectively.

Let (cx , cy ) be the location on the panel (in pixel coordinates) that the HMD lens is centered over, as mentioned
above. The "center_x" and "center_y" values in the distortion block can be calculated as:

![image](https://user-images.githubusercontent.com/3059423/181830730-eda4fccc-1ae4-4fa7-b9ee-ac6950982a58.png)

Note that the denominator in both cases is (w/2).

The “type” field specifies the mathematical form of the distortion function, and the “coeffs” field specifies
coefficients for use in that function. Let P be a pixel location on the HMD display panel, and let R be the distance
in pixels between P and the lens center location P. Let f be the focal length of the HMD lens in pixels, as
mentioned above. Let be the angle at which pixel P appears through the lens. For a lens with no distortion,
the relationship between these terms would be:

![image](https://user-images.githubusercontent.com/3059423/181830789-b5c3cb63-7291-457c-96af-ea64aed36b1c.png)

The most common form of distortion function is a function of R (so, radially symmetric) that is applied as a
multiplicative factor. In order to keep the values in a limited range the input to the function is also normalized,
currently by dividing by (w/2). For a distortion function D this gives:

![image](https://user-images.githubusercontent.com/3059423/181830832-1df34bf8-4950-4959-a5a2-fb9d52c444e7.png)

We support several different distortion function types and it is fairly simple to add new types. Two common
examples are: cubic polynomials in the (normalized) radius squared, and rational cubic polynomials in the
(normalized) radius squared:

![image](https://user-images.githubusercontent.com/3059423/181830868-786d1349-7e8c-4d27-9220-80d2f050c2e0.png)

For the first example the “type” field is DISTORT_POLY3, and for the second it is DISTORT_DPOLY3. In both
cases the first three elements of the “coeffs” field are populated with C0 ,C1 ,C2
 and the remaining five elements
are set to 0.0 (the length of the “coeffs” array is currently fixed at eight elements).

例子：
```
"distortion" : {
 "center_x" : 0.0,
 "center_y" : 0.0,
 "coeffs" : [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
 "type" : "DISTORT_DPOLY3"
 }
```

_“extrinsics”_

The “extrinsics” block is a 3x4 matrix describing the rotation and translation of the lens+panel system relative to
the user. For most normal usage this should simply express the relationship between the lens centers due to a
default stereo separation (IPD). If we let S be the default stereo separation in meters, then this matrix is:

![image](https://user-images.githubusercontent.com/3059423/181830947-ad855aab-faff-48cf-ba17-cb9bf5e2db7b.png)

With the upper-right value being positive for the left eye and negative for the right eye.

例子：
```
"extrinsics" : [
 [ 1.0, 0.0, 0.0, -0.03115000016987324 ],
 [ 0.0, 1.0, 0.0, 0.0 ],
 [ 0.0, 0.0, 1.0, 0.0 ]
 ]
```

_“grow_for_undistort”_

The “grow_for_undistort” is not a critical parameter and can (for now) be set to 0.6.

例子：

`"grow_for_undistort" : 0.6`

_“undistort_r2_cutoff”_

The “undistort_r2_cutoff” field is not a critical parameter and can (for now) be set to 1.5.

例子：

_"undistort_r2_cutoff" : 1.50_

**“type”**

A string value “Lighthouse_HMD”

This value is always “Lighthouse_HMD,” even when the device_class is set to controller.

例子：

`"type" : "Lighthouse_HMD"`

# JSON 檔案範例
```
{
 "device" : {
 "eye_target_height_in_pixels" : 1080,
 "eye_target_width_in_pixels" : 960,
 "first_eye" : "eEYE_LEFT",
 "last_eye" : "eEYE_RIGHT",
 "num_windows" : 1,
 "persistence" : 0.01666999980807304,
 "physical_aspect_x_over_y" : 0.8000000119209290
 },
 "device_class" : "controller",
 "device_pid" : 8192,
 "device_serial_number" : "LHR-F8DE9EBE",
 "device_vid" : 10462,
 "display_edid" : [ "", "" ],
 "lens_separation" : 0.06230000033974648,
 "lighthouse_config" : {
 "channelMap" : [ 17, 15, 13, 21, 19 ],
 "modelNormals" : [
 [ 0, 0, -1 ],
 [ -0.13309992849826813, 0.11159992963075638, -0.98479938507080078 ],
 [ 0.11159992963075638, 0.13309992849826813, -0.98479938507080078 ],
 [ 0.13309992849826813, -0.11159992963075638, -0.98479938507080078 ],
 [ -0.11159992963075638, -0.13309992849826813, -0.98479938507080078 ]
 ],
 "modelPoints" : [
 [ -0.0015368221793323755, 0.017447538673877716, -0.0040629836730659008 ],
 [ -0.046612702310085297, 0.039085414260625839, 0.011825915426015854 ],
 [ 0.039518974721431732, 0.046799946576356888, 0.011834526434540749 ],
 [ 0.046315468847751617, -0.038777932524681091, 0.01167147234082222 ],
 [ -0.03922756016254425, -0.046778313815593719, 0.011606470681726933 ]
 ]
 },
 "manufacturer" : "",
 "model_number" : "",
 "render_model" : "lighthouse_ufo",
 "revision" : 3,
 "tracking_to_eye_transform" : [
 {
 "distortion" : {
 "center_x" : 0.0,
 "center_y" : 0.0,
 "coeffs" : [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
 "type" : "DISTORT_DPOLY3"
 },
 "distortion_blue" : {
 "center_x" : 0.0,
 "center_y" : 0.0,
 "coeffs" : [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
 "type" : "DISTORT_DPOLY3"
 },
 "distortion_red" : {
 "center_x" : 0.0,
 "center_y" : 0.0,
 "coeffs" : [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
 "type" : "DISTORT_DPOLY3"
 },
 "extrinsics" : [
 [ 1.0, 0.0, 0.0, 0.03115000016987324 ],
 [ 0.0, 1.0, 0.0, 0.0 ],
 [ 0.0, 0.0, 1.0, 0.0 ]
 ],
 "grow_for_undistort" : 0.0,
 "intrinsics" : [
 [ 1.250, 0.0, 0.0 ],
 [ 0.0, 1.0, 0.0 ],
 [ 0.0, 0.0, -1.0 ]
 ],
 "undistort_r2_cutoff" : 1.50
 },
 {
 "distortion" : {
 "center_x" : 0.0,
 "center_y" : 0.0,
 "coeffs" : [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
 "type" : "DISTORT_DPOLY3"
 },
 "distortion_blue" : {
 "center_x" : 0.0,
 "center_y" : 0.0,
 "coeffs" : [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
 "type" : "DISTORT_DPOLY3"
 },
 "distortion_red" : {
 "center_x" : 0.0,
 "center_y" : 0.0,
 "coeffs" : [ 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
 "type" : "DISTORT_DPOLY3"
 },
 "extrinsics" : [
 [ 1.0, 0.0, 0.0, -0.03115000016987324 ],
 [ 0.0, 1.0, 0.0, 0.0 ],
 [ 0.0, 0.0, 1.0, 0.0 ]
 ],
 "grow_for_undistort" : 0.0,
 "intrinsics" : [
 [ 1.250, 0.0, 0.0 ],
 [ 0.0, 1.0, 0.0 ],
 [ 0.0, 0.0, -1.0 ]
 ],
 "undistort_r2_cutoff" : 1.50
 }
 ],
 "head" : {
 "plus_x" : [ 1, 0, 0 ],
 "plus_z" : [ 0, 0, 1 ],
 "position" : [ 0, 0, 0 ]
 },
 "imu" : {
 "acc_bias" : [ 0, 0, 0 ],
 "acc_scale" : [ 1, 1, 1 ],
 "gyro_bias" : [ 0, 0, 0 ],
 "gyro_scale" : [ 1, 1, 1 ],
 "plus_x" : [ 0, 1, 0 ],
 "plus_z" : [ 1, 0, 0 ],
 "position" : [ 0.0034600000362843275, 0.0013079999480396509,
0.077326998114585876 ]
 },
 "type" : "Lighthouse_HMD"
}
```
