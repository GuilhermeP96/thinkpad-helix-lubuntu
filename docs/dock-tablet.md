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
- The full physical detach/show/reattach/hide cycle after the changes is awaiting the owner's confirmation. A subsequent login/reboot with the new helper is also pending.
- BAT1's reported capacity is still invalid. Restoring its indicator does not repair its gauge or prove usable battery capacity.

References: [Onboard D-Bus API](https://github.com/onboard-osk/onboard/blob/main/DBUS.md), [LXQt 2.3.0 battery watcher](https://github.com/lxqt/lxqt-powermanagement/blob/2.3.0/src/batterywatcher.cpp).
