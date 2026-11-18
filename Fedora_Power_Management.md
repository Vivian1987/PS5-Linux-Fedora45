# 🔋 Gestion de l'alimentation sous Fedora GNOME / Power Management on Fedora GNOME

[🇫🇷 Français](#-français) | [🇬🇧 English](#-english)

---

## 🇫🇷 Français

Ce guide regroupe les **6 commandes exactes** pour gérer rapidement la mise en veille sous **Fedora (environnement GNOME)**, que ce soit au niveau du système, de l'écran, ou les deux combinés.

### 1. Mise en veille du système (Suspend / Hibernation via `systemd`)

* **Désactiver la veille système :**
  ```bash
  sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
  ```

* **Activer la veille système :**
  ```bash
  sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
  ```

### 2. Mise en veille de l'écran (Session GNOME)

* **Désactiver la veille de l'écran :**
  ```bash
  gsettings set org.gnome.desktop.session idle-delay 0
  ```

* **Activer la veille de l'écran (ex. 5 minutes / 300 secondes) :**
  ```bash
  gsettings set org.gnome.desktop.session idle-delay 300
  ```

### 3. Mise en veille complète (Système + Écran combinés)

* **Couper complètement toute veille (système + écran) :**
  ```bash
  sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target && gsettings set org.gnome.desktop.session idle-delay 0 && gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
  ```

* **Réactiver la veille complète (système + écran) :**
  ```bash
  sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target && gsettings set org.gnome.desktop.session idle-delay 300 && gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'suspend'
  ```

---

## 🇬🇧 English

This guide provides the **6 exact commands** to easily manage sleep and screen blanking on **Fedora (GNOME desktop)**, covering system-level, screen-level, and combined global settings.

### 1. System Sleep (Suspend / Hibernate via `systemd`)

* **Disable system sleep:**
  ```bash
  sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
  ```

* **Enable system sleep:**
  ```bash
  sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
  ```

### 2. Screen Blanking (GNOME Session)

* **Disable screen sleep/blanking:**
  ```bash
  gsettings set org.gnome.desktop.session idle-delay 0
  ```

* **Enable screen sleep/blanking (e.g., 5 minutes / 300 seconds):**
  ```bash
  gsettings set org.gnome.desktop.session idle-delay 300
  ```

### 3. Full Sleep Management (System + Screen Combined)

* **Completely disable all sleep modes (system + screen):**
  ```bash
  sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target && gsettings set org.gnome.desktop.session idle-delay 0 && gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
  ```

* **Re-enable full sleep modes (system + screen):**
  ```bash
  sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target && gsettings set org.gnome.desktop.session idle-delay 300 && gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'suspend'
  ```
