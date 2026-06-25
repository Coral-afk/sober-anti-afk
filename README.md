# antiafk

Keeps you in-game on Sober (the Roblox Linux client) by pressing Space on a set interval. Runs in the background, restores your focus after each jump, and stays out of the way.

---

## How it works

Every ~6 minutes, the script finds the Sober window, briefly brings it to focus, injects a spacebar press, then hands focus back to whatever you were doing. You won't notice it unless you're watching the terminal.

---

## Known limitation: focus stealing

The focus restore isn't fully reliable — depending on your compositor, the previous window may not regain focus correctly after the spacebar is injected. The workaround is simple:

**Run Sober in fullscreen.** When the script briefly activates the Sober window, your cursor stays over your other window. Fullscreen ensures Sober is actually visible and receives the keypress, while your compositor handles returning focus on its own. This is the recommended way to run the script.

---

## Dependencies

You need two tools installed before this does anything:

| Tool | What it does | Install |
|------|--------------|---------|
| `kdotool` | Finds and focuses windows by name | `sudo dnf install kdotool` |
| `ydotool` + `ydotoold` | Injects keypresses at the kernel level | `sudo dnf install ydotool` |

> **Why ydotool?** Sober runs under XWayland or a native Wayland compositor. `xdotool` won't reach it. `ydotool` operates through `/dev/uinput` so it works regardless of your display server.

### Start the ydotool daemon

`ydotool` needs its daemon running before you launch the script:

```bash
sudo ydotoold &
```

If you want it to start automatically on login, add a systemd user unit:

```bash
systemctl --user enable --now ydotoold
```

---

## Installation

```bash
cd ~/Downloads
chmod +x antiafk.sh
```

That's it.

---

## Usage

```bash
cd ~/Downloads
./antiafk.sh
```

Stop it with `Ctrl+C`.

You should see output like:

```
Anti-AFK running. Press Ctrl+C to stop.
[14:03:47] Jumping in Sober (ID: 12345678)
[14:09:57] Jumping in Sober (ID: 12345678)
```

If Sober isn't open yet when you start the script, it'll just keep waiting and retry every interval until it finds it.

---

## Configuration

Open `antiafk.sh` and change these two lines at the top:

```bash
INTERVAL=350      # seconds between each jump (350 = ~5.8 minutes)
WINDOW_NAME="Sober"  # exact window title to search for
```

---

## Troubleshooting

**"Sober not found" on every interval**
Run `kdotool search --name "Sober"` while Sober is open. If nothing comes back, your window title might differ. Try `kdotool getactivewindow` while focused on Sober, then `kdotool getwindowname <id>` to see the exact name.

**Spacebar not registering**
Check that `ydotoold` is running: `pgrep ydotoold`. If it's not, start it with `sudo ydotoold &`.

Also make sure Sober is running in fullscreen — if it's windowed and focus doesn't land correctly, the keypress won't go through.

**Permission denied on /dev/uinput**
```bash
sudo chmod 660 /dev/uinput
sudo usermod -aG input $USER
```
Log out and back in after adding yourself to the group.

---

## Notes

- Tested on KDE Plasma (Wayland) on Fedora. Should work on any compositor that `kdotool` supports.
- The script doesn't move your mouse or do anything besides the single space keypress.
- If Sober crashes or you close it mid-session, the script keeps running harmlessly and will pick it back up when you relaunch.
- Focus restore after the jump is best-effort. Running Sober fullscreen is the reliable workaround until a cleaner solution exists.
