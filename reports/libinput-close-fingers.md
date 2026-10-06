# ThinkPad Helix touchpad: repeated attempts needed for close-finger scrolling

Target: libinput's freedesktop GitLab tracker, after the local pressure calibration has been physically tested in a new desktop session.

Environment: Lenovo ThinkPad Helix 370242P / Synaptics TM2219-002 / PNP LEN0033, libinput 1.31.1, xf86-input-libinput 1.5.0, Xorg / LXQt / Openbox, Ubuntu 26.04.1 LTS. PS/2 transport was selected to avoid a separately observed RMI4 SMBus freeze.

The device advertises two MT slots and finger-count capabilities up to five. Both two-finger and edge scrolling methods are available. Two-finger scrolling was enabled in Xorg, yet the owner initially saw pointer motion instead of scrolling. Clickfinger was enabled, real diagnostic traces subsequently showed `POINTER_SCROLL_FINGER`, two-finger pinch and three-finger swipe events, and the owner eventually confirmed scrolling and gestures.

The remaining complaint was poor naturalness: close fingers required several attempts, even at the center of the pad, and pinch was difficult to trigger. Raw device traces included two-finger count events and both MT slots. The current data is insufficient to determine whether hardware merges close contacts, recognition thresholds are inappropriate, or gesture arbitration is responsible.

Default fresh-context diagnostics:

```text
using pressure-based touch detection (25:30)
palm: pressure threshold is 130
thumb: enabled thumb detection (area, pressure)
```

Most observed MT pressures during interaction were in the 30–76 range, with many in the 48–60 range. No palm/pressure root cause is proven.

Local experiments:

- `ScrollingPixelDistance=10` instead of 15 increases scroll output per movement; it does not change contact recognition.
- Touchégg gesture thresholds and zoom repetition were reduced/tuned; the owner reported improved responsiveness. That result also does not establish a libinput fix.
- A model-scoped `AttrPressureRange=20:15` was prepared. `libinput quirks list` accepted it and a fresh diagnostic context confirmed `(15:20)`. Xorg needs a new session to read it. **Physical validation of this pressure override is pending.** Palm/thumb settings were preserved.

Do not merge this local override as a hardware quirk based on this report alone. Next step: compare close-finger scrolling before/after the new session with a libinput recording and fresh debug-events trace, and evaluate whether the contact count itself changes.
