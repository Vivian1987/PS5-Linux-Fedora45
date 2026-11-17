# 🚀 Setup Environnement IA Locale sous Fedora 45

Ce guide détaille l'installation et la gestion d'un environnement de développement IA complet et 100 % local sous **Fedora 45**, utilisant **Ollama**, **Open WebUI** (via Podman) et **VSCodium**.

---

## 📋 Table des matières

1. [Installation d'Ollama](#-1-installation-dollama)
2. [Installation de VSCodium](#-2-installation-de-vscodium)
3. [Installation d'Open WebUI (Podman)](#-3-installation-dopen-webui-via-podman)
4. [Gestion des modèles Ollama](#-4-gestion-des-modèles-ollama)
5. [Administration et gestion des services](#-5-administration-et-gestion-des-services)

---

## 🧠 1. Installation d'Ollama

Avant d'installer Ollama, il est nécessaire d'installer `pciutils` (indispensable pour la détection matérielle, notamment sur des configurations spécifiques comme la PS5 sous Linux).

```bash
# Installation de la dépendance matérielle
sudo dnf install -y pciutils

# Installation officielle d'Ollama
curl -fsSL https://ollama.com/install.sh | sh
```

### Premier téléchargement de modèle
Choisis la taille du modèle en fonction de tes ressources matérielles :
* `:3b` : Recommandé si tu tournes uniquement sur **CPU**.
* `:7b` : Recommandé si tu disposes d'un **GPU dédié** ou d'au moins **16 Go de RAM**.

```bash
ollama pull qwen2.5-coder:3b
```

---

## 💻 2. Installation de VSCodium

Ajout du dépôt officiel de VSCodium et installation via `dnf` :

```bash
# Importation de la clé GPG
sudo rpmkeys --import https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/-/raw/master/pub.gpg

# Ajout du dépôt DNF
printf "[gitlab.com_paulcarroty_vscodium_repo]\nname=download.vscodium.com\nbaseurl=https://download.vscodium.com/rpms/\nenabled=1\ngpgcheck=1\nrepo_gpgcheck=1\ngpgkey=https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/-/raw/master/pub.gpg\nmetadata_expire=1h\n" | sudo tee /etc/yum.repos.d/vscodium.repo

# Installation
sudo dnf install -y codium
```

---

## 🌐 3. Installation d'Open WebUI (via Podman)

Open WebUI permet d'avoir une interface web type ChatGPT connectée directement à ton instance locale d'Ollama.

```bash
# Installation de Podman
sudo dnf install -y podman

# Lancement du conteneur Open WebUI
podman run -d --network=host \
  -v open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

> **Accès à l'interface :** Une fois le conteneur démarré, ouvre ton navigateur à l'adresse `http://localhost:8080`.

---

## 📦 4. Gestion des modèles Ollama

### Bibliothèque de modèles utiles
Voici une sélection de modèles (code, raisonnement, légers) prêts à être téléchargés selon tes besoins :

```bash
# Modèles orientés Code
ollama pull qwen2.5-coder:3b
ollama pull qwen2.5-coder:7b

# Modèles de Raisonnement & Polyvalents
ollama pull deepseek-r1:8b
ollama pull eriktonon/deepseek-v4.1-flash-1017:9b
ollama pull mimo-v2.6:9b
ollama pull qwen3.5:4b

# Modèles Ultra-légers & Gemma
ollama pull nemotron-3.5-bolt:e4b
ollama pull gemma4:e2b
ollama pull gemma4:e4b
ollama pull gemma4:12b
```

### Commandes courantes pour les modèles

* **Lister les modèles installés :**
  ```bash
  ollama list
  ```

* **Arrêter un modèle en cours d'exécution** *(à faire avant de le supprimer)* :
  ```bash
  ollama stop <nom_du_modele>
  ```

* **Supprimer un modèle** *(pour libérer de l'espace disque)* :
  ```bash
  ollama rm <nom_du_modele>
  ```

---

## ⚙️ 5. Administration et gestion des services

### Gestion d'Open WebUI (Podman)

| Action | Commande |
| :--- | :--- |
| **Démarrer** le conteneur | `podman start open-webui` |
| **Arrêter** le conteneur | `podman stop open-webui` |
| **Voir l'état** des conteneurs | `podman ps -a` |
| **Consulter les logs** (30 dernières lignes) | `podman logs --tail 30 open-webui` |

> **Note sur l'arrêt :** Lorsque tu exécutes `podman stop open-webui`, le terminal renvoie simplement `open-webui`. Bien que le conteneur ait la politique `--restart always`, Podman ne le relancera pas tant que tu ne redémarres pas le PC.

### Gestion d'Ollama (Systemd)

* **Arrêter le service en arrière-plan :**
  ```bash
  sudo systemctl stop ollama
  ```
  *Vérification :* `systemctl is-active ollama` doit renvoyer `inactive`.

* **Empêcher le démarrage automatique au boot (Optionnel) :**
  ```bash
  sudo systemctl disable ollama
  ```

### Tout relancer en une seule commande
Pour redémarrer simultanément Ollama et l'interface Open WebUI :

```bash
sudo systemctl start ollama && podman start open-webui
```
