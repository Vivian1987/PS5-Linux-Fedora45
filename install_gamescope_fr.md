# Bascule Bidirectionnelle GNOME <-> Steam Big Picture (Fedora 45 / PS5)

Ce guide détaille la mise en place d'un environnement de session stable, automatique et sans glitch vidéo permettant de basculer instantanément d'un simple clic (ou directement depuis le menu de Steam) entre le bureau **GNOME** et l'interface **Steam Big Picture** (Gamescope) sur une console PS5 sous Fedora 45.

## 🛠️ Étape 1 : Vérification du compte utilisateur et d'AccountsService

Avant de configurer le démarrage automatique, assurez-vous que votre compte utilisateur (`ps5`) est déverrouillé et que sa configuration AccountsService est initialisée sur GNOME par défaut.

```bash
# Vérifier l'utilisateur actif
whoami

# Définir GNOME par défaut sans écraser les paramètres de langue ou d'icône
sudo sed -i 's/^Session=.*/Session=gnome/' /var/lib/AccountsService/users/ps5 && sudo passwd -u ps5 2>/dev/null || true
```

## 🚀 Étape 2 : Configuration de l'auto-connexion GDM

Configurez GDM proprement avec un seul bloc `[daemon]` afin que la console ouvre automatiquement la session sans demander de mot de passe au démarrage ni lors des changements de session :

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

## 🔍 Étape 3 : Mise à l'échelle de l'affichage GDM (Optionnel pour 4K / PS5)

Si vous utilisez un écran 4K, ajuster le facteur d'échelle du texte garantit la lisibilité de l'interface de connexion si celle-ci vient à s'afficher :

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

## 📦 Étape 4 : Installation de Steam et des fichiers de session

Clonez le dépôt de référence et lancez son script d'installation depuis l'intérieur de son répertoire afin d'installer le fichier de session Wayland (`steam.desktop`) :

```bash
sudo dnf install -y git
if [ ! -d "/tmp/gamescope-guide" ]; then
    git clone https://github.com/shahnawazshahin/steam-using-gamescope-guide.git /tmp/gamescope-guide
fi
cd /tmp/gamescope-guide
sudo ./installer.sh
```

> **Note importante :** L'installateur crée le fichier **`steam.desktop`** dans `/usr/share/wayland-sessions/`. L'identifiant exact de la session utilisé par AccountsService est donc `steam`.

## 🔒 Étape 5 : Sécurisation des privilèges `sudoers`

Pour permettre aux scripts de bascule de lancer une unité système détachée (`systemd-run`) qui ferme proprement la session et redémarre GDM sans demander de mot de passe, configurez les règles `NOPASSWD` suivantes :

```bash
sudo tee /etc/sudoers.d/ps5-session > /dev/null << 'EOF'
ps5 ALL=(ALL) NOPASSWD: /usr/local/bin/switch-session, /usr/bin/steamos-session-select, /usr/bin/systemd-run *
EOF
sudo chmod 440 /etc/sudoers.d/ps5-session
sudo visudo -c -f /etc/sudoers.d/ps5-session
```

## ⚙️ Étape 6 : Création du script de bascule détaché (GNOME -> Steam)

Pour éviter les artéfacts et glitchs d'affichage sur le GPU AMD de la PS5 (`amdgpu`) tout en garantissant le redémarrage automatique de GDM après la déconnexion, ce script utilise :
1. **`busctl`** pour mettre à jour AccountsService de manière atomique et en temps réel.
2. **`systemd-run`** pour détacher le processus de transition de la session utilisateur afin qu'il survive à `loginctl terminate-user`.
3. **`loginctl terminate-user` + `sleep 2`** pour laisser au sous-système DRM/KMS le temps de libérer proprement les buffers vidéo avant de relancer GDM.

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

## ↩️ Étape 7 : Configuration du retour au bureau (Steam -> GNOME)

Dans le mode Steam Big Picture, sélectionner **Marche/Arrêt -> Passer au bureau** exécute `/usr/bin/steamos-session-select`. Configurez-le avec la même logique de nettoyage détaché :

```bash
sudo tee /usr/bin/steamos-session-select > /dev/null << 'EOF'
#!/bin/bash
USER_ID=$(id -u ps5)
sudo systemd-run --quiet --collect bash -c "busctl call org.freedesktop.Accounts /org/freedesktop/Accounts/User$USER_ID org.freedesktop.Accounts.User SetSession s 'gnome' && loginctl terminate-user ps5 && sleep 2 && systemctl restart gdm"
EOF
sudo chmod +x /usr/bin/steamos-session-select
```

## 🖥️ Étape 8 : Création du lanceur sur le bureau et dans le menu

Créez le raccourci sur le bureau et dans le menu des applications pour basculer de session d'un simple clic depuis GNOME :

```bash
mkdir -p ~/.local/share/applications ~/Bureau ~/Desktop

cat << 'EOF' > ~/.local/share/applications/switch-session.desktop
[Desktop Entry]
Type=Application
Name=Basculer Session (GNOME / Steam)
Comment=Basculer entre GNOME et Steam Big Picture
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

## 🎮 Test Final

Redémarrez votre console (`sudo reboot`). Vous pouvez désormais passer de GNOME à Steam via l'icône du bureau ou du menu, et revenir sur GNOME depuis le menu Marche/Arrêt de Steam Big Picture — sans mot de passe et sans glitch d'affichage.