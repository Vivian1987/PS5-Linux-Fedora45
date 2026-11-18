# Bidirectional GNOME <-> Steam Big Picture Session Switching (Fedora 45 / PS5)

This guide details how to set up a stable, automatic, and secure session environment allowing you to switch instantly with a single click (or directly from the Steam menu) between the **GNOME** desktop and the **Steam Big Picture** interface on a PS5 console running Fedora 45.

---

## 🛠️ Step 1: User Account and AccountsService Verification

Before configuring automatic startup, ensure your user account (e.g., `ps5`) is active and its AccountsService configuration defaults to GNOME.

```bash
# Check the current user
whoami

# Ensure the AccountsService file exists and set GNOME as default
sudo sed -i 's/^Session=.*/Session=gnome/' /var/lib/AccountsService/users/ps5 && sudo passwd -u ps5 2>/dev/null || true
```

---

## 🚀 Step 2: Configure GDM Auto-Login

To allow the console to log in automatically without prompting for a password at boot, configure GDM:

```bash
sudo tee /etc/gdm/custom.conf > /dev/null << 'EOF'
[daemon]
AutomaticLoginEnable=True
AutomaticLogin=ps5

[security]

[xdmcp]

[chooser]

[debug]
EOF
```

---

## 🔍 Step 3: GDM Display Scaling (Optional for 4K / PS5)

If you are using a 4K display, adjusting the text scaling factor ensures the login screen remains readable:

```bash
sudo mkdir -p /etc/dconf/profile /etc/dconf/db/gdm.d && \
sudo tee /etc/dconf/profile/gdm > /dev/null << 'EOF'
user-db:user
system-db:gdm
EOF
sudo tee /etc/dconf/db/gdm.d/00-scaling > /dev/null << 'EOF'
[org/gnome/desktop/interface]
text-scaling-factor=1.5
EOF
sudo dconf update
```

---

## 📦 Step 4: Install Steam and Session Components

Clone the reference repository to install drivers and the Wayland session file (`steam.desktop`):

```bash
sudo dnf install -y git
if [ ! -d "/tmp/gamescope-guide" ]; then
    git clone https://github.com/shahnawazshahin/steam-using-gamescope-guide.git /tmp/gamescope-guide
fi
cd /tmp/gamescope-guide
sudo ./installer.sh
```

> **Important Note:** The script installs the session file under the exact name **`steam.desktop`** inside `/usr/share/wayland-sessions/`. This name (`steam`) will be used by our D-Bus switching scripts.

---

## 🔒 Step 5: Secure `sudoers` Privileges

To allow switching scripts to modify the active session and restart GDM in the background without prompting for a password, configure `NOPASSWD` rules:

```bash
sudo tee /etc/sudoers.d/ps5-session > /dev/null << 'EOF'
ps5 ALL=(ALL) NOPASSWD: /usr/local/bin/switch-session, /usr/bin/steamos-session-select, /usr/bin/sed, /usr/bin/systemctl restart gdm, /usr/bin/busctl
EOF
sudo chmod 440 /etc/sudoers.d/ps5-session
sudo visudo -c -f /etc/sudoers.d/ps5-session
```

---

## ⚙️ Step 6: Create the Switching Script (GNOME -> Steam)

This script uses AccountsService D-Bus API (`busctl`) for real-time atomic session changes, avoiding caching delays:

```bash
sudo tee /usr/local/bin/switch-session > /dev/null << 'EOF'
#!/bin/bash
USER_ID=$(id -u ps5)
CURRENT=$(busctl get-property org.freedesktop.Accounts /org/freedesktop/Accounts/User$USER_ID org.freedesktop.Accounts.User Session | awk -F'"' '{print $2}')

if [ "$CURRENT" = "steam" ]; then
    TARGET="gnome"
else
    TARGET="steam"
fi

sudo busctl call org.freedesktop.Accounts /org/freedesktop/Accounts/User$USER_ID org.freedesktop.Accounts.User SetSession s "$TARGET"
sudo systemctl restart gdm
EOF
sudo chmod +x /usr/local/bin/switch-session
```

---

## ↩️ Step 7: Configure Return to Desktop (Steam -> GNOME)

Inside Big Picture mode, selecting **Switch to Desktop** triggers the system script `steamos-session-select`. We configure it to target the GNOME session cleanly:

```bash
sudo tee /usr/bin/steamos-session-select > /dev/null << 'EOF'
#!/bin/bash
USER_ID=$(id -u ps5)
sudo busctl call org.freedesktop.Accounts /org/freedesktop/Accounts/User$USER_ID org.freedesktop.Accounts.User SetSession s "gnome"
sudo systemctl restart gdm
EOF
sudo chmod +x /usr/bin/steamos-session-select
```

---

## 🖥️ Step 8: Create Desktop Launcher and App Shortcut

Finally, create a desktop and application launcher to toggle sessions with a single click from GNOME:

```bash
mkdir -p ~/.local/share/applications ~/Bureau ~/Desktop

cat << 'EOF' > ~/.local/share/applications/switch-session.desktop
[Desktop Entry]
Type=Application
Name=Switch Session (GNOME / Steam)
Comment=Switch between GNOME and Steam Big Picture
Exec=/usr/local/bin/switch-session
Icon=steam
Terminal=false
Categories=Game;Utility;
EOF

chmod +x ~/.local/share/applications/switch-session.desktop
cp ~/.local/share/applications/switch-session.desktop ~/Bureau/
cp ~/.local/share/applications/switch-session.desktop ~/Desktop/ 2>/dev/null || true
update-desktop-database ~/.local/share/applications/
```

---

## 🎮 Final Test

Reboot your console (`sudo reboot`). You can now trigger session switching from the desktop icon or application menu, and return directly to GNOME from the Steam Big Picture power menu in one click—completely passwordless!