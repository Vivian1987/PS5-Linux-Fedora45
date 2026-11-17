# 🚀 Local AI Environment Setup on Fedora 45

This guide details the installation and management of a complete, 100% local AI development environment on **Fedora 45**, using **Ollama**, **Open WebUI** (via Podman), and **VSCodium**.

## 📋 Table of Contents

1. [Installing Ollama](#-1-installing-ollama)
2. [Installing VSCodium](#-2-installing-vscodium)
3. [Installing Open WebUI (via Podman)](#-3-installing-open-webui-via-podman)
4. [Managing Ollama Models](#-4-managing-ollama-models)
5. [Service Administration & Management](#-5-service-administration--management)

---

## 🧠 1. Installing Ollama

Before installing Ollama, you need to install `pciutils` (essential for hardware detection, especially on specific setups like a PS5 running Linux).

```bash
# Install hardware detection dependency
sudo dnf install -y pciutils

# Official Ollama installation script
curl -fsSL https://ollama.com/install.sh | sh
```

### First Model Download

Choose the model size based on your hardware resources:
* `:3b`: Recommended if you are running strictly on **CPU**.
* `:7b`: Recommended if you have a **dedicated GPU** or at least **16 GB of RAM**.

```bash
ollama pull qwen2.5-coder:3b
```

---

## 💻 2. Installing VSCodium

Add the official VSCodium repository and install it via `dnf`:

```bash
# Import the GPG key
sudo rpmkeys --import https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/-/raw/master/pub.gpg

# Add the DNF repository
printf "[gitlab.com_paulcarroty_vscodium_repo]\nname=download.vscodium.com\nbaseurl=https://download.vscodium.com/rpms/\nenabled=1\ngpgcheck=1\nrepo_gpgcheck=1\ngpgkey=https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/-/raw/master/pub.gpg\nmetadata_expire=1h\n" | sudo tee /etc/yum.repos.d/vscodium.repo

# Install VSCodium
sudo dnf install -y codium
```

---

## 🌐 3. Installing Open WebUI (via Podman)

Open WebUI provides a ChatGPT-like web interface connected directly to your local Ollama instance.

```bash
# Install Podman
sudo dnf install -y podman

# Run the Open WebUI container
podman run -d --network=host \
  -v open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

> **Accessing the UI:** Once the container is running, open your browser and navigate to `http://localhost:8080`.

---

## 📦 4. Managing Ollama Models

### Useful Model Library

Here is a curated list of models (coding, reasoning, lightweight) ready to be pulled according to your needs:

```bash
# Coding-oriented models
ollama pull qwen2.5-coder:3b
ollama pull qwen2.5-coder:7b

# Reasoning & General-purpose models
ollama pull deepseek-r1:8b
ollama pull eriktonon/deepseek-v4.1-flash-1017:9b
ollama pull mimo-v2.6:9b
ollama pull qwen3.5:4b

# Ultra-lightweight & Gemma models
ollama pull nemotron-3.5-bolt:e4b
ollama pull gemma4:e2b
ollama pull gemma4:e4b
ollama pull gemma4:12b
```

### Common Model Commands

* **List all installed models:**
  ```bash
  ollama list
  ```

* **Stop a running model** *(must be done before removing an active model)*:
  ```bash
  ollama stop <model_name>
  ```

* **Remove a model** *(to free up disk space)*:
  ```bash
  ollama rm <model_name>
  ```

---

## ⚙️ 5. Service Administration & Management

### Open WebUI Management (Podman)

| Action | Command |
| :--- | :--- |
| **Start** the container | `podman start open-webui` |
| **Stop** the container | `podman stop open-webui` |
| **Check container status** | `podman ps -a` |
| **View logs** (last 30 lines) | `podman logs --tail 30 open-webui` |

> **Note on stopping:** When you run `podman stop open-webui`, the terminal simply outputs `open-webui` (and `podman ps` will show no active containers). Even though the container uses the `--restart always` policy, Podman will not restart it automatically until you reboot your PC.

### Ollama Management (Systemd)

* **Stop the background service:**
  ```bash
  sudo systemctl stop ollama
  ```
  *Verification:* `systemctl is-active ollama` should return `inactive`.

* **Disable automatic startup on boot (Optional):**
  ```bash
  sudo systemctl disable ollama
  ```

### Restart Everything at Once

To start both the Ollama service and the Open WebUI container simultaneously:

```bash
sudo systemctl start ollama && podman start open-webui
```
