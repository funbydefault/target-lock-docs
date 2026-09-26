# Settings reference

[Main guide](README.md) · [Integration](Integration.md) · [Upgrade notes](ReleaseNotes.md)

These settings belong to the reusable targeting system. Edit component defaults in your Blueprint. Secondary tuning controls appear under Advanced. Read each setting together with its enable switch.

## Lock component

| Setting | Effect |
| --- | --- |
| **Use External Presentation** | Keep acquisition, switching, visibility checks, and events while leaving camera, rotation, movement, and UI to your own systems. Use Set External Presentation Enabled to change this during play. |
| **Acquire Search Settings** | Rules used to find the initial target and automatic replacement targets. |
| **Line Of Sight Origin** | Where acquisition and active-lock line-of-sight traces begin. Player View Point is the default and requires a clear path from the current camera to the target. Character Eyes and Character Eyes or Player View Point remain optional alternatives. Virtual aim offsets do not move the camera visibility origin. View-cone and screen-center scoring still use the player view. |
| **Lose Target Distance** | Maximum distance in centimeters before the current target is lost. Set to 0 to disable the distance limit. |
| **Rotation Interp Speed** | Interpolation speed used when rotating the controller and character toward the target. Higher values turn faster. |
| **Focus Point World Offset** | World-space offset in centimeters added to the selected anchor location when calculating aim rotation. |
| **Rotate Controller To Target** | Rotates the owning player's control rotation toward the selected target while locked. |
| **Rotate Character To Target** | Rotates the owning Character, Pawn, or Actor's yaw toward the selected target while locked. Ignored when External Presentation is enabled. |
| **Auto Retarget When Lost** | Searches for another eligible anchor when the current target is destroyed, disabled, obstructed, or out of range. |
| **Use Locked Max Walk Speed** | Temporarily applies Locked Max Walk Speed while target lock is active, then restores the previous value. |
| **Locked Max Walk Speed** | Character movement speed in Unreal units per second while locked. |
| **Adjust Movement Rotation Settings** | Temporarily configures Character Movement for strafing while locked, then restores the previous rotation settings. |
| **Line Of Sight Check Interval** | Time in seconds between line-of-sight traces while locked. Lower values react sooner but perform more traces. |
| **Line Of Sight Loss Grace Period** | Continuous obstruction time allowed before the current target is lost. Set to 0 for immediate loss. |
| **Use Camera Position Offset** | Enables an alternate camera-relative location while locked. |
| **Target Lock Camera Relative Location** | Target relative location in centimeters for the active Camera's attached Spring Arm, or the active Camera itself when it has no Spring Arm. Falls back to the first valid Camera or Spring Arm when none is active. Also defines the virtual aim origin when camera application is disabled. |
| **Apply Camera Position Offset To Active Camera** | Applies Target Lock Camera Relative Location to the active Camera's attached Spring Arm, or to the active Camera itself when it has no Spring Arm. Falls back to the first valid Camera or Spring Arm when none is active. Disable to use it only for aim and acquisition scoring; Player View Point visibility still starts at the actual camera. |
| **Blend Camera Position Offset** | Smoothly blends to and from the target-lock camera location instead of changing it immediately. |
| **Camera Position Offset Blend In Speed** | Interpolation speed used when blending into the target-lock camera location. |
| **Camera Position Offset Blend Out Speed** | Interpolation speed used when restoring the previous camera location. |
| **Camera Aim Rotation Offset** | Rotation offset, in degrees, added after calculating the controller aim rotation. |
| **Show Focused Target Reticle** | Shows a screen-space reticle at the selected anchor while target lock is active. |
| **Focused Target Reticle Widget Class** | Widget class displayed as the focused-target reticle. The built-in class works without project assets. |
| **Focused Target Reticle World Offset** | World-space offset in centimeters added to the anchor location when positioning the focused-target reticle. |
| **Focused Target Reticle Draw Size** | Draw size of the focused-target reticle in screen pixels. |
| **Enable Automatic Toggle Input Polling** | Polls the configured legacy input keys and toggles target lock. Configure before Begin Play; Enhanced Input users can leave this disabled and call Toggle Target Lock directly. |
| **Toggle Keyboard Key** | Keyboard key checked during each automatic input poll. Changes take effect on the next poll. |
| **Toggle Gamepad Key** | Gamepad key checked during each automatic input poll. Changes take effect on the next poll. |
| **Toggle Input Poll Interval** | Time in seconds between legacy input polls. Read when polling starts at Begin Play. |
| **Direction Switch View Angle** | Full width of the camera-forward cone used for directional switching, in degrees. The default 160 allows 80 degrees to either side, including off-screen targets. Candidates must have camera line of sight; targets behind the camera never qualify. |
| **Direction Switch Min Dot** | Minimum alignment between directional input and a candidate's camera-relative direction. Higher values require more precise input. |
| **Direction Switch View Distance Weight** | Penalty applied to camera-angle separation during directional switching. Higher values favor targets nearer the current target in the camera view, even off screen. |
| **Debug Draw** | Draws the current target, view-to-target line, and cached line-of-sight result in non-Shipping builds. Disabled by default. |
| **Debug Line Thickness** | Thickness of target-lock debug lines in screen-independent Unreal debug units. |
| **Log State Transitions** | Writes activation and target-change messages to the log. Disabled by default. |
| **Sync Controller Rotation To Server** | Sends locally calculated controller rotation to the owning server at a limited rate. Enable only on replicated multiplayer Characters that need server-side control rotation. |
| **Server Control Rotation Sync Rate** | Maximum controller-rotation updates sent per second while server synchronization is enabled. |
| **On Target Lock State Changed** | Fired after target lock activates or deactivates. |
| **On Target Lock Target Changed** | Fired whenever the selected anchor changes. Either anchor can be null when selecting or clearing a target. |

## Acquisition search settings

| Setting | Effect |
| --- | --- |
| **Max Distance** | Maximum acquisition distance in centimeters. Zero prevents acquisition. |
| **Require Line Of Sight** | Require an unobstructed trace from the configured line-of-sight origin to the target anchor. |
| **Line Of Sight Trace Channel** | Collision channel used by acquisition and active-lock line-of-sight traces. |
| **Minimum View Dot** | Minimum dot product between the view direction and the target direction. One is directly ahead, zero allows targets up to 90 degrees away, and minus one allows targets behind. |
| **Use Screen Center Scoring** | Include normalized screen-center distance when a valid player controller is available. |
| **Screen Center Weight** | Contribution of screen-center distance to the score. Lower final scores win. |
| **Distance Weight** | Contribution of normalized world distance to the score. Lower final scores win. |
| **View Angle Weight** | Contribution of view-angle error to the score. Lower final scores win. |

## Target anchor

| Setting | Effect |
| --- | --- |
| **Enabled** | Whether this anchor can be acquired or retained as the current target. |
| **Selection Priority Bias** | Bias subtracted from the candidate score. Higher values make this anchor more likely to be selected; lower values make it less likely. |
| **Require Target Lockable Interface** | Require the owning actor to implement the optional Target Lockable marker interface. |
| **Target Location World Offset** | World-space offset in centimeters added to this component's world location. |
| **Additional Line Of Sight Probe Offsets** | Optional world-space offsets from the primary target location used as additional line-of-sight probes. The primary location is always tested first. Add only points that remain inside the target's visible body. |

## Reticle widget

| Setting | Effect |
| --- | --- |
| **Reticle Color** | Primary reticle color. |
| **Outline Color** | Contrast color drawn behind the reticle. Set alpha to zero to disable the outline. |
| **Line Thickness** | Thickness of the colored reticle lines in Slate units. |
| **Outline Thickness** | Thickness of the contrast outline in Slate units. |
| **Padding Ratio** | Padding between the widget edge and each reticle corner, relative to the shorter widget side. |
| **Corner Arm Ratio** | Length of each corner arm, relative to the shorter widget side. |
| **Center Diamond Ratio** | Radius of the center diamond, relative to the shorter widget side. Set to zero to hide it. |

## Practical defaults

| Setting | Default |
| --- | --- |
| Acquisition maximum distance | 2500 cm |
| Minimum view dot | -0.25 |
| Retention distance | 3000 cm; zero disables the limit |
| Switching cone | 160 degrees full width |
| Active visibility check interval | 0.05 seconds |
| Continuous obstruction grace | 0.65 seconds |
| Locked Character Movement speed | 300 cm/s |
| Rotation interpolation speed | 10; zero snaps |
| Camera-position override | Disabled |
| External presentation | Disabled |
| Automatic key polling | Disabled |
| Server rotation synchronization | Disabled |
| Debug drawing and transition logs | Disabled |

Acquisition and manual switching use different cones. The acquisition minimum view dot can admit targets behind the camera when negative; set it to zero or above for a forward-only acquisition region. Manual switching never selects behind the camera.

> [!NOTE]
> If Target Lock helps your project, consider an honest review on the **[Target Lock Fab listing](https://www.fab.com/listings/0bc4b733-f81e-44ba-9882-0189410586f9)**.
