# 🎮 PS5 Fedora 45 (Wayland) — Build Overview & Changelog (FR / EN)

---

## 🇫🇷 Version Française

### 📦 Résumé de ce qui est inclus dans la Build
Image système **Fedora 45 (x86_64)** optimisée et préconfigurée spécifiquement pour la **PlayStation 5** sous environnement graphique **GNOME (Wayland)**.

### 🆚 Différences par rapport à une build Fedora basique
Contrairement à une image Fedora Workstation générique, cette build apporte des modifications profondes adaptées à l'architecture matérielle de la PS5 :
* **Noyau Linux & Boot EFI spécifiques PS5 :** Remplacement du noyau standard par `linux-ps5` avec génération automatisée de `bzImage` et `initrd.img` directement sur la partition `/boot/efi` via `zz-update-boot`.
* **Pilotes Matériels Propriétaires & Décompressés :** Décompression manuelle des firmwares AMD `cyan_skillfish` et gestion dédiée du module Wi-Fi interne NXP IW620 (`moal`/`mlan`) via un service systemd personnalisé (`ps5-iw620.service`).
* **Gestion Thermique & Performances :** Compilation de `ps5-linux-tools` (`ps5_control`) et activation automatique de la ventilation et du mode Boost au démarrage (`ps5-performance.service`).
* **Prêt pour le Gaming (Steam / Proton) :** Intégration native des bibliothèques graphiques 32 bits (`.i686`) et de **Decky Loader** dès le premier démarrage, sans configuration manuelle requise.
* **Outils d'Administration Sur-Mesure :** Intégration de **PS5 Fedora Manager** (GTK4/Libadwaita) avec règles Polkit préconfigurées pour piloter la console sans ligne de commande.
* **Expérience "Console" Sans Friction :** Connexion automatique (Autologin GDM) sur le compte `ps5`, assistant de premier démarrage désactivé, droits `sudo` sans mot de passe (`NOPASSWD`), SELinux désactivé et redimensionnement automatique de la partition racine (`grow-rootfs.service`).

### 📋 Liste des Paquets, Dépendances et Dépôts par Catégorie

#### 1. Dépôts Tierces Parties Activés
* **RPM Fusion Free & Non-Free :** Accès aux pilotes propriétaires, codecs multimédias et paquets Steam.
* **Flathub (Flatpak) :** Dépôt officiel d'applications Flatpak préconfiguré.

#### 2. Pilotes Graphiques (GPU AMD) & Gaming 32-bit
* **Pilotes 64-bit :** `mesa-dri-drivers`, `mesa-vulkan-drivers`, `mesa-va-drivers`, `vulkan-tools`, `mesa-demos`, `libinput`
* **Pilotes 32-bit (Steam / Proton) :** `mesa-vulkan-drivers.i686`, `mesa-dri-drivers.i686`, `libva.i686`
* **Outils Gaming :** `Decky Loader` (installé pour l'utilisateur `ps5`)

#### 3. Interface PS5 Fedora Manager & Dépendances GUI
* **Application principale :** `ps5-fedora-manager` (cloné depuis GitHub et installé via `./install.sh` avec règles Polkit)
* **Dépendances GTK4 / Libadwaita :** `python3-gobject`, `gtk4`, `libadwaita`, `gobject-introspection`, `desktop-file-utils`, `xhost`, `polkit`
* **Dépendances Python / Outils tiers :** `python3-PyQt6`, `zenity`, `newt`

#### 4. Réseau, Wi-Fi PS5 (IW620) & Bluetooth
* **Gestion Réseau & Wi-Fi :** `NetworkManager`, `NetworkManager-wifi`, `network-manager-applet`, `linux-firmware`, `iw`, `wpa_supplicant`, `wireless-regdb`
* **Bluetooth :** `bluez`, `bluez-tools`
* **Accès Distant :** `openssh-server` (activé, connexion root désactivée)

#### 5. Compilation, Système, Disque & FUSE
* **Outils de compilation (pour `ps5_control`) :** `git`, `make`, `gcc`, `zlib-devel`
* **Support AppImage & Systèmes de fichiers :** `fuse`, `fuse-libs`
* **Gestion Disque & Mémoire :** `e2fsprogs`, `parted`, `cloud-utils-growpart`, `zram-generator-defaults` (ZRAM configuré à 50 % de la RAM en `zstd`), `dracut`, `kmod`, `kexec-tools`

#### 6. Environnement GNOME, Applications & Langues (FR/EN)
* **Bureau GNOME (Wayland / Xwayland) :** `gnome-shell`, `gnome-session`, `gnome-terminal`, `gdm`, `mutter`, `nautilus`, `gnome-control-center`, `gnome-settings-daemon`, `gnome-keyring`, `gnome-tweaks`, `gnome-extensions-app`, `gnome-software`, `xdg-desktop-portal-gnome`, `xorg-x11-server-Xwayland`
* **Applications incluses :** `firefox`, `Balena Etcher` (RPM), `Extension Manager` (Flatpak), `gnome-text-editor`, `gnome-system-monitor`, `gnome-disk-utility`, `gnome-calculator`, `file-roller`, `loupe`, `evince`, `baobab`
* **Localisation & Polices :** `glibc-all-langpacks`, `langpacks-fr`, `langpacks-en`, `firefox-langpacks`, `ibus`, `ibus-gtk3`, `ibus-gtk4`, `dejavu-sans-fonts`, `liberation-fonts`, `fontconfig`

### 📝 Changelog (Dernières modifications)
* **✨ Ajouts :**
  * Remplacement de l'ancien gestionnaire Bash/Zenity par le nouveau **PS5 Fedora Manager (GTK4 / Libadwaita)** déployé via `git clone` et `./install.sh`.
  * Ajout des dépendances natives GTK4/Libadwaita (`python3-gobject`, `gtk4`, `libadwaita`, `gobject-introspection`, `desktop-file-utils`, `xhost`).
  * Ajout des bibliothèques graphiques **32 bits** (`mesa-vulkan-drivers.i686`, `mesa-dri-drivers.i686`, `libva.i686`) pour la compatibilité Steam.
  * Ajout de `fuse` et `fuse-libs` pour le support natif des AppImages.
* **🔄 Modifications :**
  * Sécurisation de l'installation du noyau avec `--replacefiles` (`rpm -ivh --nosignature --replacefiles`) et appel explicite à `zz-update-boot` pour mettre à jour `/boot/efi`.
  * Conservation de `python3-PyQt6` dans les paquets système pour la compatibilité des outils tiers.
* **🗑️ Suppressions :**
  * Retrait de `ps5-hwcon-gui` (remplacé par les fonctionnalités intégrées du nouveau *PS5 Fedora Manager*).
  * Retrait de la configuration statique de mise en veille GNOME (`keyfile`), désormais gérée directement depuis *PS5 Fedora Manager*.
  * Confirmation de l'absence de `gparted` (seul `parted` en CLI est conservé pour `grow-rootfs`).

---

## 🇬🇧 English Version

### 📦 Build Overview
A custom, lightweight **Fedora 45 (x86_64)** system image specifically optimized and preconfigured for the **PlayStation 5**, running the **GNOME (Wayland)** desktop environment.

### 🆚 Differences Compared to a Standard Fedora Build
Unlike a generic Fedora Workstation image, this build introduces deep system-level modifications tailored for PS5 hardware:
* **Custom PS5 Kernel & EFI Boot Deployment:** Replaces the stock kernel with `linux-ps5`, automatically generating `bzImage` and `initrd.img` directly onto `/boot/efi` via `zz-update-boot`.
* **Custom & Uncompressed Hardware Firmware:** Manually decompresses AMD `cyan_skillfish` GPU firmwares and manages the internal NXP IW620 Wi-Fi chip (`moal`/`mlan`) via a dedicated systemd service (`ps5-iw620.service`).
* **Thermal & Power Management:** Compiles `ps5-linux-tools` (`ps5_control`) from source and enables automatic fan control and Boost mode at boot (`ps5-performance.service`).
* **Out-of-the-Box Gaming Ready (Steam / Proton):** Pre-installs 32-bit graphics libraries (`.i686`) and **Decky Loader** during image creation with zero manual setup required.
* **Custom Management Suite:** Integrates **PS5 Fedora Manager** (GTK4/Libadwaita) with preconfigured Polkit rules for GUI-based system and hardware control.
* **Console-Like Frictionless Experience:** Preconfigured GDM autologin on the `ps5` user account, initial setup wizard bypassed, passwordless `sudo` (`NOPASSWD`), SELinux disabled, and automatic root partition expansion (`grow-rootfs.service`).

### 📋 Categorized List of Added Packages, Dependencies & Repositories

#### 1. Enabled Third-Party Repositories
* **RPM Fusion Free & Non-Free:** Provides proprietary drivers, multimedia codecs, and Steam packages.
* **Flathub (Flatpak):** Official Flatpak application repository preconfigured out of the box.

#### 2. Graphics Drivers (AMD GPU) & 32-bit Gaming Support
* **64-bit Drivers:** `mesa-dri-drivers`, `mesa-vulkan-drivers`, `mesa-va-drivers`, `vulkan-tools`, `mesa-demos`, `libinput`
* **32-bit Drivers (Steam / Proton):** `mesa-vulkan-drivers.i686`, `mesa-dri-drivers.i686`, `libva.i686`
* **Gaming Tools:** `Decky Loader` (pre-installed for the `ps5` user)

#### 3. PS5 Fedora Manager & GUI Dependencies
* **Main Application:** `ps5-fedora-manager` (cloned from GitHub and deployed via `./install.sh` with Polkit policies)
* **GTK4 / Libadwaita Dependencies:** `python3-gobject`, `gtk4`, `libadwaita`, `gobject-introspection`, `desktop-file-utils`, `xhost`, `polkit`
* **Python / Third-Party GUI Libraries:** `python3-PyQt6`, `zenity`, `newt`

#### 4. Networking, PS5 Wi-Fi (IW620) & Bluetooth
* **Networking & Wi-Fi:** `NetworkManager`, `NetworkManager-wifi`, `network-manager-applet`, `linux-firmware`, `iw`, `wpa_supplicant`, `wireless-regdb`
* **Bluetooth:** `bluez`, `bluez-tools`
* **Remote Access:** `openssh-server` (enabled, root login disabled)

#### 5. Build Tools, System, Disk & FUSE
* **Compilation Tools (for `ps5_control`):** `git`, `make`, `gcc`, `zlib-devel`
* **AppImage & Filesystem Support:** `fuse`, `fuse-libs`
* **Disk & Memory Management:** `e2fsprogs`, `parted`, `cloud-utils-growpart`, `zram-generator-defaults` (ZRAM configured at 50% RAM with `zstd`), `dracut`, `kmod`, `kexec-tools`

#### 6. GNOME Desktop, Applications & Localization (FR/EN)
* **GNOME Desktop (Wayland / Xwayland):** `gnome-shell`, `gnome-session`, `gnome-terminal`, `gdm`, `mutter`, `nautilus`, `gnome-control-center`, `gnome-settings-daemon`, `gnome-keyring`, `gnome-tweaks`, `gnome-extensions-app`, `gnome-software`, `xdg-desktop-portal-gnome`, `xorg-x11-server-Xwayland`
* **Pre-installed Apps:** `firefox`, `Balena Etcher` (RPM), `Extension Manager` (Flatpak), `gnome-text-editor`, `gnome-system-monitor`, `gnome-disk-utility`, `gnome-calculator`, `file-roller`, `loupe`, `evince`, `baobab`
* **Localization & Fonts:** `glibc-all-langpacks`, `langpacks-fr`, `langpacks-en`, `firefox-langpacks`, `ibus`, `ibus-gtk3`, `ibus-gtk4`, `dejavu-sans-fonts`, `liberation-fonts`, `fontconfig`

### 📝 Changelog (Latest Updates)
* **✨ Added:**
  * Replaced the legacy Bash/Zenity script with the new **PS5 Fedora Manager (GTK4 / Libadwaita)** deployed via `git clone` and `./install.sh`.
  * Added native GTK4/Libadwaita dependencies (`python3-gobject`, `gtk4`, `libadwaita`, `gobject-introspection`, `desktop-file-utils`, `xhost`).
  * Added **32-bit** graphics libraries (`mesa-vulkan-drivers.i686`, `mesa-dri-drivers.i686`, `libva.i686`) for Steam compatibility.
  * Added `fuse` and `fuse-libs` for native AppImage support.
* **🔄 Changed:**
  * Hardened custom kernel installation using `--replacefiles` (`rpm -ivh --nosignature --replacefiles`) and explicitly invoked `zz-update-boot` to update `/boot/efi`.
  * Retained `python3-PyQt6` in system packages for third-party tool compatibility.
* **🗑️ Removed:**
  * Removed `ps5-hwcon-gui` (superseded by features built into the new *PS5 Fedora Manager*).
  * Removed static GNOME power/sleep `keyfile` overrides (power management is now handled directly within *PS5 Fedora Manager*).
  * Confirmed removal of `gparted` (only CLI `parted` is kept for `grow-rootfs`).
