---
title: "Choosing KDE over GNOME in xrdp"
source: "https://claude.ai/chat/fcea37a3-53ae-4756-b755-6850c3c1b407"
author:
published: 2026-11-10
created: 2026-11-10
description: "Claude conversation with 2 messages"
tags:
  - "clippings rhel kde xrdp"
---
**You**

when using krdc to connect to xrdp, how to choose kde instead of gnome to be default?

---

**You**

i'm on rhel 9

# Claude

[](https://claude.ai/upgrade?feature=code&returnTo=%2Fcode)

when using krdc to connect to xrdp, how to choose kde instead of gnome to be default?

The desktop session is chosen on the **server side** by xrdp, not by KRDC. KRDC just displays whatever session xrdp starts, so you need to configure the server (or the user account on it).

### 1. Install Plasma and make sure the session exists

bash

```bash
# Debian/Ubuntu
sudo apt install kde-plasma-desktop

# Fedora
sudo dnf install @kde-desktop-environment
```

### 2. Per-user: set `~/.xsession`

On the server, as the user who will log in over RDP:

bash

```bash
echo "startplasma-x11" > ~/.xsession
chmod +x ~/.xsession
```

(On older Plasma 5 installs, use `startkde` instead. Use `which startplasma-x11 startkde` to see which one you have.)

xrdp’s `startwm.sh` will pick this file up and run it instead of the system default (GNOME).

### 3. System-wide alternative: edit `/etc/xrdp/startwm.sh`

Find the bottom of the file, where it launches the default session (usually `. /etc/X11/Xsession` or similar), and replace it with:

bash

```bash
exec startplasma-x11
```

This applies to every user, but `~/.xsession` is cleaner since it survives package updates.

### 4. Debian/Ubuntu: `.xsession` is ignored sometimes

If `~/.xsession` seems to be ignored, check that `/etc/xrdp/startwm.sh` still contains the stanza that sources `~/.xsession` (the stock one does). Also, `update-alternatives` can set the system default:

bash

```bash
sudo update-alternatives --config x-session-manager
```

and choose the Plasma entry.

### 5. Restart and reconnect

bash

```bash
sudo systemctl restart xrdp xrdp-sesman
```

Log out of any existing xrdp session first, since xrdp reuses a live session instead of starting a new one. Otherwise you’ll keep getting GNOME.

### Common gotchas

- **Black screen / immediate disconnect:** you’re probably already logged in locally as that user (GNOME on the physical console). Log out locally, or use a different user for RDP.
- **Wayland:** xrdp uses X11 sessions, so use `startplasma-x11`, not `startplasma-wayland`.
- **Stale session:** a disconnected session persists. Log out properly from within the desktop (don’t just close KRDC), or kill it with `sudo loginctl terminate-user <user>`.
- **Ubuntu with GNOME:** you may also want to set `unset DBUS_SESSION_BUS_ADDRESS` and `unset XDG_RUNTIME_DIR` at the top of `~/.xsession` if Plasma fails to start due to a DBus conflict:

bash

```bash
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
exec startplasma-x11
```

What distro is the xrdp server running? I can tailor the steps if it’s something other than Debian/Ubuntu or Fedora.

i'm on rhel 9

Confirming which KDE group EPEL provides

On RHEL 9 the steps differ a bit from what I gave earlier: KDE isn’t in the base repos, and the session handling is Red Hat’s own.

### 1. Enable EPEL and install Plasma

bash

```bash
sudo dnf install epel-release
sudo crb enable        # CodeReady Builder, needed by many EPEL packages
sudo dnf groupinstall "KDE Plasma Workspaces"
sudo dnf install xrdp
```

Check the exact group name with `dnf group list | grep -i kde` if that one isn’t found.

### 2. Set the session for the user

On RHEL, xrdp’s `startwm.sh` hands off to `/etc/X11/xinit/Xsession`, which looks for `~/.xsession` and then `~/.Xclients`. Setting both is the safest:

bash

```bash
echo "exec startplasma-x11" > ~/.Xclients
cp ~/.Xclients ~/.xsession
chmod +x ~/.Xclients ~/.xsession
```

Plasma 5.27 from EPEL uses `startplasma-x11` (`startkde` is gone). Confirm with `which startplasma-x11`.

### 3. Optional system-wide default

Instead of per-user files, you can set it in `/etc/sysconfig/desktop`:

bash

```bash
echo "DESKTOP=KDE" | sudo tee /etc/sysconfig/desktop
```

This only works if `/etc/X11/xinit/Xclients` on your system recognizes KDE, so the per-user file is more reliable.

### 4. Start xrdp and open the firewall

bash

```bash
sudo systemctl enable --now xrdp xrdp-sesman
sudo firewall-cmd --permanent --add-port=3389/tcp
sudo firewall-cmd --reload
```

### 5. SELinux

If you can connect but the session dies immediately, check for denials:

bash

```bash
sudo ausearch -m avc -ts recent
```

xrdp is generally fine under the default policy, but this is the first place to look on RHEL if logs show nothing else. The xrdp logs are `/var/log/xrdp-sesman.log` and `~/.xsession-errors`.

### Gotchas

- Log out of any existing xrdp session first, or xrdp will reconnect you to the old GNOME one.
- You can’t be logged in locally as the same user (e.g. on the GNOME console) while connecting via RDP.
- If Plasma fails with a DBus error, put these at the top of `~/.xsession` and `~/.Xclients`:

bash

```bash
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
exec startplasma-x11
```

If it still lands in GNOME or shows a black screen, send me the last lines of `/var/log/xrdp-sesman.log` and `~/.xsession-errors` and I can help narrow it down.

  

[Claude is AI and can make mistakes. Please double-check responses.](https://support.anthropic.com/en/articles/8525154-claude-is-providing-incorrect-or-misleading-responses-what-s-going-on)

Claude finished the response