# Maestro Unity SDK Guide

The Unity SDK can be downloaded [here](https://github.com/Contact-Control-Interfaces/maestro-sdk-unity/releases/tag/v0.1a).

The SDK contains:
- Prefab for the left and right arm that comes fully rigged with motion capture, vibration haptics, force-feedback, gesture recognition, and pick-up logic.  
- Menus and Utilities for glove calibration, gesture recording, and general settings.
- ~~Prefabifier utility for using custom arm or hand assets.~~ *(In development; will be released soon)*

You can also find a minimal example project [here](https://github.com/Contact-Control-Interfaces/maestro-sdk-unity/releases/download/v0.1a/unitysdkdarts.zip), containing some interaction and menu examples.

## Setup

First you will need to run the [Maestro Alpha installer](https://github.com/Contact-Control-Interfaces/maestro-installer), which will handle creating configuration files needed for the Maestro to work. This is described under "Initial Setup" and "Configuration" [here](https://contact-control-interfaces.github.io/maestro-sdk-docs/C/html/index.html)

Once you have configuration files installed and the Unity Maestro SDK Unity Package imported into a Unity project, you should see the "DefaultMaestroRig [CameraRig]" prefab under `Assets/Prefabs`. This is the fully rigged headset and arms described above. If you want more control over which features are enabled, they are described below.

## Functionality

Aspects of the Maestro's functionality can be used independently, or you can use the default arm prefabs which contain all functionality.

### Motion Capture

Motion capture is controlled by scripts on the appropriate segments of the hand and fingers that set the rotation for that joint. The proper script hierarchy can be seen below:

- Wrist
	- IndexProximalBehaviour
		- IndexDistalBehaviour
			- FingerTipBehaviour
	- MiddleProximalBehaviour
		- MiddleDistalBehaviour
			- FingerTipBehaviour
	- RingProximalBehaviour
		- RingDistalBehaviour
			- FingerTipBehaviour
	- LittleProximalBehaviour
		- LittleDistalBehaviour
			- FingerTipBehaviour
	- ThumbProximalBehaviour
		- ThumbDistalBehaviour
		
### Vibration Haptics

Vibration haptics are controlled by the `VibrateOnCollideBehaviour` script. These should be placed on the finger tips. `VibrateOnCollideBehaviour` will trigger vibration for the first DistalBehaviour script it finds in its parent gameobjects, so it must be below the corresponding distal script in the hierarchy.

`VibrateOnCollideBehaviour` works for both trigger and non-trigger collisions.

The vibration effect can be controlled through the inspector or in code by modifying the "Vibration Effect" variable. [A list of the available vibration effects can be found here](https://contact-control-interfaces.github.io/maestro-sdk-docs/C/html/group__vibration_control.html#details). 

Controlling which collisions trigger vibration can be done by modifying the "Vibration Tag" and "Vibration Collision Mode" variables. There are three options that control how the "Vibration Tag" variable is treated:
- `VIBRATE_ON_ALL` -- Ignores "Vibration Tag" and vibrates on any collision.
- `VIBRATE_ON_TAG_MATCH` -- Vibration is only triggered if the colliding GameObject is associated with the tag specified by "Vibration Tag".
- `VIBRATE_ON_TAG_DIFFERENT` -- Vibration is only triggered if the colliding GameObject is not associated with the tag specified by "Vibration Tag". This is the inverse of `VIBRATION_ON_TAG_MATCH`.

An "Is Vibration" variable is also exposed which can be used to determine if vibration is currently active.

### Force Feedback

Force feedback is controlled by the `PullOnCollideBehaviour` script. These should be placed on the finger tips. `PullOnCollideBehaviour` will trigger vibration for the first DistalBehaviour script it finds in its parent gameobjects, so it must be below the corresponding distal script in the hierarchy.

`PullOnCollideBehaviour` works for both trigger and non-trigger collisions.

The force feedback system will provide minimal resistance to maintain taughtness of internal tendons while not in collision. This force is controlled by the "Default" variable, and defaults to 40.

The force feedback amplitude (how hard the feedback pulls) during collisions can be controlled through modifying the "Amplitude" variable. Values over 200 are clamped.

A "Pulling" variable is also exposed which can be used to determine if collision force feedback is currently active.

### Gestures

The GestureRecognition script is our in-house solution for creating and identifying gestures. We use the term 'gesture' to refer to a specific finger configuration, like a 'thumbs-up' or pointing with your index finger.

Please note: it is advised that you go through the Maestro Setup documentation before continuing to ensure a better understanding of the setup process and components.

#### Setting up the MaestroArm script

The MaestroArm is a largely self-driven component and requires little setup effort. One factor is assigning the ArmSide enum to either left or right, as necessary. The other important step is in assigning the fingers of your hand hierarchy. This is pre-assigned in the DefaultMaestroRig prefab, but the references have been left open in the case of a custom hand hierarchy.

Please make sure to add your finger-tip objects to the MaestroArm 'PlayerFingers' array in the following order:

0 - Thumb
1 - Index
2 - Middle
3 - Ring
4 - Little

This is crucial as the GestureRecognition system will reference these transforms in its recognition process.

#### Describing the recognition process

Recognition is achieved by comparing the angle between each of the players fingers and those of a previously saved moment. How much margin of error for each finger is decided by an 'angle leniency' number for each finger, stored in each gesture file.

For this reason, it is important that your finger tips transforms 'forward' direction is coming out the end of the finger, and 'up' is through the fingernail. If this is not the case, recognition accuracy may suffer.

Recognition accuracy may also suffer if the orientation of your fingertip transforms changes after a gesture has been saved. If this has occurred, simply delete the gesture asset and capture it again.

It is also advised to populate the PlayerPalm reference, although it is not directly used in the gesture recognition process.

#### Setting up the GestureRecognition script

Adding the GestureRegonition component is very easy. Simply select the root transform of your arm object ("maestro_left" and "maestro_right" in the DefaultMaestroRig) and add the GestureRecognition component. If there is no MaestroArm component already present on the arm, one will also be added. Setup for that is outlined above.

#### Creating a new gesture

Creating a new gesture is a relatively simple process. Pressing the 'Create Gesture' button on the GestureRecognition script will save the current arms finger configuration as a .asset file, with the file being named with the contents of the 'AssetName' text field or 'New[Left/Right]Gesture[number]' if AssetName is empty.

The file is saved directly to the Assets folder of your current project, or alternatively it can be directed to a subdirectory by populating the 'CustomSubdirectory' text field. For example, if the left MaestroArms customSubdirectory field is set to 'MaestroUnitySDK/Gestures/' and the AssetName field is empty, a new gesture will end up at ~/Assets/MaestroUnitySDK/Gestures/NewLeftGesture0.asset. Saving once more will create ~/Assets/MaestroUnitySDK/Gestures/NewLeftGesture1.asset.

If a gesture is created with the left hand, it can be safely loaded and recognised by the right hand and vice versa. This is handled automatically internally.

#### Loading a stored gesture

Gestures are saved as a '.asset' file in the Assets folder or a subfolder if described with the CustomSubdirectory text field. The benefit of storing gestures as .asset files is that the details of each gesture can be tweaked later on without needing to recapture the gesture. This can be done by finding a gestures .asset file and adjusting the contents in the Inspector window.

However, the .asset file itself is not directly used for the recognition process. Instead, when loaded, a new object is created with the GestureObject component and an invisible hand is created out of empty transforms to represent the gesture. These gesture objects are created as a child of an object called the GestureIndex, local to each respective arm, to ensure the hierarchy doesn't clutter. It is advised that loading a gesture is done outside of playmode using the 'Load Gesture' button.

#### Clearing the gesture list

Besides the gestureIndex, a list of loaded gestures is exposed in the Inspector. This list should automatically match that of the GestureIndex's children and thus ideally never be directly edited. In the case that this has happened or something else is causing problems, you can press the 'Clear Gesture List' button to clear the currently loaded gestures and start again.

#### Triggering behaviour

The script has two pre-defined UnityEvents for triggering behaviour based on which gesture has been recognised, OnGestureRecognised and OnGestureUnrecognised. When creating a function that listens to these events, a GestureObject parameter is required as one will be sent when either a gesture has been recognised or ceased. An example of this behaviour can be found in the GestureExample script, included as part of the demonstration scene.

#### Contents of a gesture file

GestureName - A string for the name of this particular gesture. This is useful for identifying which gesture has been recognised in the built-in UnityEvents.
SkipRecognition - If it is desired that a gesture be loaded for recognition but be activated or deactivated at a later point, this bool can be turned true to temporarily skip the recognition process for it.
Positions - The positions of each gestures fingertip
Rotations - The rotations of each gestures fingertip
Angle Leniencies - The angular margin-of-error (in degrees) for each finger to be recognised. The higher this value, the more likely this finger is to 'pass' the recognition test. Anything below 5 degrees could be considered very strict recognition.

### Pick Up Logic

The pickup logic is controlled by four scripts: `MaestroHandControl`, `MaestroPhysicalInteraction`, `FingerTipCollider`, and `MaestroInteractable`. In order to have the hand able to pick up an object:
- The `MaestroHandControl` and `MaestroPhysicalInteraction` scripts must be properly bound to the hand hierarchy.
- The `MaestroInteractable` script must be placed onto the desired object.

The `FingerTipCollider` script is necessary for pickup to occur, however they are spawned by the `MaestroPhysicalInteraction` script and as such require no setup. The required setup for the other three necessary scripts are as follows:

##### `MaestroHandControl`
Can be placed anywhere. Usually placed on the same object as the `MaestroGloveBehaviour` or `MaestroArm` scripts.

The "Which Hand" variable says whether the hand is a right hand or left hand.

The "Controller" variable should be set to be the `SteamVR_TrackedObject` that is controlling the hand's movement.

The various "Curl"/"Lift" fields will show the rotation ratios for each finger. Included only for debug purposes. Shows `-1` if the glove is not connected.

The "Hand Open"/"Hand Closed" booleans are shown for debug purposes. "Hand Open" will be true when the hand is mostly open, and "Hand Closed" will be true when the hand is mostly closed (i.e a fist).

##### `MaestroPhysicalInteraction`
Must be placed on the same object as `MaestroHandControl`.

The "Tip_thumb" variable should be set to a transform providing the location of the tip of the thumb.

The "Tip_index" variable should be set to a transform providing the location of the tip of the index finger.

The "Tip_middle" variable should be set to a transform providing the location of the tip of the middle finger.

The "Tip_ring" variable should be set to a transform providing the location of the tip of the ring finger.

The "Tip_pinky" variable should be set to a transform providing the location of the tip of the little (pinky) finger.

The "Palm" variable should be set to a transform providing the location of the center of the palm.

The "Lift Constant" variable defines what rotation ratio above which an object cannot be grabbed with that finger.

The "Release Constant" variable defines what rotation ratio above which an object is released while held between the palm and a finger.

The "Debug" variable should be true if the `FingerTipColliders` used for pickup should be rendered.

The "Other Hand" variable should be set to the `MaestroPhysicalInteraction` script that is on the other hand.

##### `MaestroInteractable`
Must be placed on any game object that would like to be able to be picked-up.

The "Kind" dropdown defines how the object reacts when picked up. The options are as follows:
- Non Grabbable - Object is unable to be picked up.
- Grabbable - Object is able to be picked up. Object will follow position/rotation of hand while grabbed.
- Snapping Grabbable Translate - Object will snap to snap position when grabbed, and will only translate with hand position.
- Snapping Grabbable Rotate - Object will snap to snap position when grabbed, and will only rotate with hand.
- Tool - Object snaps to snapping origin upon being picked up. Release occurs when the hand is deemed to be fully open.

The "Snapping Origin" is the transform that the object snaps to when in one of the snapping or tool modes. Automatically set to this objects transform if left null.

The "On Touched" event is fired whenever the object is touched at all.

The "On Poked" event is fired whenever the object is touched by the index finger specifically.

The "On Grabbed" event is fired whenever the object is successfully grabbed.

The "On Dropped" event is fired whenever the object is successfully released.

##### `FingerTipCollider`
Must be placed on each fingertip and in the center of the palm. Spawned and set up automatically by `MaestroPhysicalInteraction`.

<strong>Please contact us at support@contactci.co with any questions.</strong>
