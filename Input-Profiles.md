# What is the Input Profile

SteamVR uses driver-provided Input Profile JSON files to control how application actions are bound to the input state in the driver. There is one of these files for each type of device. They generally live in "_driver dir_/resources/input/<device_type>_profile.json". The vive_controller_profile.json file in _steamvr install dir_/drivers/resources/input is a good example to look at.

The driver identifies which file to load via the Prop_InputProfilePath_String property on each device. This string usually takes the form {drivername}/input/filename_profile.json, which will automatically expand {drivername} to the install path of that driver. You can have as many of these files as you need for your driver.

Generally drivers should provide input profiles for each controller and HMD exposed by that driver. SteamVR will generate an approximate profile for devices that don't have a profile provided, but a driver can do a better job than SteamVR itself.

# Input Profile Format

```
{
  "jsonid" : "input_profile",
  "controller_type": "<controller type>",
  "legacy_binding" :  "<legacy binding file>",
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

Controller types should be ASCII and short. This is not the human-readable name, it is the internal name that will be stored in data and log files.

**legacy binding file** is the path to the binding file to use for legacy apps whenever this controller is active. It is the same format as other binding files, and can be built with the binding UI and then exported. (The binding UI isn't actually available yet as of this writing, but it's coming soon.)  These paths are usually driver relative and take this form: {drivername}/input/mycontroller_legacy_bindings.json.

**UI mode** - This is one of:

* hmd - The device is an HMD and wants the UI to be in HMD mode
* single_device - The device should be on a page by itself
* controller_handed - The device is in the right and/or left hand and should be shown in a pair in the UI.

**image path** - This is the path to the driver-relative SVG or PNG file that the binding UI should show for this device. This is generally of the form {drivername}/input/mycontroller.svg.  

**transform** - This string is passed to the CSS transform field in the binding UI.

**input source path** - There is one input source specified for each input on the device. Individual components are grouped into input sources to allow the user to manipulate them as a group. For example, the input source /input/trigger contains the input components /input/trigger/value and optionally /input/trigger/click, and /input/trigger/touch.

**input source type** - Each input source has a type. The available types are:
* joystick - The input is a joystick or thumbstick. Supports these components:
  * /input/_joystickname_/x
  * /input/_joystickname_/y
  * /input/_joystickname_/click - Optional. If there is no click component the joystick won't support binding to click.
  * /input/_joystickname_/touch - Optional. If there is no touch component the joystick won't support binding to touch.
* trackpad - The input is a trackpad. It supports these components:
  * /input/_trackpadname_/x
  * /input/_trackpadname_/y
  * /input/_trackpadname_/click
  * /input/_trackpadname_/touch
* trigger - The input is a trigger (or an analog grip button.) It supports these components:
  * /input/_trigger name_/value
  * /input/_trigger name_/click - Optional
  * /input/_trigger name_/touch - Optional
* button - The input is a button. It supports these components:
  * /input/_trigger name_/click
  * /input/_trigger name_/touch - Optional
* skeleton - The input is a skeletal animation source
  * /input/_component name_/
* haptic - The input is a haptic component
  * /output/_component name_/
* pose - The input is a pose component. These are created automatically from the components in the device's rendermodel.
  * /pose/_component name_/


**binding_image_point** is the 2D position on the unscaled image that should be highlighted for this source.

**order** is an optional setting that allows the input profile to control the order of the input sources in the UI.

# Input Profile Localization

