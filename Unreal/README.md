# Maestro Unreal SDK Guide

In order to use the Maestro Glove in Unreal Engine 4, the Maestro Glove Unreal plugin must be used. The plugin can be downloaded from [the Contact CI website](https://contactci.co). [Full Unreal plugin documentation can be found here](https://docs.unrealengine.com/latest/INT/Programming/Plugins/).

## Plugin Set-up
Before the plugin can be installed, verify that the following prerequisites are met:

+ **Plugin engine version matches project engine version.**
> This can be fixed in two ways: changing the version of the project or changing the version of the plugin. 
> 
> Changing the version of the project can be done by right-clicking that project’s `.uproject` file and selecting “Switch Unreal Engine version…” followed by selecting the desired engine version.
> 
> Changing the engine version of the plugin requires it to be recompiled. This can be done a variety of ways. If the project happens to be a C++ project and the project version is greater than the plugin, the plugin will automatically be recompiled upon loading, so extra steps are not necessary. Otherwise, recompilation will have to be done manually. To recompile the plugin for a different engine version, first create a new C++ project of the desired engine version. Follow the [Plugin Installation instructions below](#plugin-installation) for installing the plugin in that project. Generate the new project’s project files by right-clicking on its `.uproject` file and selecting “Generate Visual Studio project files.” The newly generated `.sln` file with be able to recompile the plugin. By opening the solution in Visual Studio and performing a build, the plugin/project source will be recompiled into their respective binaries. Once the build succeeds, that project’s version of the Maestro Glove Unreal plugin will be compatible with the project’s engine version, and can be moved/copied to compatible projects as desired.

+ **Main plugin directory is unchanged.**
> The plugin will fail to start up if the main plugin folder “MaestroGlove” is renamed. Renaming any of the subfolders therein may also cause the plugin to fail to start, and is discouraged for this reason.

+ **Plugin contains Maestro Glove library files.**
> The Binaries folder should contain `libMaestro.dll` and `libMaestro.dll.a`, both of which are necessary at runtime for the plugin. Renaming these files or removing them will cause the plugin to fail to start.

## Plugin Installation
To install the plugin for a given project, place the Maestro Glove plugin in the `Plugins` folder in your project’s main directory. If that folder does not exist, create it first. The next time the project’s editor is opened the plugin will be loaded and ready to use. This can be verified by opening the Plugins Browser in editor by clicking on Plugins under the Edit dropdown on the main toolbar. If the plugin was loaded properly, it should be listed under Installed. Verify that the enabled checkbox has been clicked for the project here before continuing.

It is also possible to install the plugin on the specific version of UE4 itself if desired. If the plugin is placed into the `Plugins` folder for the Engine itself, usually found under `Program Files\Epic Games\UE_[VERSION]\Engine`, the plugin will startup and be available for use in any Unreal project using that version.

## Getting Started with the Plugin
The main objective of the plugin is to make it easier for developers and designers alike to interface with the Maestro Glove. For this reason, the plugin includes a Blueprint Function Library that exposes all of our C API functions to the Blueprint Editor under the Maestro Glove category. It also includes several Blueprints that should be useful when attempting to interface with the Maestro. In order to be able to see the content included with the plugin the “Show Plugin Content” checkbox under the View Options dropdown in the Content Browser must be selected. The included content is detailed in the [Useful Blueprints section below](#useful-blueprints).

If you are developing an application using C++ rather than Blueprint scripting, it may be more useful to simply include either of the C headers, `maestro.h` or `maestro_types.h`, and use those exposed functions directly. These headers are stored in the `Source\Maestro\Includes` folder of the plugin. As the included Blueprint Function Library is simply a wrapper for the C API, using the headers directly can eliminate some unnecessary overhead.

It should be noted that the plugin calls `start_maestro_detection_service` on start-up, so it is unnecessary to call this function to get the glove to connect provided the plugin is enabled and starts up properly.

## Useful Blueprints
#### MaestroHand
Example of animating a skeletal mesh with data from the Maestro. Blueprint handles haptic responses on overlap, as well as object pickup interactions. For this reason, all actors in the scene need to have overlap events enabled in order for the blueprint to react to them. All instance variables under the Maestro category are necessary for the blueprint to function properly, and their purposes are as follows:
> **Which Hand** - Whether this hand is a left hand or a right hand. Blueprint uses this to decide how to draw the skeletal mesh and which glove to retrieve data from.
> 
> **Vibration Effect** - Byte representing the vibration effect played in the fingertips during an overlap.
> 
> **Pull Amplitude** - Byte representing the amplitude of the force feedback motors during an overlap, `0` being no pull and `255` being full pull.
> 
> **Index Socket** - Name of the socket on the [MaestroHand](#maestrohand)'s skeletal mesh that it located at the tip of the index finger. This is where the index finger's collider will be attached for haptics/pickup. [Socket documentation can be found here](https://docs.unrealengine.com/latest/INT/Engine/Content/Types/SkeletalMeshes/Sockets/).
> 
> **Middle Socket** - Same as Index Socket, but for the middle finger.
> 
> **Ring Socket** - Same as Index Socket, but for the ring finger.
>
> **Little Socket** - Same as Index Socket, but for the little (pinky) finger.
> 
> **Thumb Socket** - Same as Index Socket, but for the thumb.
>
> **Palm Socket** -  Name of the socket on the Hand's skeletal mesh that it located at the center of the palm. This is where the palm's collider will be attached for pickup.
>
> **Finger Size** - Diameter of colliders used on the fingertips.
>
> **Palm Size** - Size of the collider used on the palm.

Additionally, it is generally useful to be able to calibrate based on keypresses, so three key variables are exposed in the Calibration category for this purpose.
> **Wrist Calibration Key** - Key that calibrates the wrist for this hand while held.
> 
> **Proximal Calibration Key** - Key that calibrates the proximal joints (fingers) for this hand while held.
>
> **Thumb Calibration Key** - Key that calibrates the thumb for this hand while held.

There is also a Debug category that may prove to be useful:
> **Displacements Debug Actor** - Actor reference that implements the `MaestroPassDisplacementsInterface`. Hand will pass its displacements to this actor every tick if valid. See the `DisplacementsSliderWall` blueprint for an example of how this can be used.
>
> **Show Pickup Colliders** - Boolean that says whether or not to draw the colliders on the fingertips/palm used for haptics and pickup. Colliders will appear as a blue bubble, and will turn red whenever they overlap with a valid object.

#### MaestroPawn
Pawn that tracks an HMD and two [MaestroHands](#maestrohand) for use in VR projects. By dragging this pawn in the scene and setting its device IDs to the correct values the pawn will handle your camera and hand tracking in the scene.
> **Left Tracker Device ID** - Device ID of the tracker attached to the left glove.
>
> **Right Tracker Device ID** - Device ID of the tracker attached to the right glove.
>
> **Default Player Height** - How far the HMD starts off the floor for Eye Level tracking systems (PSVR).

#### MaestroPickup
Actor component that can be attached to any overlap-enabled actor to allow the [MaestroHand](#maestrohand) to pick it up.
> **Lift Constant** - Rotation ratio above which a finger cannot grab an object.
>
> **Release Constant** - Rotation ratio above which defines a release between a finger and the palm.
>
> **Pinch Threshold** - How much your finger has to uncurl from the position it grabbed the object to define a release, specifically when the grab is between the thumb and a finger. 

#### FingertipCollider
Sphere collider that is attached to the end of a finger or to the palm to enable haptics/pickup. Spawned automatically by [MaestroHand](#maestrohand).
> **Index** - Integer index defining which finger this collider is attached to. `0` is the thumb, `1`-`4` are the fingers, `5` is the palm. Assigned automatically by [MaestroHand](#maestrohand).
>
> **Parent** - Reference to the [MaestroHand](#maestrohand) actor that owns this collider. Used to decide where to parent object upon pickup and get the glove pointer for haptic responses.
