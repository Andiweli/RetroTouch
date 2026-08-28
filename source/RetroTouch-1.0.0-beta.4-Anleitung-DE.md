# RetroTouch 1.0.0-beta.4 — Einbau- und Bedienungsanleitung

## 1. Was RetroTouch ist

RetroTouch ist eine Touch-Overlay-Bibliothek für Android-Spiele. Sie liegt über der vorhandenen
`View`, `SurfaceView`, `GLSurfaceView`, SDL-Fläche oder einem JNI-gesteuerten Renderer. RetroTouch
übernimmt Darstellung, Multitouch, Layout-Editor und Speicherung. Das Spiel behält seine eigene
Eingabelogik und ordnet die abstrakten RetroTouch-Rückmeldungen seinen bestehenden Tasten, Achsen
oder nativen Funktionen zu.

Unterstützt werden Android 4.1 (API 16) bis Android 16 (API 36), ohne Laufzeitabhängigkeiten und
ohne AndroidX-Zwang. RetroTouch steht unter der MIT-Lizenz.

Die Beta wurde in Breathless mit gleichzeitigem Laufen, Drehen und Schießen, eingerastetem RUN,
Menüsteuerung und Controller-Erkennung auf echter Hardware erprobt. Als nächste Engines folgen
Unreal und Unreal Tournament 99.

## 2. Aufgabenteilung

RetroTouch übernimmt:

- echtes Multitouch anhand der Android-Pointer-IDs;
- bis zu 16 Buttons, zwei Bewegungssticks, zwei Look-Flächen und zwei digitale Steuerkreuze;
- normalisierte Layouts für unterschiedliche Seitenverhältnisse;
- getrennte Layouts für `GAMEPLAY` und `NAVIGATION`;
- den Modus `OFF` mit vollständig durchgereichten Touch-Ereignissen;
- Layout-Editor und Speicherung pro Spiel;
- optionales gleichzeitiges FIRE und LOOK;
- Erkennung vorhandener Hardware-Controller.

Das Spiel übernimmt:

- verfügbare Aktionen und Beschriftungen;
- Standardpositionen der Bedienelemente;
- Zuordnung der Callbacks zu Java, JNI, SDL oder der Engine-Eingabequeue;
- Auswahl des aktuellen Overlay-Modus;
- Spielregeln wie den RUN-Doppeltipp;
- regelmäßige Controller-Prüfung, falls Touch bei vorhandenem Controller verschwinden soll.

## 3. AAR einbinden

`RetroTouch-1.0.0-beta.4.aar` hierhin kopieren:

```text
app/libs/retrotouch.aar
```

Im `build.gradle` des App-Moduls ergänzen:

```groovy
dependencies {
    implementation files('libs/retrotouch.aar')
}
```

Keine RetroTouch-Javaquellen zusätzlich in das Spiel kopieren. Die kleine spielspezifische Bridge
bleibt Teil des Spiels. Bei einem kompatiblen Update muss dadurch nur die AAR ersetzt und das Spiel
neu gebaut werden.

## 4. RetroTouch über die Spielansicht legen

Eine `FrameLayout`-Wurzel verwenden, zuerst das Spiel und zuletzt RetroTouch hinzufügen:

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

Das letzte Element wird über dem Spiel gezeichnet. In `RetroTouchMode.OFF` ist RetroTouch unsichtbar
und nimmt keine Berührungen an. Die bereits vorhandene Menü-Touchsteuerung oder „Tippen zum
Überspringen“ funktioniert dann unverändert.

## 5. Aktionen und Gameplay-Layout anlegen

Action-IDs sind die dauerhafte Verbindung zwischen Layout und Spiel. Empfehlenswert sind kleine
Buchstaben, Ziffern und Unterstriche. Nur Aktionen registrieren, die das Spiel wirklich verarbeitet.

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
        new RetroTouchLayout("eindeutige_spiel_id", gameplay));
```

Positionen und Größen sind von `0.0` bis `1.0` normalisiert. `x` und `y` bezeichnen die Mitte des
Bedienelements. Die Spiel-ID muss eindeutig und dauerhaft bleiben, da sie Teil des Schlüssels für
gespeicherte Benutzerlayouts ist.

Buttonbeschriftungen verwenden in jedem Spiel die kompakte Breathless-Schrift. RetroTouch misst
jeden Text, bricht ihn bei Bedarf automatisch auf höchstens zwei ausgewogene Zeilen um und
verkleinert die Schrift weiter, bis die vollständige Beschriftung in den Button passt. Mit `\n`
kann das Spiel einen bestimmten Umbruch erzwingen; spielspezifische Sonderfälle sind nicht nötig.

`setDefaultLayout()` bleibt als kompatibler Alias für `setGameplayLayout()` erhalten.

## 6. Callbacks mit dem Spiel verbinden

Alle Callbacks laufen im Android-UI-Thread:

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

- `onAction`: `pressed=true` beim Aufsetzen, `false` beim Loslassen oder Abbrechen.
- `onMove`: normalisierte Achsen von `-1.0` bis `1.0`; der sichtbare Ringrand entspricht 100 %.
- `onLook`: relative Bewegung und keine absolute Bildschirmposition.
- `onEditorStateChanged`: Das Spiel kann während der Bearbeitung pausieren oder eigene Eingaben
  unterdrücken.

JNI-Spiele übertragen diese Werte in ihren bestehenden Eingabestatus oder ihre Ereignisqueue.
Keine aufwendige native Spiellogik synchron im UI-Thread ausführen. Drücken und Loslassen immer
behandeln und gehaltene Zustände beim Pausieren der Activity löschen.

## 7. Dauerfeuer beim Drehen

Den Mittelpunkt des FIRE-Buttons innerhalb der Look-Fläche platzieren und die Aktion freigeben:

```java
retroTouch.setLookWhileHoldingAction("fire", true);
```

Der gleiche Finger kann FIRE gedrückt halten und durch Ziehen weiterhin relative Blickbewegungen
erzeugen. FIRE wird erst beim Loslassen oder Abbruch deaktiviert. Andere Buttons funktionieren
normal weiter. Ein zweiter Finger kann gleichzeitig den linken Bewegungsstick bedienen.

## 8. RUN-Doppeltipp und sichtbare Einrastung

Der RUN-Doppeltipp ist absichtlich eine Regel des jeweiligen Spiels und nicht fest in RetroTouch
verdrahtet. Im Action-Callback zwei RUN-Tipps innerhalb des gewünschten Zeitfensters erkennen,
den Immer-laufen-Zustand umschalten und die optische Markierung setzen:

```java
retroTouch.setActionLatched("run", immerLaufenAktiv);
```

Ein eingerasteter RUN-Button wird zwei Alpha-Stufen kräftiger dargestellt. Das Spiel sollte
Immer-laufen nur bei ausgelenktem Bewegungsstick anwenden und den Zustand an denselben Stellen wie
andere gehaltene Eingaben zurücksetzen, etwa bei Pause, Levelwechsel oder Programmende. Dadurch
bleibt RetroTouch auch für Spiele mit anderer RUN-Logik universell.

## 9. Optionales Navigations-Overlay

Spiele mit bereits vorhandener Touch-Menüsteuerung überspringen diesen Abschnitt und verwenden im
Menü `OFF`. Andernfalls:

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
        new RetroTouchLayout("eindeutige_spiel_id_navigation", navigation));
```

Das Steuerkreuz sendet `RetroTouchNavigation.UP`, `.DOWN`, `.LEFT` und `.RIGHT`, intern die IDs
`nav_up`, `nav_down`, `nav_left`, `nav_right`. Diese werden auf die bereits vorhandene Menüsteuerung
des Spiels gelegt.

Den Modus nur bei einer Statusänderung des Spiels umschalten:

```java
retroTouch.setMode(RetroTouchMode.OFF);        // Original-Touch geht ans Spiel
retroTouch.setMode(RetroTouchMode.GAMEPLAY);   // FPS-Steuerung
retroTouch.setMode(RetroTouchMode.NAVIGATION); // D-Pad und frei belegbare Buttons
```

Gameplay- und Navigationslayout werden getrennt gespeichert. Kein Spiel muss alle Modi verwenden.
Für UT99 empfiehlt sich zunächst `OFF` in den vorhandenen Touch-Menüs und `GAMEPLAY` im Level.

## 10. Hardware-Controller

Soll RetroTouch nur ohne vorhandenen Controller sichtbar sein:

```java
RetroTouchMode gewuenschterModus = imLevel
        ? RetroTouchMode.GAMEPLAY
        : RetroTouchMode.OFF;

boolean controllerVorhanden = RetroTouchControllers.isControllerConnected();
retroTouch.setMode(controllerVorhanden
        ? RetroTouchMode.OFF
        : gewuenschterModus);
```

Die Prüfung regelmäßig im ohnehin vorhandenen Status-Update des Spiels wiederholen; Breathless
verwendet ungefähr 200 ms. Dadurch funktionieren An- und Abstecken ohne Neustart. Geprüft werden
Androids Quellen `GAMEPAD` und `JOYSTICK`, nicht Herstellernamen. Das umfasst eingebaute Handheld-
Bedienelemente sowie Xbox-, PlayStation-, USB-, Bluetooth- und generische Controller.

`notifyControllerInput()` ist eine andere, optionale Funktion: Nach einer Controllereingabe wird
RetroTouch drei Sekunden ausgeblendet. Bei strikter Erkennung nach Controller-Verfügbarkeit sollte
dies mit `setAutoHideOnController(false)` abgeschaltet werden, damit es nur eine klare Regel gibt.

## 11. Layout-Editor und Bedienung

Das Settings-Symbol öffnet den Editor. Ein Element auswählen und anschließend:

- `ADD`: Button hinzufügen, maximal 16;
- `ACTION`: zur nächsten vom Spiel registrierten Aktion wechseln;
- `SMALL` / `LARGE`: ausgewähltes Element verkleinern oder vergrößern;
- `ALPHA`: fünf Transparenzstufen durchschalten;
- `DELETE`: ausgewähltes Element entfernen;
- `RESET`: Standardlayout des Spiels wiederherstellen;
- `SAVE`: aktuelles Layout speichern und Editor verlassen.

Elemente lassen sich direkt verschieben. Auch der Settings-Button kann ausgewählt, verschoben,
mit `SMALL` / `LARGE` skaliert und über `ALPHA` transparenter oder sichtbarer gemacht werden. Er
ist der dauerhafte Zugang zum Editor und kann deshalb nicht gelöscht werden: Bei ausgewähltem
Settings-Button ist `DELETE` sichtbar deaktiviert und ohne Funktion.

Die Leiste besitzt oberhalb der Werkzeuge einen mit `↕ DRAG` beschrifteten Griff. Diesen Griff
nach oben oder unten ziehen, wenn die Leiste ein Touch-Element verdeckt. `SAVE` speichert Position,
Größe und Alpha des Settings-Buttons sowie die vertikale Leistenposition getrennt für jedes
Gameplay- oder Navigationslayout. `RESET` setzt auch diese Editor-Elemente auf ihre Vorgaben
zurück. Bereits vorhandene Layouts bleiben kompatibel und erhalten automatisch die bisherigen
Standardpositionen, bis sie erneut gespeichert werden.

Die große Look-Fläche ist nur im Editiermodus sichtbar. Die Layouts liegen in den privaten
App-Einstellungen und bleiben nach einem Neustart erhalten.

## 12. Activity-Lebenszyklus

Empfohlenes Verhalten des Spiels:

- beim Pausieren alle spielseitigen Touch-Zustände lösen;
- beim Verlassen eines Levels den passenden Modus setzen und bei Bedarf Toggle-Zustände löschen;
- die RetroTouch-View nicht in jedem Renderframe neu erzeugen;
- Moduswechsel und RetroTouch-API-Aufrufe im UI-Thread ausführen;
- nach einer Aktion, die das Spiel beenden kann, die spielseitige Quit-Anforderung sofort
  auswerten. Breathless wartete ursprünglich auf eine weitere Eingabe, weil sein natives Quit-Flag
  nach einem RetroTouch-Callback nicht direkt abgeholt wurde.

Wann immer die Engine intern ihre Tasten- oder Achsenzustände zurücksetzt, muss RetroTouch direkt
danach ebenfalls zurückgesetzt werden:

```java
engineResetInput();
retroTouch.resetInputState();
```

Typische Aufrufstellen sind Respawn/Checkpoint-Neustart, Levelwechsel, Laden eines Spielstands,
Fokusverlust und `onPause()` der Activity. Die Methode löst gehaltene Buttons und D-Pad-Richtungen,
sendet `onMove(0, 0)`, verwirft Bewegungs- und Look-Pointer und entfernt die sichtbaren
RetroTouch-Latches. Aktueller Overlay-Modus, Layouts und Editor-Konfiguration bleiben unverändert.
Besitzt die Spiel-Bridge einen eigenen Toggle wie Breathless-Immer-laufen, muss sie diese Variable
am selben Punkt ebenfalls löschen.

`releaseAllInputs()` ist ein gleichwertiger, sprechender Alias.

## 13. RetroTouch aktualisieren

Bei API-kompatiblen Updates:

1. `app/libs/retrotouch.aar` ersetzen;
2. spielspezifische Bridge und Layouts unverändert lassen;
3. Gradle synchronisieren, Projekt bereinigen und APK/AAB neu bauen;
4. die folgende Testliste abarbeiten.

Die AAR wird in APK/AAB einkompiliert und kann nicht in einer bereits installierten App separat
ausgetauscht werden. Quellcode-Anpassungen im Spiel sind nur nötig, wenn das Changelog ausdrücklich
eine inkompatible API-Änderung ankündigt.

## 14. Testliste

Auf einem echten Touchgerät prüfen:

1. Gameplay ohne Controller starten; RetroTouch erscheint sofort.
2. Mit drei Fingern gleichzeitig laufen, drehen und FIRE halten.
3. Den gehaltenen FIRE-Finger in der Look-Fläche ziehen; Drehen und FIRE bleiben aktiv.
4. FIRE loslassen; Schießen stoppt sofort, normales Drehen funktioniert weiter.
5. Am sichtbaren Rand des Sticks wird die maximale Bewegung erreicht.
6. RUN halten und, falls im Spiel umgesetzt, RUN doppeltippen.
7. Beide Layouts bearbeiten, speichern und nach Neustart wieder laden.
8. Im Modus `OFF` funktionieren die ursprünglichen Touch-Menüs.
9. Falls verwendet, D-Pad, OK und Zurück in allen Menüs testen.
10. Eingebaute, USB- und Bluetooth-Controller während des Spiels verbinden und trennen.
11. Pause/Fortsetzen, unterstützte Drehung, Levelwechsel und Quit ohne hängende Eingaben prüfen.
12. Mindestens eine 4:3-, eine 16:9- und eine breite/Automotive-Auflösung testen.
13. Respawn oder Engine-Input-Reset bei gehaltenen Bedienelementen auslösen; der nächste Touch muss
    als vollständig neue, kontinuierliche Eingabe beginnen und keine Aktion darf hängen bleiben.

## 15. Fehlerdiagnose

- **Overlay nicht sichtbar:** Modus, Controller-Erkennung, `setOverlayEnabled`, Reihenfolge im
  `FrameLayout` und registriertes Gameplay-Layout prüfen.
- **Originalmenü erhält keine Tipps:** `RetroTouchMode.OFF` verwenden; ein transparentes
  Gameplay-Layout reicht nicht.
- **Steuerung erscheint verzögert:** Bei Controller-Verfügbarkeitserkennung nicht zusätzlich
  `notifyControllerInput()` für Menüeingaben aufrufen.
- **Button sichtbar, aber ohne Wirkung:** Action-ID registrieren und exakt dieselbe ID im Callback
  behandeln.
- **FIRE endet beim Drehen:** `setLookWhileHoldingAction("fire", true)` aktivieren und den
  Buttonmittelpunkt innerhalb der Look-Fläche platzieren.
- **QUIT benötigt eine weitere Eingabe:** Die Quit-Anforderung des Spiels direkt nach dem
  RetroTouch-Action-Callback verarbeiten.
- **Eingaben hängen nach Pause:** Spielseitige Zustände bei Pause/Abbruch löschen und nicht nur auf
  ein späteres Finger-Up vertrauen.
- **Altes Benutzerlayout verdeckt neue Standardbuttons:** Während der Entwicklung eine neue
  Layout-Spiel-ID verwenden oder im Editor `RESET` wählen.

## 16. Empfohlener erster Unreal-/UT99-Einbau

Mit der kleinsten sinnvollen Integration beginnen:

1. vorhandene UT99-Touchmenüs beibehalten und außerhalb des Gameplays `OFF` setzen;
2. eine Java-Bridge oberhalb der Render-View anlegen;
3. Aktionen in den bestehenden Unreal-Eingabepfad einspeisen statt Spielfunktionen direkt
   aufzurufen;
4. Bewegungsstick auf Vorwärts/Seitwärts und relatives LOOK auf Yaw/Pitch abbilden;
5. FIRE-and-LOOK nur für Primary Fire aktivieren;
6. RetroTouch bei jedem vorhandenen Hardware-Controller unterdrücken;
7. vor der Feineinstellung der Empfindlichkeit Frame-Pacing und Pointer-Verhalten auf ChromeOS
   prüfen.

Damit bleiben mögliche Engine-Probleme sauber vom Overlay getrennt und spätere Updates können
weiterhin ausschließlich durch Austausch der AAR erfolgen.
