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
