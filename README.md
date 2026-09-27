# Target Lock

**Configurable target selection for Unreal Engine, with Blueprint and C++ integration.**

Add targeting to your existing character, pawn, or actor. Use the built-in camera and movement behavior, or let your own systems handle presentation through events and Blueprint overrides.

> [!NOTE]
> This guide describes the **1.1 release candidate**. Check the [Fab listing](https://www.fab.com/listings/0bc4b733-f81e-44ba-9882-0189410586f9) for the currently available release and supported engine versions.

[Integration guide](Integration.md) · [Settings reference](Settings.md) · [Upgrade notes](ReleaseNotes.md)

## Install

1. Install the plugin for your Unreal Engine version from your Fab library. For a source delivery, put its `TargetLock` folder inside your project's `Plugins` folder.
2. Open the project, enable **Target Lock System** in **Edit > Plugins**, and restart if requested. Source installations require the supported Unreal C++ toolchain to compile the plugin.
3. Open your player Blueprint and add **Target Lock Component**.
4. Add **Target Lock Anchor** to each target Blueprint and place it at the intended focus point, such as the chest. Attach it to a mesh socket if it should follow an animated bone.
5. Connect your input action's **Started** event to **Toggle Target Lock**. Connect directional actions to **Switch Target By Input Direction**, using `(-1, 0)` for left and `(1, 0)` for right.

The plugin does not require a demo base class, a particular skeleton, a combat framework, or an input-mapping asset. It enables Unreal's built-in **Animation Compression Library** for the included animation content. No separately purchased plugin is required.

> [!TIP]
> Already have a camera, locomotion, or UI system? Enable **Use External Presentation** before play. Target Lock keeps selecting and tracking targets while your systems respond to its events.

## How targeting works

Anchors register with a world-local subsystem during play and unregister when removed. Acquisition filters them by eligibility, distance, view angle, and optional visibility, then chooses the lowest weighted score. **Priority Bias** on an anchor lowers its score to make it more attractive.

Visibility defaults to the current camera. An acquired target must be visible immediately; an existing lock tolerates **0.65 seconds** of continuous obstruction by default. A clear visibility check resets that timer. Destruction, disabled eligibility, an unavailable view, and excess retention distance are separate loss conditions.

Directional switching uses a **160-degree full cone** around camera forward by default, plus camera-to-target line of sight. Targets just outside the viewport can qualify; targets behind the camera cannot. Switching is anchor-based, so several weak points on the same actor can be separate candidates.

**Set Target Anchor** accepts a target chosen by another system. It bypasses automatic scoring while retaining eligibility, retention-distance, and configured visibility checks. Passing an empty anchor clears the selection.

## Connect existing systems

| Task | Connection |
| --- | --- |
| Enter or leave lock-on locomotion | **On Target Lock State Changed** |
| Update combat selection or target UI | **On Target Lock Target Changed** |
| Read the selected enemy | **Get Current Target Actor** |
| Drive an external camera | **Get Current Target Focus Location**, after checking **Is Target Lock Active** |
| Use a custom camera source | Override **Get Target Lock View Point** in a component Blueprint |
| Filter teams, health, or ability states | Override **Is Target Allowed For Source** in an anchor Blueprint |
| Aim at a computed weak point | Override **Get Target Location** |

The [integration guide](Integration.md) includes the exact Blueprint steps, ownership transitions, socket setup, and C++ extension points.

## Input and animation

The API accepts input from Enhanced Input, legacy bindings, gamepad, or touch. Toggle and switch actions should fire once per press or gesture. For a stick, trigger at a threshold and re-arm after returning to the dead zone. Calling the action every frame produces repeated switches.

Switch directions use positive X for right and positive Y for down. **Switch Target By Touch Swipe** accepts an end-minus-start drag delta in screen pixels; its default minimum distance is 48 pixels. Alternatively, **Get Touch Swipe Direction** returns a normalized direction without switching.

Animation remains part of your character. Use **Is Target Lock Active** to choose your locked locomotion state and provide forward, backward, and strafe clips appropriate to its skeleton. Match animation playback to movement speed. Target Lock does not retarget arbitrary character animations.

## Camera, movement, and UI

Built-in presentation can rotate the controller and owning actor, adjust Character Movement rotation and speed, blend a camera position, and display a reticle. Each behavior has its own switch. Character Movement options apply only to an `ACharacter` with `UCharacterMovementComponent`; other actors use the same targeting API and integrate their own movement through events.

Camera position is an **absolute relative location**, not an additive delta. It applies to the active Camera's attached Spring Arm, or the Camera itself when there is no Spring Arm. Modified rigs are restored on release. Turn off the previous Camera when changing active rigs.

**Use External Presentation** leaves camera transforms, controller and actor rotation, movement settings, and UI to your project. To change this during play, call **Set External Presentation Enabled**. Enabling it restores settings previously changed by the component and removes its reticle while preserving selection. Transfer ownership before another system starts writing those same settings.

The reusable default reticle is a native amber widget. Replace **Focused Target Reticle Widget Class** with your own `UUserWidget`, or derive from `TargetLockReticleWidget` to tune its colors and geometry. A visible reticle requires an active lock, built-in presentation, and a locally controlled Pawn.

## Performance and networking

The reusable component ticks only while actively locked or blending a camera offset. Active-lock visibility checks run at the configured interval. Debug drawing and transition logging are off by default. Acquisition scans registered anchors; large target populations should be profiled in the intended game.

> [!IMPORTANT]
> Selected targets and lock state are local. They are not an authoritative replicated combat contract. Your server must validate eligibility, range, visibility, abilities, and damage.

The optional owner-to-server controller-rotation RPC is disabled by default, unreliable, and rate-limited from 1 to 60 updates per second. It requires the owning replicated Pawn and Player Controller relationship. It does not provide server-authoritative target selection or an anti-cheat boundary.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Activation returns false | Confirm an enabled anchor in the same world, a usable view, acquisition distance and cone, the optional interface requirement, project eligibility, and cover collision. |
| Lock immediately changes or drops | Check retention distance, anchor eligibility, destruction, camera availability, and the configured trace channel. The grace timer applies to obstruction only. |
| A wall does not block targeting | Make its collision block **Line Of Sight Trace Channel**, usually Visibility. Keep additional target probes inside the target's body. |
| Switching ignores an off-screen enemy | Confirm it lies within **Direction Switch View Angle**, matches the requested direction, and has camera line of sight. |
| Camera and locomotion fight each other | Enable external presentation, or disable each built-in behavior already owned by your framework. |
| Aim is too high or low | Move the anchor or set its **World Offset**. **Focus Point World Offset** changes aiming separately from anchor visibility and reticle placement. |
| Camera jumps | Check the absolute relative camera location and which Camera or Spring Arm is active. |
| Reticle is missing | Check active state, local possession, widget class, **Show Focused Target Reticle**, and external presentation. |
| Automatic polling does nothing | Enable it before Begin Play and possess the Pawn with a local Player Controller. Direct input bindings can leave polling disabled. |
| Gamepad or touch switches vertically backwards | Convert the device's axis convention to positive Y down. |

Temporarily use **Set Debug Draw Enabled** to inspect the current target and cached visibility result in a development build. Disable diagnostics for normal use.

## Support

Use [support on Discord](https://discord.com/invite/GKjSWjvEnm) or email **funbydefault.dev@gmail.com**. Include plugin version, Unreal version, platform, steps from a fresh launch, relevant settings, and the smallest useful log excerpt. Reproduce in a blank project where possible. Remove credentials, proprietary assets, and customer data before sharing a reproduction.

> [!NOTE]
> If Target Lock helps your project, consider an honest review on the **[Target Lock Fab listing](https://www.fab.com/listings/0bc4b733-f81e-44ba-9882-0189410586f9)**.
