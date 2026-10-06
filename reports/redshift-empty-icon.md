On a fresh Lubuntu / Ubuntu 26.04.1 LTS installation using LXQt 2.3, X11 and the Oxygen icon theme, Redshift-Qt appeared not to open from the menu or application runner. Both redshift-qt and its redshift child were actually running.

Installed distribution packages: redshift-qt 0.6-4build1, redshift 1.12-4.2ubuntu5, Qt 6 SVG runtime/plugins 6.10.2-2. The installed redshift-qt executable links Qt 6. GeoClue returned a location and the engine reported RandR, Enabled, 6500 K daytime / 4500 K nighttime. No geolocation failure was established.

The registered org.kde.StatusNotifierItem had:

```text
Title: Redshift Qt
Status: Active
IconName: empty
IconPixmap: [(0, 0, [])]
Menu: /MenuBar
```

Expected: when the active icon theme does not provide redshift-status-on/off, a visible fallback tray icon should still be supplied, so the user can access the application's controls.

Workaround verified on this installation: copy the existing Papirus 22x22 panel icons redshift-status-on.svg and redshift-status-off.svg into ~/.local/share/icons/hicolor/22x22/status/, quit Redshift-Qt through its D-Bus menu, and restart. The tray item then advertised IconName=redshift-status-on. Its Show Info action opened an X11-visible Redshift Information window.

Upstream systemtray.cpp uses QIcon::fromTheme with resource fallbacks. I have not established whether the failure originates in Qt 6, missing resources in the Ubuntu package, or theme lookup; no upstream source patch or cross-distribution reproduction is claimed. This is distinct from the older request in #4 to prefer themed icons: the issue here is the fallback being empty.

Diagnosis and workaround: https://github.com/GuilhermeP96/thinkpad-helix-lubuntu/blob/main/docs/night-light.md
