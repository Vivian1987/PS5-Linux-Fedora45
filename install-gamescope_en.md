# Bidirectional GNOME <-> Steam Big Picture Session Switching (Fedora 45 / PS5)

This guide details how to set up a stable, automatic, and glitch-free session environment allowing you to switch instantly with a single click (or directly from the Steam menu) between the **GNOME** desktop and the **Steam Big Picture** (Gamescope) interface on a PS5 console running Fedora 45.

## 🛠️ Step 1: User Account and AccountsService Verification

Before configuring automatic startup, ensure your user account (`ps5`) is unlocked and its AccountsService configuration defaults to GNOME.

```bash
# Check the current user
whoami

# Set GNOME as default without overwriting language/icon settings
sudo sed -i 's/^Session=.*/Session=gnome/' /var/lib/AccountsService/users/ps5 && sudo passwd -u ps5 2>/dev/null || true
```

## 🚀 Step 2: Configure GDM Auto-Login

Configure GDM cleanly with a single `[daemon]` block to allow the console to log in automatically without prompting for a password at boot or during session switches:

```bash
sudo tee /etc/gdm/custom.conf > /dev/null << 'EOF'
# GDM configuration storage

[daemon]
AutomaticLoginEnable=True
AutomaticLogin=ps5

[security]

[xdmcp]

[chooser]

[debug]
# Uncomment the line below to turn on debugging
#Enable=true
EOF
```

## 🔍 Step 3: GDM Display Scaling (Optional for 4K / PS5)

If you are using a 4K display, adjusting the text scaling factor ensures the login interface remains readable if ever displayed:

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

## 📦 Step 4: Install Steam and Session Components

Clone the reference repository and run the installer from inside its directory to install the Wayland session file (`steam.desktop`):

```bash
sudo dnf install -y git
if [ ! -d "/tmp/gamescope-guide" ]; then
    git clone https://github.com/shahnawazshahin/steam-using-gamescope-guide.git /tmp/gamescope-guide
fi
cd /tmp/gamescope-guide
sudo ./installer.sh
```

> **Important Note:** The installer creates **`steam.desktop`** inside `/usr/share/wayland-sessions/`. Therefore, the exact session identifier used by AccountsService is `steam`.

## 🔒 Step 5: Secure `sudoers` Privileges

To allow the switching scripts to spawn a detached system unit (`systemd-run`) that cleanly terminates the session and restarts GDM without password prompts, configure `NOPASSWD` rules:

```bash
sudo tee /etc/sudoers.d/ps5-session > /dev/null << 'EOF'
ps5 ALL=(ALL) NOPASSWD: /usr/local/bin/switch-session, /usr/bin/steamos-session-select, /usr/bin/systemd-run *
EOF
sudo chmod 440 /etc/sudoers.d/ps5-session
sudo visudo -c -f /etc/sudoers.d/ps5-session
```

## ⚙️ Step 6: Create the Detached Switching Script (GNOME -> Steam)

To prevent display artifacts/glitches on the PS5 AMD GPU (`amdgpu`) and ensure GDM restarts properly after user logout, this script uses:
1. **`busctl`** to atomically update AccountsService in real time.
2. **`systemd-run`** to detach the transition process from the user session so it survives `loginctl terminate-user`.
3. **`loginctl terminate-user` + `sleep 2`** to let the DRM/KMS subsystem release video buffers cleanly before restarting GDM.

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

sudo systemd-run --quiet --collect bash -c "busctl call org.freedesktop.Accounts /org/freedesktop/Accounts/User$USER_ID org.freedesktop.Accounts.User SetSession s '$TARGET' && loginctl terminate-user ps5 && sleep 2 && systemctl restart gdm"
EOF
sudo chmod +x /usr/local/bin/switch-session
```

## ↩️ Step 7: Configure Return to Desktop (Steam -> GNOME)

Inside Steam Big Picture mode, selecting **Power -> Switch to Desktop** calls `/usr/bin/steamos-session-select`. Configure it with the same detached cleanup logic:

```bash
sudo tee /usr/bin/steamos-session-select > /dev/null << 'EOF'
#!/bin/bash
USER_ID=$(id -u ps5)
sudo systemd-run --quiet --collect bash -c "busctl call org.freedesktop.Accounts /org/freedesktop/Accounts/User$USER_ID org.freedesktop.Accounts.User SetSession s 'gnome' && loginctl terminate-user ps5 && sleep 2 && systemctl restart gdm"
EOF
sudo chmod +x /usr/bin/steamos-session-select
```

## 🖥️ Step 8: Create Desktop Launcher and App Shortcut

Create a desktop and application menu launcher to toggle sessions with a single click from GNOME:

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

## 🎮 Final Test

Reboot your console (`sudo reboot`). You can now switch from GNOME to Steam via the desktop/menu icon and return to GNOME from the Steam Big Picture power menu—completely passwordless and free of display glitches.