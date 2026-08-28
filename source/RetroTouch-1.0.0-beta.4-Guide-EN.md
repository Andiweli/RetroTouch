# RetroTouch 1.0.0-beta.4 — integration and usage guide

## 1. What RetroTouch is

RetroTouch is an Android touch-overlay library for games. It is added above the game's existing
`View`, `SurfaceView`, `GLSurfaceView`, SDL surface or JNI-driven renderer. RetroTouch handles
drawing, multitouch, layout editing and persistence. The game keeps ownership of its input logic
and maps RetroTouch's abstract callbacks to its existing buttons, keys, axes or native functions.

The library supports Android 4.1 (API 16) through Android 16 (API 36), has no runtime dependencies
and does not require AndroidX. The implementation is MIT-licensed.

The beta was validated in Breathless with simultaneous movement, looking and firing, latched RUN,
menu navigation and physical-controller detection. Further engine testing starts with Unreal and
Unreal Tournament 99.

## 2. Responsibilities

RetroTouch provides:

- pointer-ID-based multitouch;
- up to 16 buttons, two movement sticks, two look zones and two digital D-pads;
- normalized layouts for different aspect ratios;
- independent `GAMEPLAY` and `NAVIGATION` layouts;
- `OFF` mode with touch pass-through;
- a built-in editor and per-game saved layouts;
- optional FIRE-and-LOOK behavior;
- controller-presence detection.

The host game provides:

- the list of supported actions and their labels;
- default control positions;
- callback mapping to Java, JNI, SDL or the engine's input queue;
- the current overlay mode;
- any game rule such as RUN double-tap/toggle behavior;
- periodic controller-presence checks if automatic controller suppression is wanted.

## 3. Add the AAR

Copy `RetroTouch-1.0.0-beta.4.aar` to:

```text
app/libs/retrotouch.aar
```

Add the dependency to the app module's `build.gradle`:

```groovy
dependencies {
    implementation files('libs/retrotouch.aar')
}
```

No RetroTouch Java source files should be copied into the game. Keep the game-specific bridge in
the game project. A compatible update then consists of replacing the AAR and rebuilding the game.

## 4. Place RetroTouch above the game view

Use a `FrameLayout`, add the game first and RetroTouch last:

```java
FrameLayout.LayoutParams fill = new FrameLayout.LayoutParams(
        FrameLayout.LayoutParams.MATCH_PARENT,
        FrameLayout.LayoutParams.MATCH_PARENT);

FrameLayout root = new FrameLayout(this);
root.addView(gameView, fill);

RetroTouchView retroTouch = new RetroTouchView(this);
root.addView(retroTouch, fill);
setContentView(root);
```

The last child is drawn above the game. In `RetroTouchMode.OFF`, RetroTouch is not drawn and does
not consume touches, so the game's original menu or tap-to-skip handling continues to work.

## 5. Register actions and create the gameplay layout

Action IDs are the stable contract between the layout and the game. Use lowercase IDs containing
letters, digits and underscores. Do not expose actions the game cannot process.

```java
retroTouch.registerAction("fire", "Fire");
retroTouch.registerAction("use", "Use");
retroTouch.registerAction("run", "Run");
retroTouch.registerAction("weapon_next", "Next\nWeapon");
retroTouch.registerAction("menu", "Menu");

List<RetroTouchControl> gameplay = new ArrayList<RetroTouchControl>();
gameplay.add(RetroTouchControl.lookZone(
        "look", 0.72f, 0.48f, 0.56f, 0.78f));
gameplay.add(RetroTouchControl.moveStick(
        "move", 0.16f, 0.76f, 0.25f));
gameplay.add(RetroTouchControl.button(
        "fire_button", "fire", "Fire", 0.89f, 0.71f, 0.15f));
gameplay.add(RetroTouchControl.button(
        "use_button", "use", "Use", 0.77f, 0.80f, 0.11f));

retroTouch.setGameplayLayout(
        new RetroTouchLayout("unique_game_id", gameplay));
```

Coordinates and sizes are normalized from `0.0` to `1.0`. `x` and `y` identify the control's
center. Use a unique and permanent game ID because it is also part of the saved-layout key.

Button labels use the compact Breathless font style in every game. RetroTouch measures each label,
automatically wraps it to at most two balanced lines when necessary and reduces the font size until
the complete text fits inside the button. A host may force the line break with `\n`; no
game-specific label exceptions are needed.

`setDefaultLayout()` remains a compatibility alias for `setGameplayLayout()`.

## 6. Map callbacks to the game

All callbacks run on Android's UI thread:

```java
retroTouch.setListener(new RetroTouchAdapter() {
    @Override public void onAction(String actionId, boolean pressed) {
        nativeTouchAction(actionId, pressed);
    }

    @Override public void onMove(float x, float y) {
        nativeTouchMove(x, y);
    }

    @Override public void onLook(float deltaX, float deltaY) {
        nativeTouchLook(deltaX, deltaY);
    }

    @Override public void onEditorStateChanged(boolean editing) {
        nativeSetTouchEditorOpen(editing);
    }
});
```

- `onAction`: `pressed=true` on finger down and `false` on finger up/cancel.
- `onMove`: normalized axes from `-1.0` to `1.0`; the visible ring edge is full deflection.
- `onLook`: relative motion, not an absolute screen position.
- `onEditorStateChanged`: lets the host pause gameplay or suppress its own input while editing.

JNI games should transfer these values into their existing input state or event queue. Avoid doing
heavy native work synchronously on the UI thread. Always handle both press and release, and clear
held native states when the Activity pauses.

## 7. FIRE while turning

Place the FIRE button's center inside the look zone and enable its action:

```java
retroTouch.setLookWhileHoldingAction("fire", true);
```

The same finger can now press FIRE and continue producing relative look motion without releasing
FIRE. FIRE becomes inactive only when that finger is lifted or cancelled. Other buttons keep their
normal behavior. Movement can simultaneously come from another finger on the left stick.

## 8. RUN double-tap and latched display

RUN double-tap is deliberately a host-game rule, not hard-coded library behavior. In the action
callback, detect two RUN presses within the chosen interval, toggle the run state and update the
visual latch:

```java
retroTouch.setActionLatched("run", alwaysRunEnabled);
```

The latched RUN button is drawn two opacity steps stronger. The host should apply always-run only
while the movement stick is non-zero and reset it at the same points as other held input states
(pause, level transition or shutdown). This keeps RetroTouch usable by games whose RUN semantics
differ from Breathless.

## 9. Optional navigation overlay

Games with their own touch-capable menus can skip this section and use `OFF` in menus. Otherwise:

```java
retroTouch.registerAction("menu_ok", "OK");
retroTouch.registerAction("menu_back", "Back");

List<RetroTouchControl> navigation = new ArrayList<RetroTouchControl>();
navigation.add(RetroTouchControl.dPad(
        "navigation", 0.18f, 0.72f, 0.30f));
navigation.add(RetroTouchControl.button(
        "ok", "menu_ok", "OK", 0.86f, 0.68f, 0.14f));
navigation.add(RetroTouchControl.button(
        "back", "menu_back", "Back", 0.73f, 0.82f, 0.11f));

retroTouch.setNavigationLayout(
        new RetroTouchLayout("unique_game_id_navigation", navigation));
```

The D-pad emits `RetroTouchNavigation.UP`, `.DOWN`, `.LEFT` and `.RIGHT` (string values `nav_up`,
`nav_down`, `nav_left`, `nav_right`). Map these to the game's existing menu input.

Switch modes only when the game's state changes:

```java
retroTouch.setMode(RetroTouchMode.OFF);        // original touch passes to the game
retroTouch.setMode(RetroTouchMode.GAMEPLAY);   // FPS controls
retroTouch.setMode(RetroTouchMode.NAVIGATION); // D-pad and configurable buttons
```

Gameplay and navigation layouts are saved independently. A game is not required to use all modes.
For UT99, the expected first integration is `OFF` in its existing touch menus and `GAMEPLAY` in a
level.

## 10. Physical controllers

To show touch controls only when no controller is available:

```java
RetroTouchMode wantedMode = inLevel
        ? RetroTouchMode.GAMEPLAY
        : RetroTouchMode.OFF;

boolean hasController = RetroTouchControllers.isControllerConnected();
retroTouch.setMode(hasController ? RetroTouchMode.OFF : wantedMode);
```

Repeat the presence check during the host's normal state update (Breathless uses about 200 ms) to
handle hotplug without restarting. Detection uses Android `GAMEPAD` and `JOYSTICK` sources, so it
covers built-in handheld controls and Xbox, PlayStation, USB, Bluetooth and generic controllers
without vendor lists.

`notifyControllerInput()` is a different, optional feature: it temporarily hides RetroTouch for
three seconds after a controller event. If strict presence-based suppression is used, disable this
additional behavior with `setAutoHideOnController(false)` to keep the policy unambiguous.

## 11. Editor and player usage

The settings icon opens the editor. Select a control and use:

- `ADD`: add a button, up to 16;
- `ACTION`: select the next registered game action;
- `SMALL` / `LARGE`: resize the selection;
- `ALPHA`: cycle through five opacity levels;
- `DELETE`: remove the selection;
- `RESET`: restore the host game's default layout;
- `SAVE`: persist the current mode's layout and close the editor.

Controls can be dragged directly. The settings button can also be selected and moved, resized with
`SMALL` / `LARGE`, and adjusted with `ALPHA`. It is the permanent editor entry point, so it cannot
be removed: while settings is selected, `DELETE` is visibly disabled and has no effect.

The strip has a `↕ DRAG` handle above its tools. Drag that handle up or down whenever the strip
covers a touch control. `SAVE` stores the settings-button position, size and opacity plus the
toolbar's vertical position independently for every gameplay or navigation layout. `RESET` also
restores these editor elements to their defaults. Existing saved layouts remain compatible and
use the previous default editor positions until saved again.

The large look area is shown only in edit mode. Saved layouts are private app preferences; they
remain available after restarting the game.

## 12. Activity lifecycle

Recommended host behavior:

- on pause, cancel or release all game-side touch states;
- when leaving a level, switch to the correct mode and clear host-side toggles if appropriate;
- do not recreate the RetroTouch view on every rendered frame;
- perform mode changes and RetroTouch API calls on the UI thread;
- after an action that can quit the game, immediately process the game's quit request. Breathless
  originally waited for another input because its native quit flag was not consumed after a
  RetroTouch callback.

Whenever the engine internally resets its key or axis state, reset RetroTouch immediately after it:

```java
engineResetInput();
retroTouch.resetInputState();
```

Typical call sites are respawn/checkpoint restart, level transitions, savegame loading, focus loss
and the Activity's `onPause()`. The method releases held buttons and D-pad directions, emits
`onMove(0, 0)`, discards movement and look pointers, and clears RetroTouch's visual latches. It does
not change the current overlay mode, layouts or editor configuration. If the game bridge owns a
toggle such as Breathless always-run, clear that game-owned variable at the same point.

`releaseAllInputs()` is an equivalent descriptive alias.

## 13. Updating RetroTouch

For API-compatible updates:

1. replace `app/libs/retrotouch.aar`;
2. keep the game-owned bridge and layouts unchanged;
3. sync Gradle, clean and rebuild the APK/AAB;
4. run the regression checklist below.

The AAR is compiled into the APK/AAB and cannot be replaced inside an already installed app.
Source updates are only needed when the changelog explicitly announces a breaking API migration.

## 14. Test checklist

Test on a real touch device:

1. Enter gameplay without a controller; RetroTouch appears immediately.
2. Move, turn and hold FIRE simultaneously with three fingers.
3. Drag the held FIRE finger inside the look area; turning continues and FIRE remains active.
4. Release FIRE; firing stops immediately while ordinary look still works.
5. Verify full movement at the visible stick edge.
6. Test RUN hold and, if implemented by the host, RUN double-tap.
7. Open, edit, save and reload both layouts.
8. Test `OFF` touch pass-through in the original menus.
9. If used, test D-pad, OK and Back in all relevant menus.
10. Connect and disconnect built-in/USB/Bluetooth controllers during runtime.
11. Pause/resume, rotate if supported, change levels and quit without stuck inputs.
12. Test at least one 4:3, one 16:9 and one wide/Automotive resolution.
13. Trigger respawn or an engine input reset while controls are held; verify that the next touch
    starts a fresh continuous input and no action remains stuck.

## 15. Troubleshooting

- **Overlay is not visible:** check the current mode, controller detection, `setOverlayEnabled`,
  child order in the `FrameLayout`, and whether a gameplay layout was registered.
- **Original menu no longer receives taps:** use `RetroTouchMode.OFF`, not merely a transparent
  gameplay layout.
- **Input starts only after a delay:** do not call `notifyControllerInput()` for menu events when
  using presence-based controller suppression.
- **A button draws but has no effect:** register its action ID and handle the exact same ID in the
  callback.
- **FIRE stops when turning:** enable `setLookWhileHoldingAction("fire", true)` and ensure the
  button center lies inside the look-zone rectangle.
- **QUIT waits for another input:** consume the host game's quit request directly after the
  RetroTouch action callback.
- **Controls remain held after pause:** clear game-side input on pause/cancel; never depend only on
  a later finger-up event.
- **Old custom layout hides new default controls:** use a new layout game ID during development or
  choose `RESET` in the editor.

## 16. Recommended Unreal/UT99 first pass

Start with the narrowest integration:

1. keep the existing UT99 menu touch implementation and select `OFF` outside gameplay;
2. add one Java bridge above the game's render view;
3. map actions to the existing Unreal input path rather than calling gameplay functions directly;
4. map movement to forward/strafe axes and relative look to yaw/pitch deltas;
5. enable FIRE-and-LOOK only for the primary-fire action;
6. suppress RetroTouch while any hardware controller is present;
7. validate frame pacing and pointer capture on ChromeOS before tuning sensitivity.

This isolates engine-specific problems from the overlay and preserves the AAR-only update path.
