# Maestro Unity SDK Guide

The Unity SDK can be downloaded from [the Contact CI website](https://contactci.co).

The SDK contains:
- Prefabs for the left and right arm that come fully rigged with motion capture, vibration haptics, force-feedback, gesture recognition, and pick-up logic.  
- Menus and Utilities for glove calibration, gesture recording, and general settings.
- Prefabifier utility for using custom arm or hand assets.

You can also find a minimal example project here, containing virtual reality and non-VR example scenes.

## Setup

## Functionality

Aspects of the Maestro's functionality can be used independently, or you can use the default arm prefabs which contain all functionality.

### Motion Capture

Motion capture is controlled by scripts on the appropriate segments of the hand and fingers that set the rotation for that joint. The properly script hierarchy can be seen below:

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

Vibration haptics are controlled by the `VibrateOnCollide` script. These should be placed on the finger tips. `VibrateOnCollide` will trigger vibration for the first DistalBehaviour script it finds in its parent gameobjects, so it must be below the corresponding distal script in the hierarchy.

The vibration effect can be controlled through the inspector or in code by modifying the "Vibration Effect" variable. [A list of the available vibration effects can be found here](https://contact-control-interfaces.github.io/maestro-sdk-docs/C/html/group__vibration_control.html#details). 

Controlling which collisions trigger vibration can be done by modifying the "Vibration Tag" and "Vibration Collision Mode" variables. There are three options that control how the "Vibration Tag" variable is treated:
- `VIBRATE_ON_ALL` -- Ignores "Vibration Tag" and vibrates on any collision.
- `VIBRATE_ON_TAG_MATCH` -- Vibration is only triggered if the colliding GameObject is associated with the tag specified by "Vibration Tag".
- `VIBRATE_ON_TAG_DIFFERENT` -- Vibration is only triggered if the colliding GameObject is not associated with the tag specified by "Vibration Tag". This is the inverse of `VIBRATION_ON_TAG_MATCH`.

An "Is Vibration" variable is also exposed which can be used to determine if vibration is currently active.

### Force Feedback

### Gestures

### Pick Up Logic


