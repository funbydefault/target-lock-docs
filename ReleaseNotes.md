# Target Lock 1.0.1

Release candidate documentation. The published Fab version remains the authoritative compatibility record until this update is approved.

## Changes

- Add targeting to an existing Character, Pawn, or Actor without using a demo base class.
- Let an existing camera, locomotion, and UI system own presentation through **Use External Presentation** and target-change delegates.
- Override camera viewpoint, target eligibility, and focus position in Blueprint or C++.
- Use camera-origin visibility by default, with continuous-obstruction grace and optional target probes.
- Switch within a configurable camera cone, default 160 degrees, while requiring camera line of sight.
- Restore only the movement settings changed by the component.
- Replace the example arena with a medieval courtyard and directional player locomotion, roaming targets, cover, keyboard/mouse, gamepad, and touch controls.
- Expand integration, input, scene, animation, and Blueprint tooltip regression checks.

## Upgrade

Back up the project and install the package for its exact Unreal Engine version. Rebuild native modules and compile project Blueprints. Existing reflected targeting class and function names are retained; the new external-presentation mode defaults off.

The former example-map path redirects to `/TargetLock/Example/Maps/TargetLockShowcase`. Set Editor Startup Map and Game Default Map explicitly if your project should start there. Production projects can keep their own maps and GameMode.

Camera-origin visibility is now the default. Existing serialized component settings may retain their previous origin. Review **Line Of Sight Origin** on existing components. Choose **Character Eyes** explicitly only if that is the intended behavior in your game.

For custom C++ anchor subclasses, `CanBeLockedBy()` remains available. Prefer `IsTargetAllowedForSource_Implementation()` when project rules should run after built-in eligibility checks. `GetTargetLocation_Implementation()` and `GetTargetLockViewPoint_Implementation()` are the new Blueprint-native extension points.

To roll back, restore the previous plugin and the matching project backup together. Do not save shared content in a newer Unreal version if older-engine compatibility is required.

> [!NOTE]
> If Target Lock helps your project, consider an honest review on the **[Target Lock Fab listing](https://www.fab.com/listings/0bc4b733-f81e-44ba-9882-0189410586f9)**.
