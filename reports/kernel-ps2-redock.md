# ThinkPad Helix PS/2 keyboard and touchpad fail after base detach/reattach

Target: Ubuntu linux package on Launchpad. This is a follow-up symptom separate from the earlier RMI4 SMBus freeze.

Environment: first-generation Lenovo ThinkPad Helix 370242P, BIOS GFET65WW 1.44, Ubuntu / Lubuntu 26.04.1 LTS, kernel 7.0.0-38-generic, LXQt / Openbox / X11. psmouse synaptics_intertouch=0 selected to avoid the independently recorded SMBus IRQ read errors.

The owner repeatedly detached and reattached the physical base. Their configured virtual keyboard successfully appeared/disappeared and battery/charging changes were recognized, but the physical keyboard and touchpad did not resume. They rebooted to recover. TrackPoint behavior in those failed cycles was not separately confirmed.

Previous-boot kernel logs showed ThinkPad docking/undocking and BAT1 registration. psmouse logged Synaptics min/max coordinate queries on reattachment. No new AT keyboard input-device registration appeared after the boot-time one. There was no new RMI4 transport in use, and the exact failure point between EC/i8042/atkbd/psmouse remains unknown.

The local recovery helper now requests drvctl=rescan on the identified i8042 KBD and AUX ports, after a delayed native ACPI dock event, with BAT1 add as a fallback. Its initial manual run, while the base was already attached after reboot, successfully recreated AT Translated Set 2 keyboard, SynPS/2 Synaptics TouchPad and TPPS/2 IBM TrackPoint. Udev trigger configuration was checked. The owner initially confirmed all three returned after one real cycle, but subsequently reported failures starting with the second cycle and occasional recovery on a later reattachment. Therefore port rescanning is not a reliable fix even though the service recreated device nodes.

A stronger local i8042 platform-driver unbind/rebind experiment is now active. It restores the binding on cancellation and detects new serio port numbers dynamically. One manual run completed and registered the three devices; repeated physical tests still failed after the first cycle and later recovered on another reattachment. No generic i8042 boot parameters, kernel patch, regression claim or bisection was introduced. Details: ../docs/dock-tablet.md. A native Launchpad report and package diagnostics are still pending authenticated submission.

Two subsequent real dock events completed the controller-rebind service. A temporary counter observed real evdev events from the physical keyboard, touchpad and TrackPoint during the test window (event types counted only, no key content recorded). The owner reported failure from the second cycle onward, followed by normal operation in the latest cycle. Both recovery strategies remain intermittent, and the evdev-to-desktop path must be checked during a failure.
