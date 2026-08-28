# RetroTouch

> A lightweight, configurable and reusable touch-control overlay for Android games.

[![Android API](https://img.shields.io/badge/Android-API%2016--36-3DDC84?logo=android&logoColor=white)](https://developer.android.com/)
[![Status](https://img.shields.io/badge/Status-1.0.0--beta.4-orange)](#project-status)
[![Language](https://img.shields.io/badge/Java-8-ED8B00?logo=openjdk&logoColor=white)](https://www.java.com/)
![AI](https://img.shields.io/badge/AI-assisted%20coding-6e7781)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Support via PayPal](https://img.shields.io/badge/Support%20via-PayPal-0070BA?logo=paypal&logoColor=white)](https://paypal.me/andiweli)


RetroTouch is an engine-agnostic Android touch-controls library for games and source ports. It adds a resolution-independent virtual joystick, configurable action buttons, relative FPS look controls and optional menu navigation above an existing Android game view.

The overlay is delivered as a replaceable Android Archive (`.aar`) and is suitable for Java, JNI, SDL, OpenGL, `SurfaceView`, `GLSurfaceView` and native game engines. The game keeps ownership of its input system: RetroTouch emits stable action IDs and normalized movement/look callbacks, while a small game-specific bridge maps them to keys, axes or engine events.

RetroTouch was created for retro FPS ports that need modern Android multitouch controls without hard-coding a different overlay into every game.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ca590585-8451-4f00-bddb-244ef07e0c3b" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d1d895b4-6f10-4eb6-b3f6-f6b210013eb1" />


## Key features

- Android 4.1 / API 16 through Android 16 / API 36
- no AndroidX requirement and no external runtime dependencies
- true pointer-ID-based multitouch
- up to 16 configurable action buttons
- up to two movement sticks, two relative-look zones and two digital D-pads
- normalized layouts for 4:3, 16:9, 20:9, tablets and ultrawide Automotive displays
- independent `OFF`, `GAMEPLAY` and optional `NAVIGATION` modes
- persistent per-game gameplay and navigation layouts
- built-in layout editor for moving, resizing, remapping and changing opacity
- movable, scalable and opacity-adjustable Settings button
- vertically draggable editor toolbar
- automatic one- or two-line button labels with dynamic font fitting
- FPS-friendly simultaneous movement, turning and held FIRE
- host-controlled latched-button display, suitable for always-run modes
- public input-reset API for respawn, level changes and engine input resets
- controller detection for built-in, USB, Bluetooth, Xbox, PlayStation and generic gamepads
- compatible AAR updates without copying RetroTouch source into each game
- MIT license for use in open-source and commercial projects

## Typical use cases

RetroTouch is designed for:

- Android game ports and source ports
- retro FPS and action games
- SDL and JNI-based games
- OpenGL and custom-rendered Android games
- Unreal Engine 1 ports, including Unreal and Unreal Tournament 99
- Android handhelds, phones, tablets, ChromeOS and Android Automotive
- projects that need an editable virtual joystick and gamepad-style touch overlay

## Project status

The current release is **RetroTouch 1.0.0-beta.4**.

The complete core feature set has been tested with the Android port of **Breathless**, including multitouch movement, relative look, held FIRE while turning, always-run indication, D-pad menu navigation, controller detection and editable layouts. The beta phase continues while RetroTouch is tested across additional engines and input stacks.

## How RetroTouch is structured

RetroTouch intentionally separates reusable overlay code from game-specific input logic:

1. **`retrotouch.aar`** — reusable RetroTouch library
2. **game bridge** — maps RetroTouch callbacks to the game's input system
3. **game layout** — defines available actions, labels and default positions

With this structure, a compatible RetroTouch update normally requires only replacing `app/libs/retrotouch.aar` and rebuilding the APK or AAB. The game bridge and native engine code remain unchanged.

The AAR is compiled into the final application. It cannot be exchanged inside an already installed APK without rebuilding and reinstalling the game.

## Installation with the AAR

Copy the current AAR into the application module:

```text
app/libs/retrotouch.aar
```

Add it to the app module's `build.gradle`:

```groovy
dependencies {
    implementation files('libs/retrotouch.aar')
}
```

Do not also copy RetroTouch Java sources into the app module. Keeping the library behind the AAR boundary makes future updates predictable.

## Minimal integration

Place RetroTouch above the game's existing view:

```java
FrameLayout.LayoutParams matchParent = new FrameLayout.LayoutParams(
        FrameLayout.LayoutParams.MATCH_PARENT,
        FrameLayout.LayoutParams.MATCH_PARENT);

FrameLayout root = new FrameLayout(this);
root.addView(gameView, matchParent);

RetroTouchView retroTouch = new RetroTouchView(this);
root.addView(retroTouch, matchParent);

setContentView(root);
```

Register the actions supported by the game:

```java
retroTouch.registerAction("fire", "Fire");
retroTouch.registerAction("use", "Use");
retroTouch.registerAction("run", "Run");
retroTouch.registerAction("weapon_next", "Next Weapon");
retroTouch.registerAction("menu", "Menu");
```

Define a normalized default gameplay layout:

```java
List<RetroTouchControl> gameplay = new ArrayList<RetroTouchControl>();

gameplay.add(RetroTouchControl.lookZone(
        "look", 0.72f, 0.48f, 0.56f, 0.78f));

gameplay.add(RetroTouchControl.moveStick(
        "move", 0.16f, 0.76f, 0.25f));

gameplay.add(RetroTouchControl.button(
        "fire_button", "fire", "Fire", 0.89f, 0.71f, 0.15f));

retroTouch.setGameplayLayout(
        new RetroTouchLayout("my_game", gameplay));
```

Map callbacks to the input path already used by the game:

```java
retroTouch.setListener(new RetroTouchAdapter() {
    @Override
    public void onAction(String actionId, boolean pressed) {
        nativeTouchAction(actionId, pressed);
    }

    @Override
    public void onMove(float x, float y) {
        nativeTouchMove(x, y);
    }

    @Override
    public void onLook(float deltaX, float deltaY) {
        nativeTouchLook(deltaX, deltaY);
    }
});
```

RetroTouch deliberately does not decide what `fire`, `use` or `menu` means. The host game maps each action ID to its own keyboard, mouse, controller, SDL, JNI or native engine input.

## Overlay modes

RetroTouch provides three universal modes:

```java
retroTouch.setMode(RetroTouchMode.OFF);
retroTouch.setMode(RetroTouchMode.GAMEPLAY);
retroTouch.setMode(RetroTouchMode.NAVIGATION);
```

- **`OFF`** — draws nothing and lets the game's original touch input handle menus, intros or skip screens
- **`GAMEPLAY`** — shows movement, look and action controls
- **`NAVIGATION`** — optionally shows a digital D-pad plus freely mapped OK/Back-style buttons

A game is not required to use every mode. For example, a port with native touch menus can keep RetroTouch `OFF` outside gameplay.

## FPS FIRE and LOOK

A FIRE button whose center is placed inside a look zone can remain pressed while the same finger continues turning:

```java
retroTouch.setLookWhileHoldingAction("fire", true);
```

The action remains held until the finger is lifted. A second finger can simultaneously operate the movement stick.

## Always-run indication

Always-run or double-tap logic belongs to the host game because every engine handles running differently. RetroTouch can visually mark the state:

```java
retroTouch.setActionLatched("run", alwaysRunEnabled);
```

A latched action is drawn two opacity levels stronger. The game remains responsible for detecting the double tap and applying the run multiplier while movement is active.

## Layout editor

The Settings button opens the in-game editor. Players can:

- add action buttons
- change a button's registered action
- move and resize controls
- cycle through opacity levels
- delete ordinary controls
- reset the current layout
- save the current gameplay or navigation layout

The Settings button itself can be selected, moved, resized and adjusted with `ALPHA`, but cannot be deleted. The editor toolbar can be dragged vertically when it covers another control. All settings are stored privately per game/layout ID.

## Controller coexistence

Games that should show RetroTouch only when no physical controller is available can use:

```java
boolean controllerConnected =
        RetroTouchControllers.isControllerConnected();

retroTouch.setMode(controllerConnected
        ? RetroTouchMode.OFF
        : wantedMode);
```

Detection uses Android's `GAMEPAD` and `JOYSTICK` input sources instead of vendor lists. This covers built-in handheld controls as well as Xbox, PlayStation, USB, Bluetooth and generic Android controllers.

Repeat the check during the game's normal state update to support hot-plugging without restarting.

## Resetting input safely

Engines may discard their internal key or axis state during respawn, checkpoint restart, level loading, focus loss or pause. Reset RetroTouch at the same point:

```java
engineResetInput();
retroTouch.resetInputState();
```

The call releases held actions, sends neutral movement, removes active touch pointers and clears RetroTouch-owned latches without changing the current mode or saved layout. `releaseAllInputs()` is an equivalent alias.

## Building from source

Recommended development environment:

- Android Studio 2025 or newer
- JDK 17
- Android Gradle Plugin 8.11.1
- Gradle 8.13
- compile SDK 36
- minimum SDK 16

Build the library, sample application and lint checks:

```bash
./gradlew :retrotouch:assembleRelease :sample:assembleDebug lint
```

Expected outputs:

```text
retrotouch/build/outputs/aar/retrotouch-release.aar
sample/build/outputs/apk/debug/sample-debug.apk
```

## Compatibility policy

Starting with the 1.0 beta series, compatible updates preserve existing public classes and methods. New functionality is added without silently removing established APIs. Stable action IDs and game/layout IDs should not be renamed after release because saved user layouts depend on them.

Breaking changes, if ever required, will be documented explicitly and reserved for an appropriate major-version update.

## Contributing and bug reports

Bug reports and tested integrations are welcome through [GitHub Issues](https://github.com/Andiweli/RetroTouch/issues).

Please include:

- device and Android version
- game engine or rendering stack
- RetroTouch version
- controller model, if relevant
- reproduction steps
- relevant Logcat output

## Origin and clean-room implementation

RetroTouch's interaction model was informed by Emile Belanger's MobileTouchControls project. RetroTouch is an independent clean-room implementation and contains no MobileTouchControls source code.

This separation is important because MobileTouchControls is licensed under GPLv2, while RetroTouch uses the permissive MIT License for straightforward reuse in both open-source and closed-source Android games.

## License

RetroTouch is released under the [MIT License](LICENSE).

[![Support via PayPal](https://img.shields.io/badge/Support%20via-PayPal-0070BA?logo=paypal&logoColor=white)](https://paypal.me/andiweli)

Copyright © 2026 Andreas 'Andiweli' Stürmer.
