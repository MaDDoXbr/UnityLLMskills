# unity-spine

Guidance for writing Unity C# code that uses the Spine Unity runtime plugin, focused on the basic `SkeletonAnimation` workflow and the most common `AnimationState` operations.

For additional or version-specific details, consult the official Spine Unity main components documentation:
https://esotericsoftware.com/spine-unity-main-components
https://esotericsoftware.com/spine-api-reference

## Scope

Use this skill when the user asks for Unity C# code, debugging help, or implementation guidance involving Spine characters using the Spine Unity plugin.

Prioritize the standard `SkeletonAnimation` workflow unless the user explicitly says they are using `SkeletonGraphic` for UI or `SkeletonMecanim` for Unity Mecanim integration.

## Core concepts

- `SkeletonAnimation` is the usual runtime component for a Spine skeleton in a Unity scene.
- It exposes the skeleton instance through `SkeletonAnimation.Skeleton`.
- It exposes animation playback through `SkeletonAnimation.AnimationState`.
- `AnimationState` controls tracks, current animations, queued animations, mixing, and animation events.
- Track `0` is normally the base/full-body animation track. Higher tracks are commonly used for layered or override animations.

## Critical lifecycle rule

`SkeletonAnimation` component properties must only be accessed or cached in `Start` or later Unity event functions.

Do not cache `SkeletonAnimation`, `Skeleton`, or `AnimationState` in `Awake` for generated code unless the user explicitly knows the initialization order and requests it. In generated examples, use `Start`, `OnEnable` after initialization, or later events.

Preferred pattern:

```csharp
using Spine;
using Spine.Unity;
using UnityEngine;

public sealed class SpineCharacterAnimator : MonoBehaviour {
    private SkeletonAnimation skeletonAnimation;
    private AnimationState animationState;
    private Skeleton skeleton;

    private void Start() {
        skeletonAnimation = GetComponent<SkeletonAnimation>();
        animationState = skeletonAnimation.AnimationState;
        skeleton = skeletonAnimation.Skeleton;
    }
}
```

## Setting animations

Use `AnimationState.SetAnimation(trackIndex, animationName, loop)` to set the animation on a track.

Meaning:

- `trackIndex`: which animation track to control, usually `0` for the main animation.
- `animationName`: the Spine animation name, preferably selected with `[SpineAnimation]` when exposed in the Inspector.
- `loop`: whether the animation repeats.
- Return value: a `TrackEntry`, useful for setting options or listening for animation-specific events.

`SetAnimation` defines the animation for a track, sets whether it is loopable, and starts playing it from the beginning.

Correct usage:

```csharp
using Spine;
using Spine.Unity;
using UnityEngine;

public sealed class PlayerSpineAnimation : MonoBehaviour {
    [SpineAnimation] public string idleAnimation;
    [SpineAnimation] public string runAnimation;

    private SkeletonAnimation skeletonAnimation;
    private AnimationState animationState;
    private string currentAnimation;

    private void Start() {
        skeletonAnimation = GetComponent<SkeletonAnimation>();
        animationState = skeletonAnimation.AnimationState;

        PlayAnimation(idleAnimation, true);
    }

    public void PlayAnimation(string animationName, bool loop) {
        if (currentAnimation == animationName)
            return;

        currentAnimation = animationName;
        animationState.SetAnimation(0, animationName, loop);
    }
}
```

## `SetAnimation` anti-pattern

Do not call `SetAnimation` continuously from `Update` without checking whether the animation actually changed.

Bad pattern:

```csharp
private void Update() {
    animationState.SetAnimation(0, runAnimation, true);
}
```

Why this is wrong:

- Every call restarts the animation from frame/time zero.
- If called every frame, the character can appear stuck on the first frame.
- It creates repeated `TrackEntry` instances and unnecessary mixing work.

Correct pattern:

```csharp
private void Update() {
    bool shouldRun = Mathf.Abs(input.x) > 0.01f;
    string targetAnimation = shouldRun ? runAnimation : idleAnimation;

    PlayAnimation(targetAnimation, true);
}
```

## Queueing animations

Use `AddAnimation(trackIndex, animationName, loop, delay)` when an animation should play after the current animation on the same track.

```csharp
animationState.SetAnimation(0, attackAnimation, false);
animationState.AddAnimation(0, idleAnimation, true, 0f);
```

Use this for one-shot animations such as attack, hit, emote, or interact, followed by an idle or locomotion fallback.

## Clearing tracks

Use these when a track should stop applying animations:

```csharp
animationState.ClearTrack(trackIndex);
animationState.ClearTracks();
```

Use `ClearTrack` for a specific overlay track. Use `ClearTracks` only when intentionally clearing the whole animation state.

## Working with skins

After changing skins or attachments, reset slots if needed and apply the result before expecting the visual state to be correct.

Common pattern:

```csharp
skeleton.SetSkin("skin-name");
skeleton.SetSlotsToSetupPose();
skeletonAnimation.Update(0f);
```

Use generated or validated skin names when possible. Avoid hard-coded names unless the user provided the Spine setup.

## Events

Use `TrackEntry` callbacks for animation-specific events:

```csharp
TrackEntry entry = animationState.SetAnimation(0, attackAnimation, false);
entry.Complete += OnAttackComplete;
```

Unsubscribe when reusing persistent callbacks or when object lifetime matters. Do not keep using a `TrackEntry` after it has been disposed.

For user-defined Spine events, subscribe to `AnimationState.Event` or to a specific `TrackEntry.Event` when the event only matters for one animation.

## WaitForSpineEvent. 

Waits until a Spine.AnimationState raises a user-defined Spine.Event (named in Spine editor).

```csharp
yield return new WaitForSpineEvent(skeletonAnimation.state, "spawn bullet");
// You can also pass a Spine.Event's Spine.EventData reference.
Spine.EventData spawnBulletEvent; // cached in e.g. Start()
..
yield return new WaitForSpineEvent(skeletonAnimation.state, spawnBulletEvent);
```

## Inspector attributes

When exposing Spine names in Unity inspectors, prefer Spine Unity attributes to reduce string mistakes:

```csharp
[SpineAnimation] public string idleAnimation;
[SpineAnimation] public string runAnimation;
[SpineSkin] public string defaultSkin;
[SpineSlot] public string weaponSlot;
[SpineAttachment] public string swordAttachment;
```

## Recommended response style for code help

When generating code:

- Include `using Spine;` and `using Spine.Unity;` when relevant.
- Cache `SkeletonAnimation`, `AnimationState`, and `Skeleton` in `Start` or later.
- Guard `SetAnimation` calls so they only run on actual animation changes.
- Use `[SpineAnimation]` for animation-name fields exposed in the Inspector.
- Prefer small, complete `MonoBehaviour` examples.
- Mention that exact animation, skin, slot, and attachment names must match the exported Spine project.

## Common diagnostic checks

When troubleshooting:

- Confirm the GameObject has a `SkeletonAnimation` component.
- Confirm the `SkeletonDataAsset` is assigned.
- Confirm animation names match the Spine export exactly.
- Confirm code is not repeatedly calling `SetAnimation` every frame.
- Confirm the script is reading or modifying Spine state after `SkeletonAnimation` has initialized.
- Confirm whether the user is using `SkeletonAnimation`, `SkeletonGraphic`, or `SkeletonMecanim`; APIs and rendering context differ.
