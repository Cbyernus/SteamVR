注意：此頁面最初是提供給 Lighthouse 硬體供應商的 JSON 檔案的移植。

# JSON文件

### SteamVR™ 追蹤器

## 介紹

SteamVR™追蹤系統中的每個被追蹤對像都包含一個文件，該文件描述其感測器幾何形狀以及有關設備的其他重要數據。該文件是使用 JSON 文件格式編寫的。
儘管許多檔案使用 JSON 格式，但儲存在追蹤物件中的檔案對於物件的開發和效能至關重要，因此被稱為「JSON 檔案」。 JSON 檔案以最少的感測器位置和方向開始其生命週期，但在整個設計和整合過程中不斷增強，包括 IMU 資料、鏡頭畸變資料以及 SteamVR™ 使用的其他元資料。
最後，校準例程將原始感測器幾何值細化為特定物件上的精確感測器位置，並重寫 JSON 文件，從而為每個追蹤物件產生唯一的 JSON 檔案。

本文檔描述了可能儲存在 JSON 檔案中的所有變數、它們的含義以及如何指定它們。本文檔引用自其他描述設計過程中各個步驟的文件。若要了解每個變數何時、為何以及如何新增至 JSON 檔案中，請遵循物件設計和整合概述中概述的流程。

## JSON格式

JSON 代表 JavaScript 物件表示法，是一種輕量級資料交換格式。資料交換格式是格式化資料以在不同運算平台（有時是人類讀者）之間共享的方法。有關 JSON 格式的完整說明，請造訪 www.json.org 。
由於 JSON 檔案表示 JSON 對象，因此它以大括號 { 開頭，並以大括號 } 結束。 JSON 檔案中儲存的不同成員成對存儲，並使用逗號分隔。每對都由一個字串標識，該字串與其值之間用冒號分隔，「名稱」：值。
下面描述了追蹤物件 JSON 檔案的有效成員，並在文件末尾提供了完整的 JSON 檔案作為範例。

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
光學和 IMU 校準後。物件產生的序號顯示為燈塔_控制台。

例子：

`"device_serial_number" : "LHR-F8DE9EBE"`

**“lighthouse_config”**

包含三個成員數組的物件。每個陣列索引對應於被追蹤物件上的一個感測器。
指定 modelPoints、modelNormals 和 channelMap 中的值定義了感測器、其方向及其電氣連接（連接埠號）。
當 SteamVR™ 接收追蹤資料時從特定連接埠上的感測器，它可以使用 lighthouse_config 中的資料將該訊號與位於被追蹤物體上的特定物理感測器。 
SteamVR™ 也會讀取所描述的感測器位置在 lighthouse_config 中為裝置建立精確的感測器幾何形狀。 
SteamVR™解決傳入根據已知的感測器幾何形狀追蹤數據，以確定物體的當前姿態。

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

[x, y, z] 座標數組，指定光學感測器感光區域中心位置
在物體的座標系中。座標值以米表示。

如果我們將以下尺寸分配給 HMD 側面的感測器位置，我們需要
在 modelPoints 陣列中表示這些維度，如下所示。

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

連接埠號碼數組。物件上的每個感測器在數組中都有一個元素。數組中的值
元件對應連接到感測器的電通道。當 SteamVR™ 收到追蹤時
如果來自通道的數據，它可以使用此數組將該資料對應到 modelPoints 中指定的感測器位置。

如果本範例中 HMD 側面顯示的五個感測器連接到電氣通道，如下所示
如下圖所示，我們需要channelMap陣列[3,5,7,11,13]。感測器通道數為
由感測器與物件 FPGA 的電氣連接確定。

![image](https://user-images.githubusercontent.com/3059423/181828261-c138c653-2775-4b0e-8852-e2a00b1b6c53.png)

**注意**：_通道編號從零開始，涵蓋範圍 0 - 31。
原理圖功能強制編號從 1 開始。
Altium 中的網路名稱。 Net SENSOR_X1 最有可能連接到 FPGA 頻道號碼 0。
1 從參考指示符取得正確的頻道號碼。

**“head”**

一個對象，其成員將追蹤對象的座標系定向到現實世界。團長成員
有兩種不同的意義，取決於 device_class 的值。

_HMD Use Case_

當追蹤設備充當 HMD 時， head 變數將 SteamVR™ 的座標系定向到追蹤的位置目的。 SteamVR™ 的原點是一般使用者瞳孔之間的點，方向為 +Y 向上，+X
指向使用者的右側，-Z 沿著視線方向指向。如果模型有一個座標系與此不符，則需要更換頭部部件。

![image](https://user-images.githubusercontent.com/3059423/181828580-a70ae58a-49c7-411c-aa64-80c50b37b3e5.png)

例如，考慮下面的 HMD 物件及其座標系。

![image](https://user-images.githubusercontent.com/3059423/181828618-95fe6d31-35a1-4dc3-846c-488a29e61f64.png)

“plus_x”值回答了這個問題：“頭部的+X軸在HMD的CAD坐標系中指向哪個方向？”頭部的+X軸指向HMD的-X軸。因此，“plus_x”的值為[-1, 0, 0]。

同樣，頭部坐標系的+Z軸指向HMD坐標系的-Z軸。這需要“plus_z”值為[0, 0, -1]來對齊方向。

頭部的原點位於兩個瞳孔之間，而HMD的原點位於HMD鏡頭之間。佩戴時，頭部的原點在-H軸上比HMD的原點向後20毫米。要對齊原點，"位置"坐標需要的值為[0.0, 0.0, -0.020]。

例子：
```
"head" : {
 "plus_x" : [ -1, 0, 0 ],
 "plus_z" : [ 0, 0, -1 ],
 "position" : [ 0.0, 0.0, -0.020 ]
}
```

_Controller_

當物體作為控制器時，頭部變量將SteamVR™中顯示的渲染模型定向至被追蹤的物體。渲染模型是在SteamVR™坐標系中創建的，如渲染模型中所述。確定頭部變量的一種方法是將渲染模型導出為STL文件，然後將該STL文件導入物體的3D CAD空間中。接著，從物體的原點測量渲染模型的plus_x和plus_z法向量及原點位置。

例子：

```
"head" : {
 "plus_x": [1, 0, 0],
 "plus_z": [0, 0.05233595624, 0.99862953475],
 "position": [0, 0.015, -0.040]
}
```

_“plus_x”_

HMD：一個單位向量 [x, y, z]，表示頭部的+X軸在物體坐標系中的方向。

控制器：一個單位向量 [x, y, z]，使SteamVR™中的渲染模型與物體的坐標系對齊。

_“plus_z”_

HMD：單位向量[x,y,z]，表示物體座標系中頭部+Z軸的方向。

控制器：一個單位向量 [x, y, z]，將 SteamVR™ 中的渲染模型與物件的座標系對齊。

_“position”_

HMD：座標 [x, y, z] 位於一般使用者物理瞳孔位置之間的中點。
這是通常透過沿著主光軸從透鏡表面延伸向量來確定，直到它
穿過普通用戶瞳孔深度的平面。

控制器：座標 [x, y, z]，將 SteamVR™ 中的渲染模型定位在物件的座標中系統。

**注意**：_不需要「plus_y」成員，因為假定使用右手座標系。

**"imu"**

包含有關 IMU 的校準和方向資料的構件。 plus_x、plus_z 和position 構件將 IMU 座標系映射到追蹤設備的座標系。 
ACC 和陀螺儀成員持有加速度計和陀螺儀校準資料。
例如，考慮以下圖片，一塊帶有IMU的PCB放置在 HMD 追蹤設備。

![image](https://user-images.githubusercontent.com/3059423/181829693-b7846937-f4d4-4f92-a483-e525c91587e7.png)

數據表指定了IMU的坐標系。然而，該坐標系並未與HMD的坐標系對齊。
事實上，所有三個軸都需要調整。IMU的+X軸指向HMD的+Y方向，因此需要調整plus_x值為[0, 1, 0]。IMU的+Z軸指向HMD的-Z方向，因此需要調整plus_z值為[0, 0, -1]。IMU的Y軸也不同於HMD，但指定X軸和Z軸會約束Y軸的方向。

![image](https://user-images.githubusercontent.com/3059423/181829741-d705b84f-66f5-41d5-956c-8d2692e64f45.png)

不僅IMU的軸不同，IMU也不位於HMD坐標系的原點。IMU位於X/Y平面上，這意味著Z偏移為0毫米，但X偏移為-40毫米，Y偏移為+10毫米。因此，所需的位置信息為[-0.040, 0.010, 0.0]。

加速度計和陀螺儀具有設備固有的偏移。例如，加速度計的偏移可能最低為0.025g。然而，這些偏移在組裝過程中會增加，並且在部件的整個使用壽命中可能增至0.150g或更多。初始的陀螺儀偏移通常為3°/s至5°/s。加速度計和陀螺儀的偏移會隨著溫度變化。考慮到這一點，SteamVR™擁有內部校準機制，能夠在使用過程中不斷消除這些誤差。為了確保SteamVR™能夠儘快收斂到實際的偏移值，JSON文件中的IMU成員保存了製造時的初始校準數據。確定這些字段的正確值可以使用IMU校準中描述的軟件工具輕鬆完成。IMU校準工具會輸出一段JSON片段，以便複製並粘貼到JSON文件的IMU成員中。

IMU校準器輸出：
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

"plus_x"
一個單位向量 [x, y, z]，表示IMU的+X軸在物體坐標系中的方向。

"plus_z"
一個單位向量 [x, y, z]，表示IMU的+Z軸在物體坐標系中的方向。

"position"
一個坐標 [x, y, z]，指定IMU封裝的中心在物體坐標系中的位置。

注意：不需要“plus_y”成員，因為假定使用的是右手坐標系。

"render_model"
"render_model"成員通常包含IMU的渲染模型數據，這些數據用於在虛擬環境中正確顯示IMU的外觀和位置。這可以包括模型的幾何形狀、材質和其他視覺屬性。

例子：

`"render_model" : "ref_controller"`

**“display_edid”**

描述

Example:
`"display_edid" : [ "", "" ]`

**“direct_mode_edid_vid”**

整數顯示 EDID 供應商 ID。該號碼必須被 NVIDIA 列入白名單，以便您的顯示器能夠
以直接模式運轉。之後，該值告訴 SteamVR™ 在向使用者顯示 VR 時使用哪個顯示器
頭顯。請注意，省略任何文字前綴。

例子：

`"direct_mode_edid_vid" : xxxxx`

**“direct_mode_edid_pid”**

整數顯示EDID產品ID。該號碼必須被 NVIDIA 列入白名單，以便您的顯示器能夠
以直接模式運轉。之後，該值告訴 SteamVR™ 在向使用者顯示 VR 時使用哪個顯示器
頭顯。請注意，省略任何文字前綴。

例子：

`"direct_mode_edid_vid" : xxxxx`

*"device"*

描述
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

該值表示 HMD 中每隻眼睛顯示的垂直解析度。在直連模式下，SteamVR 使用該值以及 eye_target_width_in_pixels 來決定要關聯的顯示器與被追蹤的 HMD 設備。

_“eye_target_width_in_pixels”_

該值表示 HMD 中每隻眼睛顯示的水平解析度。由於 HMD 分割了一個顯示輸出饋送到兩個實體顯示器，該值應該是顯示器寬度的一半，因為它出現在作業系統的顯示設定中。在此範例中，HMD 的分辨率為 1920x1080 分割雙眼之間。
當脫離直接模式運作時，SteamVR 使用該值以及eye_target_width_in_pixels 決定哪個顯示器與追蹤的 HMD 裝置關聯。

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
