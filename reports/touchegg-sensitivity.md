Additional hardware feedback for the sensitivity discussion in #410:

On a first-generation Lenovo ThinkPad Helix 370242P, Lubuntu / Ubuntu 26.04.1 LTS, LXQt / Openbox / X11, Touchégg 2.0.18 and libinput 1.31.1, the documented daemon-threshold override helped make gestures more responsive.

The touchpad was `SynPS/2 Synaptics TouchPad` (Synaptics TM2219-002 / LEN0033), on PS/2 after a separate RMI4 SMBus freeze. Its reported size was 95.9048 × 61.2857 mm. With automatic thresholds the daemon logged:

```text
start_threshold: 24.1282
finish_threshold_horizontal: 339.82
finish_threshold_vertical: 217.154
```

We first used `--daemon 3 60`, then tuned to `--daemon 1.5 35`. For `SEND_KEYS` two-finger pinch zoom, `repeat=true` and `times=4` replaced `times=2`. The machine owner confirmed improved responsiveness after the last tuning. Native two-finger scroll remained managed by Xorg/libinput, and zoom remained discrete Ctrl +/- steps.

The persistent override was put in a systemd drop-in, preserving the packaged service:

```ini
[Service]
ExecStart=
ExecStart=/usr/bin/touchegg --daemon 1.5 35
```

This is hardware-specific feedback, not a suggested universal threshold or a claim that Touchégg caused the original pointer/scroll problems. A separate libinput pressure experiment still needs physical validation after a new desktop session.

The configuration and validation notes are published at https://github.com/GuilhermeP96/thinkpad-helix-lubuntu .
