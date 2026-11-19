# 🎮 PS5 Fedora 45 (Wayland) — Résumé de la Build & Changelog

Image système **Fedora 45 (x86_64)** optimisée et préconfigurée spécifiquement pour la **PlayStation 5** sous environnement graphique **GNOME (Wayland)**.

---

## 📦 Résumé de ce qui est inclus dans la Build

### 🖥️ Environnement de Bureau & Système
* **Base OS :** Fedora 45 allégée, configurée en français/anglais (`fr`, `en`).
* **Interface Graphique :** GNOME Desktop complet sous **Wayland** (avec support `Xwayland` pour la compatibilité des applications et jeux).
* **Session & Connexion :** Compte utilisateur par défaut `ps5` (mot de passe `ps5`), droits administrateur (`sudo`) sans mot de passe, **autologin GDM** activé et assistant de premier démarrage désactivé.
* **Performances & Stockage :**
  * **ZRAM :** Swap compressé en mémoire (`zstd`, 50 % de la RAM totale).
  * **Partitionnement automatique :** Service `grow-rootfs.service` pour étendre automatiquement la partition système au premier démarrage.
  * **FUSE :** Support complet (`fuse`, `fuse-libs`) pour l'exécution d'AppImages et systèmes de fichiers virtuels.

### 🔧 Pilotes & Optimisations Matérielles PS5
* **Noyau Linux Custom (`linux-ps5`) :** Déploiement automatisé des RPM personnalisés avec mise à jour des modules (`depmod`) et génération de l'image de démarrage (`bzImage` / `initrd.img`) sur la partition `/boot/efi`.
* **GPU AMD (Oberon / Cyan Skillfish) :** Pilotes Mesa complets (DRI, Vulkan, VA-API), firmwares `cyan_skillfish` décompressés pour un chargement immédiat, et bibliothèques graphiques **32 bits** (`mesa-vulkan-drivers.i686`, `mesa-dri-drivers.i686`, `libva.i686`) prêtes pour **Steam / Proton**.
* **Wi-Fi Interne PS5 (NXP IW620) :** Gestion dédiée via blacklist du chargement automatique (`moal`/`mlan`) et service systemd `ps5-iw620.service` pour charger le firmware et activer la carte réseau proprement au démarrage.
* **Refroidissement & Puissance :** Compilation de `ps5-linux-tools` (`ps5_control`) et service `ps5-performance.service` activant automatiquement le contrôle du ventilateur et le mode Boost au démarrage.

### 🛠️ Logiciels & Outils Préinstallés
* **PS5 Fedora Manager :** Gestionnaire système, matériel et logiciel développé sur mesure en **GTK4 / Libadwaita** (`python3-gobject`, `gtk4`, `libadwaita`) avec règles Polkit intégrées.
* **Decky Loader :** Préinstallé pour l'utilisateur `ps5` (prêt pour Steam Big Picture / Gamescope).
* **Balena Etcher :** Utilitaire de flashage d'images disque / clés USB.
* **Outils GNOME & Web :** Firefox, Extension Manager (Flatpak/Flathub), Gestionnaire d'archives, Moniteur système, Éditeur de texte, Disques, Tweaks.
* **Dépôts activés :** RPM Fusion (Free & Non-Free) et Flathub.
* **Accès distant :** Serveur SSH (`sshd`) actif (connexion par mot de passe autorisée, accès direct `root` désactivé).

---

## 📝 Changelog (Dernières modifications)

### ✨ Ajouts
* **Nouveau PS5 Fedora Manager (GTK4 / Libadwaita) :** Remplacement de l'ancien script Bash/Zenity par la nouvelle version clonée depuis `nexgen999/ps5-fedora-manager` et déployée via `./install.sh`. Ajout des dépendances natives (`python3-gobject`, `gtk4`, `libadwaita`, `gobject-introspection`, `desktop-file-utils`, `xhost`) dans l'image.
* **Support Steam 32 bits :** Intégration des paquets `mesa-vulkan-drivers.i686`, `mesa-dri-drivers.i686` et `libva.i686` dès la construction de l'image.
* **Support AppImage / FUSE :** Ajout de `fuse` et `fuse-libs` dans les paquets système de base.

### 🔄 Modifications & Améliorations
* **Déploiement du Kernel sécurisé :** Ajout de l'option `--replacefiles` lors de l'installation des RPM du noyau (`rpm -ivh --nosignature --replacefiles`) et appel explicite à `zz-update-boot` pour fiabiliser la mise à jour de `/boot/efi`.
* **Bibliothèques Python :** Conservation de `python3-PyQt6` dans les paquets système pour la compatibilité des outils tiers.

### 🗑️ Suppressions & Nettoyage
* **Retrait de `ps5-hwcon-gui` :** Suppression du téléchargement et de l'installation de l'ancienne interface PyQt6 (remplacée par les fonctionnalités du nouveau *PS5 Fedora Manager*).
* **Retrait du bloc de mise en veille GNOME :** Suppression de la configuration statique `keyfile` dans `/home/ps5/.config/glib-2.0/settings/` (la gestion de l'énergie et de la veille est désormais pilotée directement depuis *PS5 Fedora Manager*).
* **Nettoyage des paquets :** Confirmation de l'absence de `gparted` (seul `parted` en ligne de commande est conservé pour le redimensionnement automatique `grow-rootfs`).
```