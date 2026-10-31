# Prop Radar: Final Handoff Plan

**Change from the last version:** the app now starts on the **front camera** so you can test on yourself while watching the screen. A FLIP button switches to the rear camera for real use. Everything else is as approved.

The basic scan sequence is included. Since the 8BitDo model isn't confirmed yet, the controller comes with a test panel that shows what each button sends, so the mapping can be adjusted for any model.

---

## Goal

A single web page in Safari on an iPhone that turns the phone into a targeting scope:

- **Beeping:** faster, higher, and louder as a face gets closer and more centered in the reticle.
- **Controls:** every function has a large on-screen button for testing, and the 8BitDo controller drives the same functions.
- **Gyro sweep:** added at the end.

**In scope:** camera with front/rear switching, face detection, proximity beeping, lock-on, a basic scan-and-result sequence, controller support, gyro sweep, and on-screen debug controls.

**Out of scope:** reference-photo matching and the physical build.

---

## How you'll use it

1. Open the link in Safari. Optionally use "Add to Home Screen" so it runs fullscreen.
2. Tap **START** to unlock the camera, sound, and motion sensors. iOS requires a tap for all three.
3. **Testing:** the front camera is on, so hold the phone at arm's length facing you.
   - Bring it closer and the beeping ramps up.
   - Move your face around the reticle to test centering.
   - Someone standing behind you also counts as a target.
4. **Real use:** tap FLIP to switch to the rear camera, mount the phone on the prop, and aim.

---

## Technical approach

- **One self-contained HTML file.** Google's MediaPipe face detection loads from a CDN, and sound is generated live with Web Audio.
- **Hosting:** first try publishing it as a Claude artifact for an instant link. If the camera is blocked inside the artifact's sandbox, the fallback is Netlify Drop: drag the file onto their page and get an HTTPS link in about 30 seconds, no account required.

---

## Phase 1: Version 1

### Camera

- **Front camera by default,** with the preview mirrored so it behaves like a selfie view.
- **Mirroring is handled internally:** the reticle, face boxes, and any left/right cues match what you see.
- **FLIP** switches to the rear camera, unmirrored. Making the rear camera the default later is a one-line change.
- **Screen stays awake** while running.

### Face detection

- Runs at about 15–20 frames per second and draws a box around each face.
- **Range risk:** the current model is tuned for faces within about 2–3 m. That's fine for front-camera testing, but it may struggle across a room once you switch to the rear camera. If so, switch to the longer-range model or to body detection, which is built as a one-line swap.

### Target before lock-on

The face nearest the reticle, with size as the tiebreaker.

### Signal (0–100)

Combines how much of the frame the face fills with how close it is to the reticle.

### Beeps

- The signal controls spacing (about 1 per second up to a near-continuous rattle), pitch, and volume.
- Two sound styles: Geiger clicks or sonar ping.

### HUD

- Reticle, signal bar, and a label word (default "GAYDAR", editable).
- Status text such as "SCANNING…", "TARGET ACQUIRED", and "TARGET LOST".

### Debug overlay (toggle)

- Frames per second, face count, and face size.
- Signal value, active camera, and last controller input.

### On-screen test buttons

| Button | Does |
|---|---|
| START | Unlocks camera, sound, and motion |
| FLIP | Switches between front and rear camera |
| LOCK / RELEASE | Locks onto the current target |
| SCAN | Runs the scan sequence |
| RESULT ▸ | Cycles the forced result: Auto / Low / High / "Too fabulous to measure" |
| SOUND | Switches between Geiger and sonar |
| MUTE | Turns sound on and off |
| LABEL | Cycles preset words, or lets you type your own |
| DEBUG | Shows or hides the overlay |
| Sensitivity slider | Sets how close counts as "max signal" |

### Lock-on

- Press LOCK and the app follows that one face frame to frame, ignoring everyone else.
- A brief dropout keeps the lock for about 1 second before "TARGET LOST".

### Scan sequence

- The meter climbs, stutters, and pauses dramatically, then shows the result.
- The forced result always wins. On Auto, the result is based on the signal.

---

## Phase 2: Controller

- **Keyboard mode first:** set the 8BitDo to keyboard mode, which Safari on iPhone handles most reliably.
- **Gamepad mode as a fallback:** the code also supports gamepad mode, though iOS support for it varies by controller.
- **Controller test panel:** shows the last key or button received, so you can see what each physical button sends.
- **Button mapping (8BitDo Zero 2):** editable as `CONFIG.keys` (keyboard mode) and `CONFIG.pad` (gamepad mode) at the top of the script.

  | Button | Does |
  |---|---|
  | D-pad ◀ / ▶ | Select the nearest person left / right (pink); if locked, move the lock (orange) |
  | A | Lock / release |
  | B | Scan; press again on the result to dismiss it |
  | X | Forced result GAY → STRAIGHT → RANDOM |
  | Y | Tracking FACE → BODY → AUTO (face up close, body when no face is found) |
  | R | Forced result TOO FAB |
  | L | Flip camera |
  | D-pad ▲ | Mark / unmark bearing |
  | D-pad ▼ | Debug overlay |
  | Select | Show / hide control panel |
  | Start | Mute |

- **Secret mode dot:** a tiny dot under the badges shows the forced result (pink = GAY, blue = STRAIGHT, white = RANDOM, faint = AUTO, rainbow = TOO FAB).
- **Checking the map:** turn DEBUG on (D-pad ▼ or the on-screen button) and press each button; the INPUT line shows the key code or pad index and the action it triggered, or "(unmapped)". Copy any unmapped code into `CONFIG.keys`.
- **D-pad in gamepad mode:** the app reads the D-pad from buttons 12–15, the stick axes or the hat axis, whichever the controller sends. If it still shows nothing, check the Zero 2 manual for the D-pad / stick emulation switch.
- **On-screen buttons stay active** alongside the controller.

---

## Phase 3: Gyro sweep

- **Mark:** on lock-on (or with a MARK button), the app saves the direction the phone is facing.
- **Sweep:** turn away and back. Pings intensify as you swing back toward the marked direction, and a HUD arrow shows which way to turn.
- **Direction source:** relative turning measured by the gyroscope, which is smoother indoors than the compass. RE-ZERO resets drift.
- **Modes:** the sweep can run alone, or combine with face detection so the lock resumes when the face reappears in view.
- **Front-camera testing:** the marked direction is wherever the camera currently points, so the sweep works the same with either camera.

---

## iPhone notes

- **Silent switch:** it can mute web audio. The app uses the audio mode that plays through silent mode where iOS allows it. If you hear nothing, check the switch first.
- **Permissions:** if camera or motion access is denied, re-enable it under Settings → Safari.
- **Battery and heat:** detection is capped at 15–20 frames per second to limit warming.
- **Lighting:** dim lighting reduces detection reliability.

---

## Test checklist

1. **Front camera:** beeping ramps up as you bring your face closer and center it in the reticle.
2. **Mirroring:** moving your face left moves the box left on screen.
3. **FLIP:** the rear camera works, and beeping still responds to someone walking toward you.
4. **Lock-on:** the lock holds on one face while another passes through the frame.
5. **Scan:** a forced result always wins.
6. **Controller:** each button shows up in the test panel and triggers its function.
7. **Sweep:** mark, turn 180°, and turn back. The pings should peak on the marked direction.

---

## Deliverables

1. **Phase 1** (version 1, lock-on, and scan) as a link or file for testing.
2. **Phase 2** added to the same page.
3. **Phase 3** added to the same page.

Each phase comes with a short "what changed and what to test" note.

---

## Open item

- **8BitDo model** (Micro, Zero 2, or other), which sets the default key map.
