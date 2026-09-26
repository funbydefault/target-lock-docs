# Use Target Lock in an existing project

Add Target Lock to your existing Character, Pawn, or Actor. Keep your GameMode, controller, input mapping, mesh, skeleton, animation Blueprint, and combat system. The showcase classes are optional examples.

## Basic Blueprint setup

1. Enable **Target Lock System** and restart the editor if requested.
2. In your player Blueprint, add **Target Lock Component**.
3. In each target Blueprint, add **Target Lock Anchor**. Place it at the chest or another intended focus point. For a moving bone, attach the anchor to a socket on the target's skeletal mesh.
4. Bind your lock input action's **Started** event to **Toggle Target Lock**. For a UMG button, use **On Clicked**. Leave **Automatic Toggle Input Polling** disabled when your input system handles this.
5. Play. Bind left and right actions to **Switch Target By Input Direction**, using `(-1, 0)` and `(1, 0)`.

The default component supplies camera-facing rotation, optional Character Movement settings, and a reticle. Acquisition and visibility use the owning Pawn's Player Controller camera view. Without a Player Controller, it uses an active Camera Component on the owner, then the actor's eye viewpoint. A separate camera actor used through `Set View Target` is already covered by the Player Controller path.

Anchors register and unregister automatically during play. Targets need no shared base class or interface unless you explicitly enable **Require Target Lockable Interface**. Multiple anchors can be used for different weak points; switching selects anchors, so two points on the same actor may both be candidates.

## Keep your existing camera, movement, and UI

Enable **Target Lock → Integration → Use External Presentation** on the component before play.

Target Lock still handles acquisition, switching, eligibility, visibility, loss grace, and events. Your systems own camera position, controller rotation, actor rotation, movement settings, and UI. This is the starting point for a custom locomotion framework, ability-driven movement, Mover, a vehicle, or an existing combat camera; those specific frameworks still need your project's event wiring.

| Connection | Use |
| --- | --- |
| **On Target Lock State Changed** | Enter or leave your lock-on movement/camera state. |
| **On Target Lock Target Changed** | Notify combat, update target UI, or play a switch cue. `New Target` can be empty when selection clears. |
| **Is Target Lock Active** | Check before applying ongoing camera or animation behavior. |
| **Get Current Target Actor** | Resolve the selected enemy for your combat system; returns empty without a selection. |
| **Get Current Target Focus Location** | Read the current world focus position for your camera; returns zero without a selection. |
| **Get Current Target Anchor** | Read the exact weak point when an actor has multiple anchors. |

Use **Set External Presentation Enabled** to change ownership during play. Enabling it restores settings previously changed by the component and removes its reticle, while retaining the selected target. Disabling it reapplies the configured built-in presentation. Switch to external presentation before letting another system take over the same movement or camera properties.

For partial customization, leave external presentation disabled and turn off just the built-in behaviors you replace: **Rotate Controller To Target**, **Rotate Character To Target**, **Adjust Movement Rotation Settings**, **Use Locked Max Walk Speed**, **Use Camera Position Offset**, or **Show Focused Target Reticle**. The component restores only the movement setting groups it actually changed.

### Supply a custom camera view

If your view does not come from the owning Pawn's Player Controller or an owner Camera Component:

1. Create a Blueprint derived from **Target Lock Component** and add that component to the owner in place of the base component.
2. Override **Get Target Lock View Point**.
3. Return your camera's world position and rotation, and set the return value to true. Return false when that view is unavailable; acquisition fails and an existing lock is released.

The override drives selection, camera-origin visibility, and directional switching. Queries can run several times per update: read state without changing it. If this view differs from the Player Controller's camera, screen-center scoring is skipped automatically; distance and view-angle scoring still apply.

## Connect health, teams, or abilities

Create a Blueprint derived from **Target Lock Anchor Component** and use it on your enemies. Override **Is Target Allowed For Source** to read the rules your game already owns, for example:

```text
Source Actor → resolve player's team
Get Owner → resolve target's team and health
Return: target is alive AND target is hostile to Source Actor
```

Use your existing interface, component, tags, or ability system to answer those queries. The plugin does not require a particular health or team implementation. Keep the query free of side effects. A false result excludes that anchor from acquisition and switching, and releases an existing lock on the next target update. Eligibility loss is immediate; the visibility grace period applies to obstruction only.

For a simple death/disable path, call **Set Anchor Enabled(false)**. The built-in enabled, self-target, and optional marker-interface checks still apply before the Blueprint rule.

## Choose the focus point

Usually, moving the anchor or attaching it to a mesh socket is enough. **World Offset** adds a world-space offset in centimeters. For a computed weak point, override **Get Target Location** in your anchor Blueprint and return a finite world-space position.

That point controls selection, visibility probes, and the reticle. **Focus Point World Offset** on the lock component adds a final aiming offset for built-in rotation and **Get Current Target Focus Location**; it does not move the anchor or visibility probes.

## Tune selection and visibility

| Setting | Effect |
| --- | --- |
| **Acquire Search Settings / Max Distance** | Maximum acquisition distance from the search viewpoint, in centimeters. Activation also respects the owner's retention distance. |
| **Minimum View Dot** | Acquisition cone: `1` is directly forward, `0` reaches 90° to either side, negative values allow acquisition behind the view. |
| **Direction Switch View Angle** | Full camera-forward switching cone, default 160°. An off-screen target can qualify inside the cone; targets behind the camera cannot. |
| **Line Of Sight Trace Channel** | Channel your walls and cover should block. |
| **Line Of Sight Origin** | Defaults to **Player View Point**, including a custom view override. Manual switches always require camera-to-target visibility. |
| **Line Of Sight Loss Grace Period** | How long an existing target must remain continuously obstructed before it is lost. Default 0.65 seconds. Acquisition and switching still require visibility immediately. |
| **Lose Target Distance** | Separate retention distance. Zero disables distance-based loss. |
| **Auto Retarget When Lost** | Select another eligible target when the current one becomes invalid. |
| **Additional Line Of Sight Probe Offsets** | Extra points for a large target. Keep them inside the target's body to avoid visibility around unrelated cover. |

The lowest weighted score wins; **Priority Bias** favors an anchor by subtracting from its score. Target Lock does not require an enemy to face the player.

If another targeting system already chooses an enemy, pass its anchor to **Set Target Anchor** with **Activate If Needed** enabled. This explicit selection bypasses automatic candidate scoring while preserving eligibility, retention-distance, and configured visibility validation. Passing an empty anchor clears the selection.

## Input and animation

The same API accepts Enhanced Input, legacy input, gamepad, or touch. For a gamepad stick, trigger one directional switch as the stick crosses a threshold, then re-arm after it returns to the dead zone. Do not call Toggle or Switch every frame from a continuously triggered input action. Positive X means right; positive Y means down, so invert a device axis that reports up as positive.

Touch can call **Switch Target By Touch Swipe** with a screen-pixel drag delta. It returns false below the supplied threshold. Trigger once per gesture, then re-arm on release.

Use **Is Target Lock Active** to choose your animation Blueprint's locked locomotion state. Supply forward, backward, and strafe animations for your own skeleton, and match playback to movement speed. Target selection has no skeleton dependency; it does not retarget animations or build an arbitrary character's locomotion for you.

## C++ integration

Add `TargetLock` to your module's dependencies. Use a public dependency if Target Lock types appear in public headers; otherwise use a private dependency.

```cpp
#include "TargetLock/TargetLockComponent.h"
#include "TargetLock/TargetLockAnchorComponent.h"

// In your existing Actor/Pawn/Character constructor:
TargetLock = CreateDefaultSubobject<UTargetLockComponent>(TEXT("TargetLock"));
TargetLock->bUseExternalPresentation = true;

// In your existing target's constructor, after creating its root:
TargetAnchor = CreateDefaultSubobject<UTargetLockAnchorComponent>(TEXT("TargetAnchor"));
TargetAnchor->SetupAttachment(GetRootComponent());
```

Store these pointers in reflected component properties. Bind input to `ToggleTargetLock()` and `SwitchTargetByInputDirection()`. Bind to `OnTargetLockStateChanged` and `OnTargetLockTargetChanged` for project behavior.

Component subclasses can override these methods:

```cpp
bool GetTargetLockViewPoint_Implementation(FVector& ViewLocation, FRotator& ViewRotation) const override;
bool IsTargetAllowedForSource_Implementation(const AActor* SourceActor) const override;
FVector GetTargetLocation_Implementation() const override;
```

The first belongs to the lock component; the other two belong to the anchor. Existing C++ `CanBeLockedBy()` overrides remain available. Prefer the new eligibility hook when you want to preserve the built-in guards automatically.

## Boundaries and upgrading

Built-in speed and movement-rotation settings apply to `ACharacter` with `UCharacterMovementComponent`. Generic Pawns and Actors use the same targeting logic; integrate their movement through events or external presentation. No showcase Character, GameMode, controller, HUD, animation, sound, or map is required for reusable targeting.

Selection and lock state are local. The optional controller-rotation RPC does not replicate an authoritative combat target. Your server must validate target eligibility, range, line of sight, and damage using your existing multiplayer architecture.

Existing reflected class, component, property, and asset names remain unchanged. External presentation defaults off, so existing Character setups retain their configured presentation. Existing Blueprint calls to **Get Target Location** retain the same function name; subclasses can now override it. Rebuild C++ modules and compile project Blueprints after updating. Back up production projects before replacing plugin source or assets.

The independent integration checks cover UE 5.4 and UE 5.8 locally. Individual third-party frameworks and the complete release engine/platform matrix need their own validation.

If Target Lock helps your project, consider an honest review on the **[Target Lock Fab listing](https://www.fab.com/listings/0bc4b733-f81e-44ba-9882-0189410586f9)**.
