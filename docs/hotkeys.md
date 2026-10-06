# Lenovo hotkeys under LXQt / X11

The owner initially reported that no special keys responded. During a second physical test, Linux received volume-down, brightness-down/up, display-switch and configuration events. The owner then confirmed that the tested special functions work when pressed with **Fn**. The current Fn mode produces standard F1–F12 without Fn. Fn+Esc switches the firmware's Fn Lock mode; this was not changed programmatically.

| Icon key | Desktop action |
|---|---|
| F1 | Mute/unmute speakers through the existing LXQt panel |
| F2 / F3 | Volume down / up through the existing LXQt panel |
| F4 | Toggle PipeWire's default microphone mute, with a notification |
| F5 / F6 | Brightness down / up, with a notification |
| F7 | Open LXQt display configuration |
| F8 | Toggle Wi-Fi, with a notification (does not toggle Bluetooth) |
| F9 | Open LXQt configuration center |
| F10 | Toggle LXQt's application runner through its existing Super+R shortcut |
| F11 | Open Openbox's running-window list |
| F12 | Toggle LXQt's application menu through its existing Super shortcut |

F11 uses an Openbox `XF86LaunchA` binding, corresponding to Linux `KEY_SCALE`. The application-menu handler covers `XF86Explorer` (`KEY_COMPUTER`) and `XF86LaunchB`. These bindings depend on the machine emitting the corresponding keysyms; the owner physically confirmed Fn+F10, Fn+F11 and Fn+F12 work. Standard F keys remain available to applications.

`helix-hotkeys` is a user-session helper, not a privileged service. Dependencies are Python 3, `wpctl`, `nmcli`, `notify-send`, `lxqt-config-brightness`, `lxqt-config`, and `xdotool`. The existing volume handlers provide their own panel feedback. Microphone feedback reports PipeWire's default source state; no new LED synchronization was implemented.

The LXQt configuration was updated through the live daemon's D-Bus API. Duplicate Ctrl+Alt+L, Ctrl+Alt+T and Print command actions were removed. The original client action for `XF86Sleep` was disabled and an enabled command action uses `lxqt-leave --suspend`, because hibernation is not configured. Power and suspend buttons were not executed during setup.

The user configuration snapshots show the resulting bindings. On another machine, merge relevant entries in LXQt's shortcut editor; do not replace all existing shortcuts blindly. Openbox's snapshot preserves Lubuntu defaults and adds only the F11 window-list binding. Reload Openbox after merging the binding.

## Validation and limits

- Kernel input captured the above physical special-key events; full raw keyboard logs are not published.
- The owner confirmed the tested functions work with Fn, including F10 application search, F11 window list and F12 application menu.
- Injecting `XF86AudioMicMute` through X11 changed PipeWire's mute state; the previous state was restored.
- A brightness test overlapped the owner's physical key presses, so it was not treated as an isolated direction/step test.
- Active D-Bus actions, saved settings, script syntax and Openbox XML were checked.
- Wireless toggling, sleep/resume, Fn Lock switching and indicator LEDs need additional validation.

`thinkpad_acpi` was already using its recommended hotkey mask. No driver mask override was needed. See the [Linux kernel's ThinkPad ACPI documentation](https://docs.kernel.org/admin-guide/laptops/thinkpad-acpi.html).
