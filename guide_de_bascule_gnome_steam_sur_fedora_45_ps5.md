# Bascule Bidirectionnelle GNOME <-> Steam Big Picture (Fedora 45 / PS5)

Ce guide détaille la mise en place d'un environnement de session stable, automatique et sécurisé permettant de basculer instantanément d'un simple clic (ou depuis le menu Steam) entre le bureau **GNOME** et l'interface **Steam Big Picture** sur une console PS5 sous Fedora 45.

---

## 🛠️ Étape 1 : Vérification et configuration de l'utilisateur

Avant de configurer le démarrage automatique, nous nous assurons que le compte utilisateur (par exemple `ps5`) est actif et que son fichier AccountsService est configuré sur GNOME par défaut.

```bash
# Vérifier l'utilisateur connecté
whoami

# S'assurer que le fichier AccountsService existe et définir GNOME par défaut
sudo sed -i 's/^Session=.*/Session=gnome/' /var/lib/AccountsService/users/ps5 && sudo passwd -u ps5 2>/dev/null || true
```

---

## 🚀 Étape 2 : Configuration de l'auto-connexion (GDM)

Pour que la console se connecte directement à la session sans demander de mot de passe au démarrage, configurez le fichier de GDM :

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

## 🔍 Étape 3 : Mise à l'échelle de l'affichage (GDM)

Si vous utilisez un écran 4K (comme sur PS5), l'ajustement du facteur d'échelle garantit que l'écran de connexion reste parfaitement lisible :

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

## 📦 Étape 4 : Installation de Steam et des composants de session

Nous clonons le dépôt de référence pour installer les pilotes et le fichier de session Wayland (`steam.desktop`) :

```bash
sudo dnf install -y git
if [ ! -d "/tmp/gamescope-guide" ]; then
    git clone https://github.com/shahnawazshahin/steam-using-gamescope-guide.git /tmp/gamescope-guide
fi
cd /tmp/gamescope-guide
sudo ./installer.sh
```
> **Note importante :** Le script installe le fichier de session sous le nom exact **`steam.desktop`** dans `/usr/share/wayland-sessions/`. C'est ce nom (`steam`) qui sera utilisé pour les bascules D-Bus.

---

## 🔒 Étape 5 : Sécurisation des privilèges (`sudoers`)

Pour que les scripts de bascule puissent modifier la session active et redémarrer GDM en arrière-plan sans demander de mot de passe, configurez les règles `NOPASSWD` :

```bash
sudo tee /etc/sudoers.d/ps5-session > /dev/null << 'EOF'
ps5 ALL=(ALL) NOPASSWD: /usr/local/bin/switch-session, /usr/bin/steamos-session-select, /usr/bin/sed, /usr/bin/systemctl restart gdm, /usr/bin/busctl
EOF
sudo chmod 440 /etc/sudoers.d/ps5-session
sudo visudo -c -f /etc/sudoers.d/ps5-session
```

---

## ⚙️ Étape 6 : Création du script de bascule (GNOME -> Steam)

Ce script utilise l'API D-Bus (`busctl`) d'AccountsService pour basculer instantanément de GNOME vers Steam (ou inversement) en temps réel, évitant ainsi les délais de cache du démon AccountsService :

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

## ↩️ Étape 7 : Configuration du retour au bureau (Steam -> GNOME)

Dans l'interface Big Picture, le bouton **Passer au bureau** appelle le script système `steamos-session-select`. Nous le configurons pour cibler proprement la session GNOME :

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

## 🖥️ Étape 8 : Création du raccourci graphique et du lanceur

Enfin, créez le lanceur sur le bureau et dans le menu des applications pour pouvoir basculer d'un simple clic depuis GNOME :

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

---

## 🎮 Test final
Redémarrez votre console (`sudo reboot`). Vous pouvez désormais lancer la bascule depuis l'icône du bureau ou du menu, et revenir sur GNOME directement depuis le menu de Steam Big Picture en un clic et sans mot de passe !