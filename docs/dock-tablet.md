# Detaching the Helix base: keyboard and battery indicator

The owner reported that removing the base did not open a virtual keyboard and reconnecting it did not restore the second battery in the desktop indicator. The previous local helper only handled rotation; it did not automatically open Onboard. That omission was corrected in this repository, not reported as an upstream Onboard defect.

## Tablet / laptop transitions

The helper now checks both the ThinkPad firmware tablet switch and the Helix battery-bay docking state. It selects the battery_bay whose firmware node is PNP0C0A:01, ignoring the separate ATA bay. Undocked means tablet mode even if the firmware switch remains zero; the firmware tablet signal still supports tablet orientation while attached.

On a transition to tablet mode, the helper starts Onboard when needed and requests its D-Bus Show method, retrying while the service starts. When returning to laptop mode, it requests Hide. It does not repeatedly force the keyboard open after the owner dismisses it. Ctrl+Alt+K retains the manual toggle. The keyboard transition is independent of the rotation lock and accelerometer availability.

The monitor starts immediately and polls once per second. Rotation retains the previous external-monitor and manual-lock rules. XDG autostart supplies the current graphical-session environment at login.

## Battery display refresh

Kernel docking logs and UPower showed BAT1 present after reconnection, despite the owner reporting the missing desktop indicator. LXQt powermanagement 2.3.0 builds its battery list at startup. The local workaround debounces battery-topology changes for four seconds, then refreshes only lxqt-powermanagement. A separate user systemd service keeps that process independent of a temporary command or the tablet monitor; it is started on demand rather than enabled at login alongside the original LXQt autostart.

Required user unit: user-config/systemd/user/helix-power-indicator.service. Install it in ~/.config/systemd/user/ and run systemctl --user daemon-reload. The helper imports DISPLAY and XAUTHORITY from its actual session before starting the service. Power-manager restart resets its runtime idle/pause state, while saved power settings and TLP thresholds are preserved.

The manual recovery command is helix-controls refresh-batteries; the running monitor handles the request. No UPower or kernel-driver reload, forced battery discharge, or firmware change is used.

The battery hotplug symptom was [reported to LXQt as #495](https://github.com/lxqt/lxqt-powermanagement/issues/495).

## Validation

- Kernel and UPower confirmed BAT0 and BAT1 present; the TLP profile changed from battery back to AC when the owner reconnected the charger.
- The monitor started docked and Onboard's Visible property became false. Explicit D-Bus Show and Hide were also checked against Visible=true/false.
- Refreshing the power manager restored two registered LXQt battery tray items, with one live manager process.
- A temporary filesystem check passed: docked / detached / reattached transitions, ignoring the ATA bay, and retaining the firmware tablet switch.
- The owner later confirmed that repeated detach/reattach cycles show/hide the virtual keyboard and update charging behavior. However, the physical keyboard and touchpad did not return, requiring a reboot. A separate PS/2 recovery is being validated below.
- BAT1's reported capacity is still invalid. Restoring its indicator does not repair its gauge or prove usable battery capacity.

References: [Onboard D-Bus API](https://github.com/onboard-osk/onboard/blob/main/DBUS.md), [LXQt 2.3.0 battery watcher](https://github.com/lxqt/lxqt-powermanagement/blob/2.3.0/src/batterywatcher.cpp).

## Physical keyboard / touchpad recovery after redocking

After several physical cycles, the owner reported that tablet/virtual-keyboard transitions and charging behavior worked but the physical keyboard and touchpad did not return until reboot. Previous-boot logs showed docking and BAT1 registration, plus partial psmouse reconnect queries; no new AT keyboard device appeared on redocking.

The installed local workaround triggers helix-dock-input.service on the native ACPI battery-bay EVENT=dock notification; undocking cancels a pending recovery. A BAT1 add rule is a secondary trigger. The BAT1 power-supply node was reused during a physical cycle, so relying only on add did not reliably trigger the service. It waits two seconds, checks the ThinkPad Helix model and the specific base battery bay, then writes rescan to the identified i8042 keyboard and AUX serio ports. This requests full input-device re-enumeration, retaining the existing PS/2 transport choice. Cold-boot events in the first 30 seconds are skipped. The helper waits for keyboard, touchpad and TrackPoint input-device registration, with a timeout.

The first manual service run succeeded and kernel logs showed a new AT keyboard, SynPS/2 touchpad and TrackPoint. The udev dry-run confirmed SYSTEMD_WANTS=helix-dock-input.service. Device registration is evidence of re-enumeration, not proof that physical input works after a real redock. The owner initially confirmed that all three worked after one cycle, but then reported failures from the second cycle onward and occasional spontaneous recovery on a later reattachment. Thus port rescanning was not a reliable fix, despite successful device registration.

The session monitor delays hiding the virtual keyboard briefly on return to the base and waits while the recovery service is activating. If that service fails, it leaves the virtual keyboard available and reports the failure. Existing root-owned helper and system service require no new passwordless polkit or sudo grants.

Installation on the matching Helix requires the included root helper, systemd service and udev rule, followed by systemctl daemon-reload and udevadm control --reload-rules. This is a local workaround, not an upstream kernel patch. To remove it, remove the rule and service, reload udev/systemd, and restore the previous session helper to remove its recovery-service dependency.

Validation also passed udev rule verification, systemd unit verification and Python syntax checks. Both batteries retained 60/80 charge limits after the physical cycles.

### Current controller-rebind experiment

Because port rescanning remained intermittent, the service now invokes the helper with --controller. It unbinds and rebinds the i8042 platform driver, rebuilding its keyboard/AUX ports and IRQ setup. Touchscreen and pen are USB devices and are outside this controller. A finally block restores binding on normal cancellation; ExecStopPost also restores it if the helper exits while the controller is unbound.

Serio indices changed from serio0/1 to higher values during the first controller rebind. The helper now discovers both direct i8042 ports by their description instead of fixed indices, including after rebind. The manual pointer-reset helper was updated for the same dynamic naming. A second manual run with this correction completed and registered all three physical input devices.

The stronger strategy has **not yet been physically validated across repeated detach/reattach cycles**. Successful driver binding and device registration are not a claim of restored real input. No generic i8042 boot flags, upstream source patch, or long-term fix is claimed. The virtual keyboard was explicitly left available during testing.
