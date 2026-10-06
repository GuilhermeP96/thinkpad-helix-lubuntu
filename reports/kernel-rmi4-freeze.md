# ThinkPad Helix 3702: RMI4 SMBus read IRQ errors freeze touchpad and TrackPoint

Target: Ubuntu `linux` package on Launchpad; kernel input/RMI4 maintainers if mainline reproduction is later established.

On a newly installed Lubuntu / Ubuntu 26.04.1 LTS system with `7.0.0-38-generic`, a Lenovo ThinkPad Helix 370242P (BIOS GFET65WW 1.44) repeatedly lost both touchpad and TrackPoint pointer input. Xorg still listed both devices as enabled, send-events modes were enabled, and `thinkpad_acpi/hotkey_tablet_mode` was 0.

The installed Synaptics device is TM2219-002 / PNP LEN0033. The kernel selected RMI4 SMBus automatically (`psmouse.synaptics_intertouch=-1`). TrackPoint was connected through its RMI4 PS/2 pass-through port. The boot log showed repeated:

```text
rmi4_physical rmi4-00: Failed to read irqs, code=-6
```

The owner noticed the freeze after using the Wacom pen. That is an observation, not proof of a causal relationship: the same IRQ errors were already present in earlier logs.

Disabling and enabling Xorg pointer devices did not provide a clear confirmed recovery. Unbinding and rebinding the specific `rmi4_smbus` I2C device restored both pointers, which the owner confirmed. The driver also logged a failure to change enabled interrupts during teardown.

As a workaround, `options psmouse synaptics_intertouch=0` was applied and included in the initramfs. A live psmouse reload plus explicit AUX port rescan re-enumerated the touchpad as `SynPS/2 Synaptics TouchPad`, retaining `TPPS/2 IBM TrackPoint`. The owner confirmed both pointers worked. Two-finger scrolling and gestures were subsequently demonstrated after desktop configuration.

Expected behavior: the normal SMBus transport should continue accepting touchpad and pass-through TrackPoint input without manual resets.

Unknowns: exact deterministic trigger, long-term stability, suspend/resume behavior, and behavior on a vanilla/mainline kernel. No bisection was performed; this is not claimed to be a new regression. The BIOS and batteries are old. No unrelated kernel parameters were changed.

Sanitized hardware details and the workaround are available in this repository. A full boot journal is deliberately not public. A package-specific diagnostic attachment can be collected privately if maintainers request it.
