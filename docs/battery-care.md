# Battery charge thresholds

After reboot, the owner reported that the system was operating normally. TLP 1.8.0 was enabled, active in AC mode, and its thinkpad plugin / native thinkpad_acpi driver exposed start/stop charge thresholds for BAT0 and BAT1.

At the owner's request to preserve the old batteries until replacement, the existing TLP configuration was extended:

```ini
START_CHARGE_THRESH_BAT0=60
STOP_CHARGE_THRESH_BAT0=80
START_CHARGE_THRESH_BAT1=60
STOP_CHARGE_THRESH_BAT1=80
RESTORE_THRESHOLDS_ON_BAT=1
```

`tlp start` applied the configuration. `tlp-stat -b` and the kernel sysfs values reported 60/80 for both batteries, with charge_behaviour still auto. TLP remains enabled for startup. These settings prevent recharging above the start threshold and stop an active charge at the stop threshold; they do not deliberately drain the batteries while AC is connected. Normal operation on battery power remains available, including below 60%.

BAT0, the tablet battery, reported about 67% charge and Not charging, consistent with the selected interval. Its reported full capacity was 16.07 Wh versus a 42.01 Wh design capacity (~38%). BAT1, the base battery, still reported implausible full capacity (653.05 Wh versus 28.05 Wh design). Acceptance of its thresholds does not prove correct charging behavior when its charge gauge is unreliable. No forced discharge or recalibration was run.

The configuration is persistent, but a further reboot with the new thresholds and a real charging cycle have not yet been observed. No recovery of lost capacity is claimed.

## Temporary full charge for travel

These commands temporarily restore the vendor's full-charge limits; they do not wait for charging to finish:

```sh
sudo tlp fullcharge BAT0
sudo tlp fullcharge BAT1
```

With RESTORE_THRESHOLDS_ON_BAT=1, configured limits are restored when AC is unplugged. They can also be restored explicitly:

```sh
sudo tlp setcharge 60 80 BAT0
sudo tlp setcharge 60 80 BAT1
```

To return permanently to full-charge defaults, configure start 0 / stop 100 for both batteries and run `sudo tlp start`. Back up the configuration first.

See the [TLP battery care settings](https://linrunner.de/tlp/settings/battery.html) and [command documentation](https://linrunner.de/tlp/usage/tlp.html). Behavior was verified against the installed TLP 1.8.0 configuration and exposed kernel values; later TLP versions may have different defaults.

## Low tablet charge during detachment

During repeated docking tests BAT0 reported 7% and was charging on AC, while BAT1 reported 100% with its implausible capacity. Once the base is physically detached, BAT1 cannot power the tablet. Its displayed charge therefore cannot prevent a genuine low-BAT0 condition. LXQt was configured to hibernate below 10%, with a 60-second warning.

The current system has no configured resume device (sysfs resume 0:0); disk swap is only 512 MiB, with the remaining swap in zram. Hibernation is not configured for reliable resume. The user setting powerLowAction was changed from 1 (hibernate) to 4 (suspend), retaining powerLowLevel=10 and powerLowWarning=60. The power indicator service was restarted successfully. Suspension still consumes some battery; this change does not extend autonomy or protect unsaved work indefinitely. Charge BAT0 before further undocked tests. This action has not been physically triggered as a validation test.
