### Hyprland Autostart Tiler

This script fixes the common problem where Hyprland apps open in a random order or wrong size when you first log in. It launches your apps, waits for each one to actually show up, then puts it on the right workspace, in the right group or at the right size, so your desktop looks the same every time.

<div align="center">
  <h4>Built with ❤️ by <b>l4n1skyy</b></h4>
</div>

## ✨ What it does
- **Generates your autostart:** one command writes `~/.config/hypr/autostart.lua` from `layout.conf`, launching every app and then the layout pass.
- **Waits for apps:** it only touches a window once it's visible, retrying every 1.5 s, and gives up after 60 s.
- **Moves strays:** an app that opened on the wrong workspace is moved to its own.
- **Tabbed groups:** puts apps (like Discord and Beeper) into one tabbed group, tiling any that opened floating.
- **Exact splits:** a left/right split by percentage (like 30% / 70%), checked on screen and corrected if it lands wrong.
- **Focus and start workspace:** picks which window has focus on each workspace and which workspace you land on when it's done.
- **Runs once, then gets out of the way:** nothing keeps running, so you can resize everything freely afterwards. A notification tells you when it's finished.

## 💬 Requirements
- **Hyprland 0.56 or newer with a Lua config** (`hyprland.lua`). The script talks to Hyprland through its Lua API (`hyprctl dispatch 'hl.dsp…'` and `hyprctl repl`), so it doesn't work with the older `hyprland.conf`.
- **jq** and **awk**.
- **uwsm**: apps are launched through `uwsm-app --`.
- **notify-send** (libnotify) and a notification daemon, for the "done" message.
- **The master layout** on any workspace that uses `RATIO`.
- **kanshi**: the generated autostart also starts kanshi (monitor profiles). If you don't use it, delete that line from the generator in `layout.sh`.

The generated file calls `o.launch(command)`, the launch helper from Omarchy 4's Lua config, which runs the command through `uwsm-app --`. Without Omarchy, define it before the autostart is loaded:

```lua
o = o or {}
function o.launch(command) return "uwsm-app -- " .. command end
```

## 📂 Where to put it

`layout.sh` and `layout.conf` go together in any folder (I use `~/.local/bin/hypr/`):

```
~/.local/bin/hypr/
├── layout.sh             # The script
└── layout.conf           # Your apps and layout rules (must sit next to layout.sh)

~/.config/hypr/
├── hyprland.lua          # Load the autostart from here
└── autostart.lua         # The script creates this for you

~/.cache/
├── layout-engine.log     # Check here if something goes wrong
├── hypr-autostart.log    # What the autostart launched, with timestamps
└── disable_autostart     # Create this file to skip launching apps
```

Then load the autostart from `hyprland.lua`:

```lua
require("hypr.autostart")
```

## 🛠 How to use

### 1. Define your apps
Open `layout.conf` and list your apps.

``` hyprctl clients -j | jq -r '.[] | "\(.class) -> \(.title)"' ```   lists the classes of the apps that are running now

``` APP <alias> <class-name> <launch-command> ```   format

The class is matched as a substring, so `zen` matches `zen` and `zen-browser`.

example
```
APP discord  vesktop                              vesktop
APP beeper   Beeper                               beeper
APP todoist  chrome-app.todoist.com__app-Default  fo-launch-webapp https://app.todoist.com/app
APP calendar chrome-calendar.google.com__-Default fo-launch-webapp https://calendar.google.com
APP obsidian obsidian                             obsidian
APP zen      zen                                  zen-browser
```

Apps start in the order they're listed. Comment out an `APP` line with `#` to stop launching it.

### 2. Set your rules

```
# LAYOUT RULES
# GROUP <workspace> <alias> <alias> ...                 tabbed group of these apps
# SPAWN <workspace> <alias>                             just put the app on this workspace
# RATIO <workspace> <alias> <l|r> <height%> <width%>   left/right split (master layout)
# FOCUS <workspace> <alias>                             which window gets focus there
# START <workspace>                                     where you land when it's done
```

`RATIO` sets one left/right split between two tiled windows, using the master layout's split size. Write one rule for each side. Height is part of the format but isn't used yet.

example
```
START 1

# --- WORKSPACE 10: SOCIAL ---
GROUP 10 beeper discord
FOCUS 10 beeper

# --- WORKSPACE 9: PRODUCTIVITY ---
RATIO 9 todoist     l   100%    30%
RATIO 9 calendar    r   100%    70%

# --- WORKSPACE 8: NOTETAKING ---
SPAWN 8 obsidian
FOCUS 8 obsidian

# --- WORKSPACE 1: THE DASHBOARD ---
SPAWN 1 zen
FOCUS 1 zen
```

### 3. Generate your autostart

Run this whenever you change the `APP` lines:
```
chmod +x layout.sh
./layout.sh --generate
```

It rewrites `~/.config/hypr/autostart.lua`. On the next login, Hyprland launches every app and then runs the layout pass. Changing only the rules doesn't need a new `--generate`, because the script reads `layout.conf` each time it runs.

### 4. Re-apply the layout any time

```
./layout.sh
```

This runs one layout pass on the windows that are open now. Starting it while another pass is still running replaces that pass, so two never fight over the same windows.

To log in without your apps opening (to debug, say), create `~/.cache/disable_autostart`. Delete it to go back to normal.

## ⚖️ License
MIT
