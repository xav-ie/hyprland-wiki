# Table of Contents

{{< toc format=html >}}

# My apps are pixelated

This just means they are running through XWayland, which physically cannot scale by fractional amounts.

To force them to run in wayland-native mode, see [the Master Tutorial](../../Getting-Started/Master-Tutorial/#force-apps-to-use-wayland).

If they can't, see [the XWayland page](../../Configuring/XWayland).

# Nothing renders / screen is empty / crash on opening first app

Possible causes:

> Your themes are not set up properly, making apps crash.

Use something like `qt6ct` (Qt) and `nwg-look` (GTK) (\*for GTK you can also
set up themes with envvars) to set up your themes.

> Your PC is very, _very_ old.

In that case, see the
[Installation Page](../../Getting-Started/Installation) and
try compiling with LEGACY_RENDERER

_For more info about bugs and crashes, see this_
_[wiki page](../../Crashes-and-Bugs)_

# Me cursor no render?

Are you on NVIDIA? If so, then you have been a naughty boy and haven't listened
to my tips on other pages. Use the `WLR_NO_HARDWARE_CURSORS=1` environment
variable.

# My external monitor is blank / doesn't render / receives no signal (laptop)

For Nvidia graphics - This issue appears to be resolved when using Nvidia Drivers 525.60.11 or later, but it may persist with older drivers.

Outside those, there is a way to fix it that _might_ work for you though:

**Option 1:** Use _only_ the external monitor

By using `WLR_DRM_DEVICES=/dev/dri/card1` (or `card0`) you can force Hyprland to use only your dGPU,
meaning your laptop's screen will be gone but your external one will work.

**Option 2:** Use all outputs, at the cost of battery life.

By switching your laptop to only use the dGPU in the BIOS, you _might_ be able to get everything to work,
at the cost of high battery usage.

_Please note these are highly model-specific and might or might not work. If they don't, you're unfortunately out of luck._

You might try a USB-C to HDMI adapter though, maybe that could route the external monitor through the iGPU.

# How do I screenshot?

Install `grim` and `slurp`

Use a keybind (or execute) `grim -g "$(slurp)"`, select a region. A screenshot
will pop into your `~/Pictures/` (You can configure grim and slurp, see their
GitHub pages).

If you want those screenshots to go directly to your clipboard, consider using `wl-copy`, from [`wl-clipboard`](https://github.com/bugaevc/wl-clipboard). Here's an example binding:
`bind = , Print, exec, grim -g "$(slurp -d)" - | wl-copy`
For a more complete utility, try our own screenshotting utility:
[Grimblast](https://github.com/hyprwm/contrib).

For recording videos, [wf-recorder](https://github.com/ammen99/wf-recorder), [wl-screenrec](https://github.com/russelltg/wl-screenrec) or [OBS Studio](https://obsproject.com/) could be used.

# Screenshare / OBS no worky

Check [Screensharing](../Useful-Utilities/Screen-Sharing).

Also install `qt6-wayland` if you plan to use obs.

# How do I change my wallpaper?

See [Wallpapers](../Useful-Utilities/Wallpapers).

# How heavy is this?

Not that much heavier than Xorg. If you want maximum performance, consider
turning off the blur and animations.

# My monitor no worky

Try changing the mode in your config. If your preferred one doesn't work, try a
lower one. A good way to list all modes is to get `wlr-randr` and do a
`wlr-randr --dryrun`

# My monitor has flickering brightness when I turn on VRR

Change the VRR option to `2` (fullscreen), so that it is only used in games.
This happens because the brightness on some monitors can depends on the refresh
rate, and rapidly changing refresh rates (for example, when the screen momentarily
updates after pressing a key) will cause rapid changes in brightness.

# How do I update?

Open a terminal where you cloned the repo.
```bash
git pull 
make all && sudo make install
```

If you are using the AUR (hyprland-git) package, you
will need to cleanbuild to update the package. Paru
has been problematic with updating before, use Yay.

# How do I screen lock?

Use a wayland-compatible locking utility using WLR protocols, e.g. `swaylock`.

# How do I change me mouse cursor?

1. Set the GTK cursor using [nwg-look](https://github.com/nwg-piotr/nwg-look).
2. Add `exec-once=hyprctl setcursor [THEME] [SIZE]` to your config and restart Hyprland.

If using flatpak, run `flatpak override --env=~/.themes:ro --env=~/.icons:ro --user` and put your themes in both `/usr/share/themes` and `~/.themes`, and put your icons and cursors in both `/usr/share/icons` and `~/.icons`.

For Qt applications, Hyprland exports XCURSOR_SIZE as 24, which is the default.
You can overwrite this by exporting XCURSOR_SIZE to a different value with `env`.

You can also try running `gsettings set org.gnome.desktop.interface cursor-theme 'theme-name'` or adding it after `exec-once=` in your config.

If you do not want to install a GTK settings editor, change the config files according to the 
[XDG specification (Arch Wiki link)](https://wiki.archlinux.org/title/Cursor_themes#Configuration). 
Make sure to also edit `~/.config/gtk-4.0/settings.ini` and `~/.gtkrc-2.0` if _not_ using a tool 
(like `nwg-look`).

# GTK Settings no work / whatever

[https://github.com/swaywm/sway/wiki/GTK-3-settings-on-Wayland](https://github.com/swaywm/sway/wiki/GTK-3-settings-on-Wayland)

# My \[program name\] is freezing

Make sure you have a notification daemon running, for example `dunst`. Autostart
it with the `exec-once` keyword.

# Waybar workspaces no worky???

Waybar has a set of caveats or settings that you need to be aware of. See
[Status bars](../Useful-Utilities/Status-Bars) for solutions.

# How do I autostart my favorite apps?

Using the window rules to assign apps to workspace you can open a bunch of
applications on various workspaces. The following method will start these apps
silently (i.e.  without the flickering from workspace to workspace).

Put the following in your `hyprland.conf`: (example)

```ini
exec-once=[workspace 1 silent] kitty
exec-once=[workspace 1 silent] subl
exec-once=[workspace 3 silent] mailspring
exec-once=[workspace 4 silent] firefox
```

# How do I move my favorite workspaces to a new monitor when I plug it in?

if you want workspaces to automatically go to a monitor upon connection, use the
following:

In hyprland.conf:

```ini
exec-once=handle_monitor_connect.sh
```

where `handle_monitor_connect.sh` is: (example)

```sh
#!/bin/sh

handle() {
  case $1 in monitoradded*)
    hyprctl dispatch moveworkspacetomonitor "1 1"
    hyprctl dispatch moveworkspacetomonitor "2 1"
    hyprctl dispatch moveworkspacetomonitor "4 1"
    hyprctl dispatch moveworkspacetomonitor "5 1"
  esac
}

socat - "UNIX-CONNECT:/tmp/hypr/${HYPRLAND_INSTANCE_SIGNATURE}/.socket2.sock" | while read -r line; do handle "$line"; done
```

if you want workspaces 1 2 4 5 to go to monitor 1 when connecting it.

Please note this requires `socat` to be installed.

# My tablet no worky??

Use [Open Tablet Driver](https://github.com/OpenTabletDriver/OpenTabletDriver)
to configure your tablet. In the future it will be supported in the config.
Until then, OTD is the way to go.

# Some of my apps take a really long time to open...?

_~/.config/hypr/hyprland.conf_

```ini
exec-once=dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP
```

Make sure that your portals launch _after_ this gets executed. For some people,
they might launch before that has happened.

In such cases, a script like this:

```bash
#!/bin/bash
sleep 4
killall -e xdg-desktop-portal-wlr
killall xdg-desktop-portal
/usr/lib/xdg-desktop-portal-wlr &
sleep 4
/usr/lib/xdg-desktop-portal &
```

launched with `exec-once` should fix all issues. Adjust the sleep durations to taste.

# How do I export envvars for Hyprland?

See [Environment Variables](../Configuring/Environment-variables)

The `env` keyword is used for this purpose. For example:
```ini
env = XDG_CURRENT_DESKTOP,Hyprland
```

# How to disable middle-click paste?

You can simply intercept the middle-click action all together, via hyprland binds for example. The drawbacks to this solution are that 1. it disables the rest of the functionality of the middle-click action, such as auto scroll, closing browser tabs, etc., and 2. many applications (such as kitty) manually intercept the middle-click events and bind them to paste from the primary buffer themselves, bypassing the solution altogether. For this solution, add this bind to your config:

`bind = , mouse:274, exec, ;`. Note that the exact bindcode may vary, so you may want to check it with `wev` first.

<details>
  <summary> Alternative method using wl-paste (warning: major power consumption)</summary>
	
  The middle-click paste action pastes from a separate buffer (primary buffer) than what the regular clipboard uses (clipboard buffer). Since the primary buffer is unrelated to the clipboard buffer, it's easy to simply keep the primary buffer empty, allowing the middle-click action to retain the rest of its functionality without having anything to paste. Run the following command (in your config with `exec-once`, for example) to achieve this:

  `wl-paste -p --watch wl-copy -p ''` (`wl-paste -p --watch` watches for changes to the primary buffer, `wl-copy -p ''` clears the primary buffer)

  **As you can see, however, this creates an endless loop (found copied text -> copy -> found copied text...). Therefore this method is not recommended.**

</details>

# How do I make Hyprland draw as little power as possible on my laptop?

**_Useful Optimizations_**:

* `decoration:blur = false` and `decoration:drop_shadow = false` to disable
   fancy but battery hungry effects.

* `misc:vfr = true`, since it'll lower the amount of sent frames when nothing is happening on-screen.

# How to fix games with window dancing?

Read [this trick](../Configuring/Uncommon-tips--tricks/#window-dancing).

# My apps take a long time to start / can't screenshare

See [The XDPH Page](../Useful-Utilities/Hyprland-desktop-portal).

You most likely have multiple portal impls / an impl is failing to launch.

# My screenshot utilities won't work with multiple screens
Some programs like flameshot (currently) has limited wayland support so on most Wayland compositors, you will have to do few tweaks.
For Hyprland, you can add these window rules to your config to make said programs work with both of your screens.
```windowrulev2=float,title:^(flameshot)
windowrulev2=move 0 0,title:^(flameshot)
windowrulev2=nofullscreenrequest,title:^(flameshot)
```
# I cannot bind SUPER as my mod key on my laptop

Many laptops have a built-in function to toggle `SUPER` between single key press mode and hold mode. This is usually indicated by a padlock on the `SUPER` key.

First, install and run `wev`, then press `SUPER`. If you see a key press event followed by an instant key release event, then it's likely your `SUPER` key is set to single press mode.

On most laptops this can be fixed by pressing `FN+SUPER` and verified in `wev`. You should be able to hold `SUPER` and not see an instant release event. In case `FN+SUPER` doesn't work, consult your laptop's manual.

# My VM doesn't receive keybinds I have set in Hyprland

This is expected, as Hyprland takes precedence.

A simple fix is to create an empty "passthrough" submap:

```ini
bind = MOD,KEY,submap,passthru
submap = passthru
bind = SUPER,Escape,submap,reset
submap = reset
```

set `MOD` and `KEY` to desired values.

By pressing the selected combo you will enter a mode where hyprland ignores your keybinds and passes them on to the vm.

Then, pressing `SUPER + Escape` will leave that mode.

# Some of my drop-down/pop-up windows in apps disappear

In some apps like Steam or VSCode, the drop-down windows may disappear if you hover over them. This can be fixed with window rules.

First, find the title and class of the pop-up window with `hyprctl clients`. You can try something like `sleep 3 && hyprctl clients` so you have time to open the pop-up. It should look something like this:

```bash
Window 55d794495400 -> :
	...
	class: [CLASS here]
	title: [TITLE here]
	...
```

If the pop-up disappears as you hover over it, you can add to your config:

```ini
windowrulev2 = stayfocused, title:^(TITLE)$, class:^(CLASS)$
```

This has a downside of not being able to click on anything in the main UI until you've interacted with the pop-up.

If the pop-up disappears immediately, you can use:

```ini
windowrulev2 = minsize 1 1, title:^(TITLE)$, class:^(CLASS)$
```
