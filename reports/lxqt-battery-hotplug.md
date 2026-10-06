Environment: Lubuntu / Ubuntu 26.04.1 LTS, lxqt-powermanagement 2.3.0-0ubuntu1, LXQt 2.3, Openbox / X11, kernel 7.0.0-38-generic; first-generation Lenovo ThinkPad Helix 3702 with tablet BAT0 and base BAT1.

The owner detached the keyboard/base and reattached it during the same desktop session. They reported that the second battery did not return to the desktop indicator. Kernel logs subsequently showed BAT1 docking/registration, /sys/class/power_supply/BAT1/present was 1, and upower -e / upower -i showed both battery devices present.

Expected: the battery watcher and its information dialog should update when a primary battery is removed or added, without restarting the desktop power manager.

A local restart of lxqt-powermanagement, with both batteries present, restored two registered battery StatusNotifierItems. One power-manager process was running. This validates a UI-refresh workaround, not a kernel battery fix. The local helper now debounces battery-topology changes and restarts the manager; the full new detach/reattach cycle is still awaiting physical confirmation.

Source inspection of the 2.3.0 tag found that BatteryWatcher obtains Solid::Device::listFromType once in its constructor, stores mBatteries, constructs BatteryInfoDialog once, and connects existing batteries' energy/state signals. No device-added/removed subscription was found there. This is a likely relevant lifecycle issue; Solid backend behavior was not independently instrumented.

The base battery also has a separate, implausible energy_full report (653.05 Wh versus 28.05 Wh design), which can distort aggregate percentages. The hotplug UI complaint is recorded separately from this gauge problem. UPower was installed and working; this is distinct from #223, which was resolved by identifying removed UPower packages.

Local diagnosis / workaround: https://github.com/GuilhermeP96/thinkpad-helix-lubuntu/blob/main/docs/dock-tablet.md
Source: https://github.com/lxqt/lxqt-powermanagement/blob/2.3.0/src/batterywatcher.cpp
