# MSIKeyboardService
## What Is It

It is a service that controls certain types of keyboard backlights installed on some MSI notebooks (e.g. MSI GT60 2PC). This keyboards usually marked with label like "Keyboard by SteelSeries".

## What It Is Able To Do

* Configure and use some (in future - all) modes from SteelSeries app (Windows), plus some undocumented (i.e. discovered accidentally) modes
* React on lid closed/opened events
* More later

## Dependencies

* python3
* dbus
* python3-hidapi
* python3-dbus
* python3-yaml
* (optional, if you want to install service as described in next section) systemd as system init (e.g. Ubuntu)

## How To Install It

1. Create directory '/usr/local/lib/msikeyboard/'
2. Place files 'msikbapi.py' and 'msikblightd.py' into newly created directory
3. Set this files' owner as root:root and set +x flag on file 'msikblightd.py'
4. Create directory '/etc/msikeyboard/' and place file 'config.yaml' into it
5. Copy file 'org.morozzz.MSIKeyboardService.conf' to directory '/etc/dbus-1/system.d/'
6. Copy file 'msikeyboardd.service' to directory '/lib/systemd/system/'
7. In root terminal (or with sudo) execute following commands:
> systemctl daemon-reload  
  systemctl reload dbus  
  systemctl enable msikeyboardd  
  systemctl start msikeyboardd
8. ???
9. PROFIT

## How To Control It

This service exposes several methods to DBus system bus:
* SetMode(t) -> b - select backlight mode by index (i.e. by index in 'modes' configuration list), returns true if selected successfully
* SetDefaultMode() - select default (i.e. bright white) backlight mode.
* SetOffMode() - select off (i.e. no backlight at all) mode.
* RestoreLastMode() -> b - restore last set by index mode. Useful after SetDefaultMode and SetOffMode invocations.

Also it connects to PropertiesChanged signal to react on lid events. When lid closes, backlight enters Off mode, when opens -- restores last by-index mode

You can write your own utility, that will control this service via DBus, or use standard dbus-send utility for same purpose. For example, to set mode #0 (first mode from configuration), you can invoke dbus-send like:
> dbus-send --system --dest="org.morozzz.MSIKeyboardService" --type=method_call /org/morozzz/MSIKeyboardService org.morozzz.MSIKeyboardService.SetMode uint64:0

I am using Ubuntu, so I bound keys Ctrl-Alt-{0-9} to SetMode({0-9}) invocations, Ctrl-Alt-- (minus, key that comes after 0) to SetOffMode invocation, Ctrl-Alt-= (equals, key after minus and before backspace) to SetDefaultMode invoacation and Ctrl-Alt-Backspace to RestoreLastMode invocation via dbus-send, using System Settings -- Keyboard -- Shortcuts settings.

## Configuration

Configuration is stored in '/etc/msikeyboard/config.yaml' file in [YAML](https://en.wikipedia.org/wiki/YAML) serialization format. Configuration keys:
* default_index (int) - Mode index, that will be set immediately after service start
* handle_lid (bool) - Handle lid events
* modes (list) - List of mode configurations
 * type (str) - Mode type name
 * config (dict) - Mode configuration

Available modes:
* 'Off' (note quotes) - Disabled backlight
 * No (empty) configuration
* Default - Bright white backlight
 * No (empty) configuration
* Normal - Coloured backlight, each zone (left, middle and right) has its own color
 * each zone has dict of three keys - 'r', 'g', 'b', that means red, green and blue color intensity, 0-255
 * zones has names 'left', 'middle' and 'right'
* DualColor - Two colors blending over time:
 * 'color_a' - first color
 * 'color_b' - second color
 * 'fade_times' - blend times, colorwise
 * each key has dict of three keys - 'r', 'g', 'b', that means red, green and blue color
 * times are approximately in seconds (0 - fastest, but not immediate, 255 - slowest)
* More modes incoming...

## TODO:

* Expose DualColorAdvanced mode (like DualColor, but each zone has different colors/times, msikbapi already have appropriate methods)
* Expose 'plain' modes (16-28, looks like plain one-color backlight, also changes behaviour of Normal mode)
* Sniff more modes from SteelSeries app :)
* Explore modes 8,9 and 11-15 (8 looks like DualColor, 9 looks like Waves but this needs thorough research)
* Move service daemon from root to different system user to improve security
* Prepare Python package for PEP conformance
* Prepare .deb package for easier install