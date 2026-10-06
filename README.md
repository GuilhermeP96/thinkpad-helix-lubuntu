# ThinkPad Helix on Lubuntu

Device-specific configuration, controls, and troubleshooting notes by **GuilhermeP96** for a first-generation Lenovo ThinkPad Helix (type 3702).

Tested interactively on Lubuntu / Ubuntu 26.04.1 LTS, Linux 7.0.0-38-generic, LXQt 2.3, Openbox, X11, libinput 1.31.1 and Touchégg 2.0.18. This is a record of one machine's setup, not a claim of compatibility with every Helix revision. No kernel or upstream driver source was patched.

## Results and remaining validation

| Area | Observation / change | Validation |
|---|---|---|
| Touchpad and TrackPoint | RMI4/SMBus froze both pointers; PS/2 transport used instead | Owner confirmed both work after switching |
| Scrolling and gestures | Two-finger natural scrolling, clickfinger, Touchégg actions | Owner confirmed scrolling and gestures; later tuning improved responsiveness |
| Light touches | libinput pressure range changed from default down/up 30/25 to 20/15 | Parser and fresh libinput context verified; desktop login and physical validation still pending |
| Graphics | Existing i915 / Mesa stack, no proprietary driver needed | OpenGL acceleration confirmed; native VA-API i965 exposes H.264 and other legacy profiles |
| Bluetooth | Missing BCM20702A1 firmware supplied for USB 0a5c:21e6 | Kernel loaded build 1757; controller powered on; pairing not tested |
| Touch and pen | Atmel touchscreen and Wacom stylus/eraser | Enumerated; screen mapping applied; full pressure/rotation testing pending |
| Cameras / audio | Existing uvcvideo / snd_hda_intel / PipeWire stack | Both cameras and analog audio enumerated; capture/playback not tested |
| Power / memory / SSD | TLP, thermald, LZ4 zram, periodic TRIM | Services active; zram ~3.8 GiB; TRIM executed |

## Files and application

`files/` mirrors the destination paths on the machine. `user-config/` contains per-user settings. Inspect each file and make backups before installing it on another machine. There is deliberately no automatic installer: these choices assume an X11 Helix with the hardware described in [docs/hardware.json](docs/hardware.json).

Keep scripts executable. Rotation requires `python3-gi`, `xrandr`, `xinput`, `notify-send`, `iio-sensor-proxy`, `thinkpad_acpi`, and an X11 session. Recovery uses `pkexec` to run a fixed, root-owned helper and does not grant passwordless administrative access.

### Pointer freeze

The original log repeatedly contained:

```text
rmi4_physical rmi4-00: Failed to read irqs, code=-6
```

Resetting the specific RMI4 SMBus device restored both pointers. The persistent workaround in `files/etc/modprobe.d/helix-touchpad.conf` is:

```text
options psmouse synaptics_intertouch=0
```

After changing this file, regenerate the initramfs with `sudo update-initramfs -u`. Reboot when convenient. This selects PS/2; it does not disable the touchpad, TrackPoint, pen, or touchscreen. It can have different reporting characteristics from SMBus. On this machine the owner verified both pointers after the change. Long-term stability and suspend/resume are still untested.

To revert, remove only this override, regenerate the initramfs and reboot. The original SMBus freeze may return.

### Scrolling, taps and gestures

The Xorg InputClass enables tapping, two-finger scrolling, natural scrolling, clickfinger, and disable-while-typing. `ScrollingPixelDistance=10` replaces the default 15: the same movement produces more scroll. This controls scroll amount; it does not improve the sensor's ability to distinguish fingers.

The local libinput quirk sets `AttrPressureRange=20:15`, scoped to Lenovo type 3702 and the PS/2 touchpad name. A fresh libinput diagnostic context accepted the range. **Log out and back in to apply it to Xorg. Physical validation after login is pending.** Palm and thumb rejection remain enabled. Local quirks are version-dependent; see the [libinput calibration instructions](https://wayland.freedesktop.org/libinput/doc/latest/touchpad-pressure-debugging.html).

Touchégg's default automatic thresholds on this device were start `24.1282`, horizontal finish `339.82`, vertical finish `217.154`. The final daemon override uses `--daemon 1.5 35`, and repeated two-finger zoom shortcuts use `times=5` when pinching in (zoom out) and `times=4` when pinching out (zoom in). The extra zoom-out step was requested subsequently and still needs subjective validation. The owner reported improved responsiveness after tuning. These are subjective device-specific settings, not universal defaults.

| Gesture | Action |
|---|---|
| Two-finger slide | Natural scrolling |
| Two-finger tap | Right click |
| Two-finger pinch | Zoom through Ctrl +/- |
| Three-finger horizontal swipe | Change workspace |
| Three-finger swipe up / down | Maximize/restore / minimize |
| Four-finger horizontal swipe | Change window |
| Four-finger swipe down | Show desktop |
| Three-finger pinch in / out | Alt+Left / Alt+Right browser navigation |

Zoom uses keyboard shortcuts and remains discrete and application-dependent under X11. Touchégg also detects the touchscreen, so compatible configured actions may apply there. Obtain Touchégg from the [maintainer's project](https://github.com/JoseExposito/touchegg). This setup used its release `.deb`, with no added PPA.

### Tablet controls

`helix-controls daemon` starts through XDG autostart in LXQt. It claims the accelerometer and rotates only when `thinkpad_acpi` reports tablet mode. Automatic rotation pauses with another active display. Manual rotation synchronizes screen and the Atmel/Wacom coordinate transformations. A lock file is stored under `~/.config/helix/`.

The following shortcuts were registered through `org.lxqt.global_key_shortcuts.daemon.addCommandAction`, rather than replacing the user's shortcut configuration:

| Shortcut | Command argument |
|---|---|
| Ctrl+Alt+K | `helix-controls keyboard` |
| Ctrl+Alt+R | `helix-controls rotate` |
| Ctrl+Alt+N | `helix-controls normal` |
| Ctrl+Alt+A | `helix-controls auto` |
| Ctrl+Alt+P | `helix-controls recover` (administrative password) |

Lenovo hotkeys now include microphone and brightness feedback, Wi-Fi toggle, configuration center, application search, running-window list and application menu. The owner confirmed the tested special keys respond with Fn. Existing LXQt panel volume actions remain in use. See [the hotkey mapping and validation notes](docs/hotkeys.md); the owner also confirmed Fn+F10/F11/F12 work. Touchpad and rotation keys use `helix-controls` when the hardware emits their XF86 keysyms.

### Night light

The existing Redshift-Qt application was running with an invisible tray icon under the Oxygen theme. A user hicolor fallback using installed Papirus icons restored its advertised status icon, and Show Info opened a visible window. Automatic daytime/nighttime defaults remain unchanged. See [the diagnosis and workaround](docs/night-light.md).

### Bluetooth firmware

The distribution's installed firmware package did not supply the file requested by the Broadcom controller. The matching `brcm/BCM20702A1-0a5c-21e6.hcd` was obtained from [winterheart/broadcom-bt-firmware](https://github.com/winterheart/broadcom-bt-firmware), checked against that project's device checksum and parsed as an HCD stream before installation. Reloading `btusb` changed the kernel-reported firmware build from 0000 to 1757.

No firmware binary is redistributed here. Obtain it from its source and review its Broadcom license. This is a successful use of an existing supported firmware file, not a new driver fix.

### Power, memory and SSD

The included TLP profile keeps `schedutil` and Turbo enabled, favors responsiveness on AC and balanced energy use on battery. USB autosuspend is disabled for peripheral reliability; Wi-Fi power saving is disabled. These choices trade some battery savings for reliability. Existing thermald remains enabled; there is no manual fan control, overclocking, or removal of CPU security mitigations.

Zram uses LZ4, 50% logical RAM capacity, priority 100; storage is allocated on demand. The existing small disk swap was retained. For the SSD, continuous `discard` was replaced with `defaults,noatime` on the root filesystem and `fstrim.timer` enabled. No machine-specific fstab UUIDs are published.

## Limits and reports

The tablet battery reported approximately 38% of design capacity. The base battery reported implausible full energy (653 Wh versus 28 Wh design), and the owner suspects hardware deterioration. No general battery fix is claimed. Hibernate was not configured; the disk swap is too small. Suspend/resume, physical screen rotation, external display mapping and long-term reliability require further testing.

Technical reports are in [reports/](reports/). Verified facts and pending experiments are identified explicitly. Upstream destinations and publication status are tracked in [docs/publication-status.md](docs/publication-status.md).

Full raw journals, serial numbers, MAC addresses, disk UUIDs, credentials and user keyboard events are excluded.
