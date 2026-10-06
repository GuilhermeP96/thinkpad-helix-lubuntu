# Night light: Redshift-Qt and an invisible tray icon

Lubuntu already supplied `redshift` 1.12 and `redshift-qt` 0.6. The application appeared not to open from the menu or application search. Inspection found both processes running, GeoClue returning a location, and Redshift using RandR. It was daytime, with the normal 6500 K setting; the default night setting is 4500 K.

The desktop's icon theme was Oxygen. Redshift's StatusNotifierItem was registered with title `Redshift Qt`, but its `IconName` was empty and its pixmap was 0×0. This explained the invisible tray control on this installation. No application crash or geolocation failure was established.

## Applied workaround

Copy the two existing Papirus icons into the user's fallback hicolor theme:

```sh
mkdir -p ~/.local/share/icons/hicolor/22x22/status
cp /usr/share/icons/Papirus/22x22/panel/redshift-status-on.svg    ~/.local/share/icons/hicolor/22x22/status/
cp /usr/share/icons/Papirus/22x22/panel/redshift-status-off.svg    ~/.local/share/icons/hicolor/22x22/status/
```

Quit Redshift through its tray menu and relaunch `redshift-qt`. If the icon is initially invisible, its existing D-Bus menu can be used to quit cleanly; do not start competing Redshift processes. The icons are supplied by the installed Papirus package and are not redistributed in this repository.

After restarting, the tray item advertised `IconName=redshift-status-on`. Its Show Info menu action opened a `Redshift Information` window visible in the X11 window tree. The engine reported Enabled and RandR adjustment, with 6500 K day / 4500 K night. Nighttime appearance and user visibility confirmation remain pending.

No color-temperature preferences, location coordinates or login autostart were saved during this repair. Redshift-Qt is running in the current graphical session. Starting it from the menu normally uses the tray rather than opening a settings window. Right-click the tray icon for information, temporary suspension or exit.

To revert this icon workaround, remove only the two copied files and restart Redshift. See the [Lubuntu Redshift manual](https://manual.lubuntu.me/master/2/2.4/2.4.8/Redshift.html) and [upstream tray implementation](https://github.com/Chemrat/redshift-qt/blob/master/systemtray.cpp).
