# Linux RDP and Xhost Setup Commands

## 1. Enable GNOME Remote Desktop (RDP)

Run these commands once:

```bash
grdctl rdp enable
grdctl rdp disable-view-only
```

Check the RDP configuration:

```bash
grdctl rdp status
```

To disable RDP later:

```bash
grdctl rdp disable
```

These `grdctl` settings are persistent and normally do not need to be run after every reboot.

---

## 2. Test `xhost +local:`

Run:

```bash
xhost +local:
```

Check the current Xhost settings:

```bash
xhost
```

`xhost +local:` itself is normally not persistent across logout/reboot.

---

## 3. Make `xhost +local:` Run Automatically

Create the systemd user-service directory:

```bash
mkdir -p ~/.config/systemd/user
```

Create the service file:

```bash
nano ~/.config/systemd/user/xhost-local.service
```

Paste this into the file:

```ini
[Unit]
Description=Allow local X connections
After=graphical-session.target

[Service]
Type=oneshot
ExecStart=/usr/bin/xhost +local:
RemainAfterExit=yes

[Install]
WantedBy=graphical-session.target
```

### Save the nano file

Press:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 4. Enable and Start the Service

Run:

```bash
systemctl --user daemon-reload
systemctl --user enable xhost-local.service
systemctl --user start xhost-local.service
```

Check whether it is enabled:

```bash
systemctl --user is-enabled xhost-local.service
```

Expected output:

```text
enabled
```

Check the service status:

```bash
systemctl --user status xhost-local.service
```

Check Xhost:

```bash
xhost
```

You should see:

```text
LOCAL:
```

---

## 5. Summary

### Run once

```bash
grdctl rdp enable
grdctl rdp disable-view-only

mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/xhost-local.service

systemctl --user daemon-reload
systemctl --user enable xhost-local.service
systemctl --user start xhost-local.service
```

### After setup

- RDP settings remain enabled until you disable them.
- The systemd service automatically runs `xhost +local:` for your user graphical session.
- You normally do not need to manually run these commands after every reboot/login.

> Note: `xhost` controls X11/XWayland access. On GNOME Wayland, native Wayland applications are not controlled by `xhost`.

---

## 6. Verify the GNOME RDP Service

After creating and enabling `gnome-rdp.service`, verify it:

```bash
systemctl --user status gnome-rdp.service
```

A successful service should show:

```text
Loaded: loaded (.../gnome-rdp.service; enabled; ...)
Active: active (exited)
```

The two `grdctl` commands should also show:

```text
grdctl rdp enable
→ status=0/SUCCESS

grdctl rdp disable-view-only
→ status=0/SUCCESS
```

### Check whether the service is enabled

```bash
systemctl --user is-enabled gnome-rdp.service
```

Expected:

```text
enabled
```

### Important: `grdctl rdp status` may not exist

Some versions of `grdctl` do **not** provide a `status` subcommand.

If you run:

```bash
grdctl rdp status
```

and get:

```text
Unknown subcommand 'status'
```

that is not necessarily an error with your RDP setup. It means your installed version does not support that command.

To see the commands supported by your installed version, use:

```bash
grdctl
```

or:

```bash
grdctl rdp
```

### Test persistence

Reboot the system:

```bash
reboot
```

After logging back into GNOME, check:

```bash
systemctl --user status gnome-rdp.service
```

and:

```bash
systemctl --user is-enabled gnome-rdp.service
```

If the service is still `enabled` and `active (exited)`, the `grdctl` commands are being automatically executed at graphical-session startup.

---

## 7. Python PyAutoGUI Super/Windows Key

Start Python:

```bash
python
```

Then:

```python
import pyautogui as pg
pg.press("win")
quit()
```

`pg.press("win")` simulates pressing the Windows/Super key.

### Common syntax mistake

This is incorrect:

```python
pg.press9"win
```

It produces a `SyntaxError`.

The correct syntax is:

```python
pg.press("win")
```

