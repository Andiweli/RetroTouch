# RetroTouch

> A lightweight, configurable and reusable touch-control overlay for Android games.

[![Android API](https://img.shields.io/badge/Android-API%2016--36-3DDC84?logo=android&logoColor=white)](https://developer.android.com/)
[![Status](https://img.shields.io/badge/Status-1.0.0--beta.9-orange)](#project-status)
[![Language](https://img.shields.io/badge/Java-8-ED8B00?logo=openjdk&logoColor=white)](https://www.java.com/)
![AI](https://img.shields.io/badge/AI-assisted%20coding-6e7781)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

[![Support via PayPal](https://img.shields.io/badge/Support%20via-PayPal-0070BA?logo=paypal&logoColor=white)](https://paypal.me/andiweli)


RetroTouch is an engine-agnostic Android touch-controls library for games and source ports. It adds a resolution-independent virtual joystick, configurable action buttons, relative FPS look controls and optional menu navigation above an existing Android game view.

The overlay is delivered as a replaceable Android Archive (`.aar`) and is suitable for Java, JNI, SDL, OpenGL, `SurfaceView`, `GLSurfaceView` and native game engines. The game keeps ownership of its input system: RetroTouch emits stable action IDs and normalized movement/look callbacks, while a small game-specific bridge maps them to keys, axes or engine events.

RetroTouch was created for retro FPS ports that need modern Android multitouch controls without hard-coding a different overlay into every game.

<img width="1536" height="864" alt="image" src="https://github.com/user-attachments/assets/4ca08bab-cddf-4aa9-8c40-598676343f4a" />
<img width="1536" height="864" alt="image" src="https://github.com/user-attachments/assets/ca590585-8451-4f00-bddb-244ef07e0c3b" />
<img width="1536" height="864" alt="image" src="https://github.com/user-attachments/assets/d1d895b4-6f10-4eb6-b3f6-f6b210013eb1" />


## Key features

**Platform**
- Android 4.1 / API 16 through Android 16 / API 36
- no AndroidX requirement and no external runtime dependencies
- compatible AAR updates without copying RetroTouch source into each game
- MIT license for use in open-source and commercial projects

**Controls**
- true pointer-ID-based multitouch
- up to 16 configurable action buttons
- up to two analog sticks, two relative-look zones and two digital D-pads
- a move stick plus an optional second analog stick, for example a GameCube C-stick, that reports separately
- stick knobs that run with their center along the ring
- normalized layouts for 4:3, 16:9, 20:9, tablets and ultrawide Automotive displays

**Appearance**
- automatic one- or two-line button labels with dynamic font fitting
- large, centered one-character labels such as `A` or `B` that scale with the button
- rectangle buttons with slightly rounded corners for START, SELECT, MENU, shoulders and triggers
- optional game-defined colors per action, per control or for the whole overlay, including the border

**Modes and layouts**
- independent `OFF`, `GAMEPLAY` and optional `NAVIGATION` modes
- persistent per-game gameplay and navigation layouts

**Layout editor**
- built-in editor for adding, moving, resizing, remapping and changing opacity
- every control can be dragged right up to each screen edge
- a selection frame that fits the visible control
- movable, scalable and opacity-adjustable Settings button
- vertically draggable editor toolbar

**Gameplay support**
- FPS-friendly simultaneous movement, turning and held FIRE
- host-controlled latched-button display, suitable for always-run modes
- public input-reset API for respawn, level changes and engine input resets
- controller detection for built-in, USB, Bluetooth, Xbox, PlayStation and generic gamepads

## Typical use cases

RetroTouch is designed for:

- Android game ports and source ports
- retro FPS and action games
- emulator-style ports with console button layouts
- SDL and JNI-based games
- OpenGL and custom-rendered Android games
- Unreal Engine 1 ports, including Unreal and Unreal Tournament 99
- Android handhelds, phones, tablets, ChromeOS and Android Automotive
- projects that need an editable virtual joystick and gamepad-style touch overlay

## Project status

The current release is **RetroTouch 1.0.0-beta.9**.

| Version | Date | Highlights |
|---|---|---|
| beta.9 | 2026-11-03 | Controls reach every screen edge, thicker borders, stick knobs run along the ring |
| beta.8 | 2026-11-03 | Rectangle buttons for START, SELECT, MENU, shoulders and triggers; border colors; fitted editor frame |
| beta.7 | 2026-10-26 | Optional second analog stick that reports separately (`onStick`) |
| beta.6 | 2026-10-19 | Large, centered one-character labels |
| beta.5 | 2026-10-19 | Optional game-defined colors |
| beta.4 | 2026-08-28 | Movable settings button, draggable editor toolbar |
| beta.3 | 2026-08-28 | Input reset for respawn, level changes and focus loss |
| beta.2 | 2026-08-27 | Universal one- or two-line button labels |
| beta.1 | 2026-08-27 | First public beta with the complete core feature set |

The complete core feature set has been tested with the Android port of **Breathless**, including multitouch movement, relative look, held FIRE while turning, always-run indication, D-pad menu navigation, controller detection and editable layouts. The additions of beta.5 to beta.9 are covered by automated rendering tests against the released AAR. The beta phase continues while RetroTouch is tested across additional engines and input stacks.

## How RetroTouch is structured

RetroTouch intentionally separates reusable overlay code from game-specific input logic:

1. **`retrotouch.aar`** — reusable RetroTouch library
2. **game bridge** — maps RetroTouch callbacks to the game's input system
3. **game layout** — defines available actions, labels, default positions and, optionally, colors and button shapes

With this structure, a compatible RetroTouch update normally requires only replacing `app/libs/retrotouch.aar` and rebuilding the APK or AAB. The game bridge and native engine code remain unchanged.

The AAR is compiled into the final application. It cannot be exchanged inside an already installed APK without rebuilding and reinstalling the game.

## Installation with the AAR

The current AAR is in this repository as [`source/RetroTouch-1.0.0-beta.9.aar`](source/RetroTouch-1.0.0-beta.9.aar). Copy it into the application module as:

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

The `source/` folder also contains the complete source package (`RetroTouch-1.0.0-beta.9-source.zip`) and detailed integration guides in [English](source/RetroTouch-1.0.0-beta.9-Guide-EN.md) and [German](source/RetroTouch-1.0.0-beta.9-Anleitung-DE.md).

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

## Labels and button shapes

Labels come from the game. RetroTouch wraps long labels to at most two balanced lines, shrinks the font until the label fits and honors an explicit `\n`. Labels with exactly one character, such as `A`, `B`, `X` or `Y`, are drawn twice as large, scale with the button and sit exactly in the center.

RetroTouch chooses a button's shape from its label, so existing games get it with a new AAR alone:

| Label | Shape |
|---|---|
| `START`, `SELECT`, `MENU` | compact rectangle |
| `L`, `R`, `ZL`, `ZR`, `L1`, `R1`, `L2`, `R2`, `LB`, `RB`, `LT`, `RT` | slightly larger rectangle |
| everything else | round |

Rectangles have slightly rounded corners and are touched on their whole area. A game can choose the shape per action, for example a rectangular GameCube `Z` button:

```java
retroTouch.setButtonShape("z", RetroTouchButtonShape.SHOULDER); // RECTANGLE, ROUND, AUTO
```

## Optional colors

By default all controls use RetroTouch's neutral grey. A game can supply its own colors, for example a green A and a red B button as on a GameCube controller. Like labels, colors come from the game: players cannot change them, and `ALPHA` still controls opacity only.

```java
// One fill color is enough: pressed color, border and readable text are derived.
RetroTouchColors green = RetroTouchColors.of(Color.rgb(31, 166, 74));
RetroTouchColors red = RetroTouchColors.of(Color.rgb(217, 43, 43));

// On the action: every button mapped to "a" or "b" uses these colors.
retroTouch.registerAction("a", "A", green);
retroTouch.registerAction("b", "B", red);

// Only a different border (ring) color.
retroTouch.registerAction("x", "X",
        RetroTouchColors.of(Color.rgb(208, 208, 208)).withBorderColor(Color.rgb(255, 215, 0)));

// On a single control: the optional last parameter of button(), moveStick() or dPad().
gameplay.add(RetroTouchControl.moveStick(
        "move", 0.16f, 0.76f, 0.25f, RetroTouchColors.of(Color.rgb(90, 90, 90))));

// For everything else: one base color for all buttons, sticks and D-pads.
retroTouch.setDefaultColors(RetroTouchColors.of(Color.rgb(40, 104, 200)));
```

`RetroTouchColors` holds a fill, pressed, border and text color. `RetroTouchColors.of(fill, pressed, text)` and `new RetroTouchColors(fill, pressed, border, text)` set them explicitly. The most specific setting wins: control, then action, then the default colors, then the built-in grey.

Colors are not stored in saved player layouts. RetroTouch re-applies the game's colors on every load, so existing layouts keep working and pick up color changes from a game update.

## Analog sticks

The move stick reports through `onMove` with values from `-1.0` to `1.0`. The knob runs with its center along the ring, and full deflection is reached at the ring.

A game that needs a second, independent stick, such as the C-stick of a GameCube controller, registers a stick action. In the default layout it adds a button with that action, and RetroTouch turns it into a stick automatically:

```java
retroTouch.registerStickAction("c_stick", "C", RetroTouchColors.of(Color.rgb(245, 200, 0)));

gameplay.add(RetroTouchControl.button(
        "c_stick", "c_stick", "C", 0.80f, 0.70f, 0.20f));
```

The stick reports through `RetroTouchStickListener.onStick(actionId, x, y)` with values from `-1.0` to `1.0`, y pointing down, and `0, 0` on release. The listener implements this interface next to `RetroTouchListener`, so it must be a named class:

```java
final class TouchBridge extends RetroTouchAdapter implements RetroTouchStickListener {
    @Override
    public void onMove(float x, float y) {
        nativeTouchMove(x, y);
    }

    @Override
    public void onStick(String actionId, float x, float y) {
        if ("c_stick".equals(actionId)) nativeTouchCStick(x, y);
    }
}

retroTouch.setListener(new TouchBridge());
```

A layout holds at most two sticks, including the move stick, and 16 buttons. Stick actions never arrive through `onAction`.

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

- add action buttons (`ADD`)
- change a button's registered action (`ACTION`), which also turns a button into a second analog stick and back
- move controls right up to each screen edge
- resize controls (`SMALL` / `LARGE`)
- cycle through opacity levels (`ALPHA`)
- delete ordinary controls (`DELETE`)
- reset the current layout (`RESET`)
- save the current gameplay or navigation layout (`SAVE`)

The white selection frame fits the visible control: a square around round buttons, sticks and D-pads, and the rectangle around rectangle buttons. The Settings button itself can be selected, moved, resized and adjusted with `ALPHA`, but cannot be deleted. The editor toolbar can be dragged vertically when it covers another control. All settings are stored privately per game/layout ID.

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

The call releases held actions, sends neutral movement for the move stick and every stick action, removes active touch pointers and clears RetroTouch-owned latches without changing the current mode or saved layout. `releaseAllInputs()` is an equivalent alias.

## Public API overview

| Class | Purpose |
|---|---|
| `RetroTouchView` | The overlay: actions, layouts, modes, colors, shapes, latches, input reset |
| `RetroTouchControl` | Factory for buttons, move sticks, look zones and D-pads |
| `RetroTouchLayout` | A game's set of controls with a stable game/layout ID |
| `RetroTouchAction` | An action ID with its label and optional colors |
| `RetroTouchListener` / `RetroTouchAdapter` | Callbacks: `onAction`, `onMove`, `onLook`, `onEditorStateChanged` |
| `RetroTouchStickListener` | Optional callback for second analog sticks: `onStick` (beta.7) |
| `RetroTouchColors` | Fill, pressed, border and text colors (beta.5, `withBorderColor` since beta.8) |
| `RetroTouchButtonShape` | `AUTO`, `ROUND`, `RECTANGLE`, `SHOULDER` (beta.8) |
| `RetroTouchMode` | `OFF`, `GAMEPLAY`, `NAVIGATION` |
| `RetroTouchNavigation` | Stable D-pad action IDs `nav_up`, `nav_down`, `nav_left`, `nav_right` |
| `RetroTouchControllers` | `isControllerConnected()` |

## Building from source

Recommended development environment:

- Android Studio 2025 or newer
- JDK 17
- Android Gradle Plugin 8.11.1
- Gradle 8.13
- compile SDK 36
- minimum SDK 16

Unpack the source package and build the library, sample application and lint checks:

```bash
./gradlew :retrotouch:assembleRelease :sample:assembleDebug lint
```

Expected outputs:

```text
retrotouch/build/outputs/aar/retrotouch-release.aar
sample/build/outputs/apk/debug/sample-debug.apk
```

## Compatibility policy

Starting with the 1.0 beta series, compatible updates preserve existing public classes and methods. New functionality is added without silently removing established APIs, and every beta so far has been an AAR-only update. Saved player layouts use format 2 throughout the beta series. Stable action IDs and game/layout IDs should not be renamed after release because saved user layouts depend on them.

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
