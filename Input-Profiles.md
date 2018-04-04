SteamVR uses driver-provided Input Profile JSON files to control how application actions are bound to the input state in the driver. There is one of these files for each type of device. They generally live in "<driver dir>/resources/input/<device_type>_profile.json". The vive_controller_profile.json file in <steamvr>/drivers/resources/input is a good example to look at.

The driver identifies which file to load via the Prop_InputProfilePath_String property on each device. This string usually takes the form {drivername}/input/filename_profile.json, which will automatically expand {drivername} to the install path of that driver. You can have as many of these files as you need for your driver.

Generally drivers should provide input profiles for each controller and HMD exposed by that driver. SteamVR will generate an approximate profile for devices that don't have a profile provided, but a driver can do a better job than SteamVR itself.

The file looks like this:
```
{
  "jsonid" : "input_profile",
  "controller_type": "<controller type>",
  "legacy_binding" :  "<legacy_binding_file>",
  "input_bindingui_mode" : "<UI mode>",
  "input_bindingui_left" :
  {
    "transform" : "scale(-1,1)",
    "image": "<image path>"
  },
  "input_bindingui_right" :
  {
    "image": "<image path>"
  },
  "input_source" :
  {
    "<input source path>" : { 
        "type" : "<input source type>",
        "binding_image_point" : [ 10, 59 ],
        "order" : 2
    },
    ...
    "/pose/raw" : {
        "type" : "pose",
        "binding_image_point" : [ 14, 16 ]
    },
    ...
  }
}
```

**controller type** is the key that's used to store bindings per-app for this controller. Including your driver name or brand name in the controller type is a good idea so it doesn't collide with controller types from other drivers. Some examples of controller types:

* vive
* vive_pro
* vive_controller
* gamepad

**legacy binding file** is the path to the binding file to use for legacy apps whenever this controller is active. It is the same format as other binding files, and can be built with the binding UI and then exported. (The binding UI isn't actually available yet as of this writing, but it's coming soon.)  These paths are usually driver relative and take this form: {drivername}/input/mycontroller_legacy_bindings.json.

**UI mode** - This is one of:

* hmd - The device is an HMD and wants the UI to be in HMD mode
* single_device - The device should be on a page by itself
* controller_handed - The device is in the right and/or left hand and should be shown in a pair in the UI.

**image path** - This is the path to the driver-relative SVG or PNG file that the binding UI should show for this device. This is generally of the form {drivername}/input/mycontroller.svg.  

**input source path** - There is one input source specified for each input on the device. Individual components are grouped into input sources to allow the user to manipulate them as a group. For example, the input source /input/trigger contains the input components /input/trigger/value and optionally /input/trigger/click, and /input/trigger/touch.

**input source type** - Each input source has a type. The available types are:
* joystick - The input is a joystick or thumbstick. Supports these components:
  * /input/<joystickname>/x
  * /input/<joystickname>/y
  * /input/<joystickname>/click - Optional. If there is no click component the joystick won't support binding to click.
  * /input/<joystickname>/touch - Optional. If there is no touch component the joystick won't support binding to touch.
* trackpad - The input is a trackpad. It supports these components:
  * /input/<trackpadname>/x
  * /input/<trackpadname>/y
  * /input/<trackpadname>/click
  * /input/<trackpadname>/touch
* trigger - The input is a trigger (or an analog grip button.) It supports these components:
  * /input/<trigger name>/value
  * /input/<trigger name>/click - Optional
  * /input/<trigger name>/touch - Optional
* button - The input is a button. It supports these components:
  * /input/<trigger name>/click
  * /input/<trigger name>/touch - Optional
* skeleton - The input is a skeletal animation source
  * /input/<component name>/
* haptic - The input is a haptic component
  * /output/<component name>/
* pose - The input is a pose component. These are created automatically from the components in the device's rendermodel.
  * /pose/<component name>/


**binding_image_point** is the 2D position on the provided image that should be highlighted for this source.

**order** is an optional setting that allows the input profile to control the order of the input sources in the UI.
