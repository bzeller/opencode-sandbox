# OpenCode Sandbox Manager

A Python-powered wrapper for **Rootless Podman** designed to provide isolated, project-specific development environments based on **openSUSE Tumbleweed**.

This tool allows you to spin up a sandboxed environment for your code in seconds, ensuring that your host system remains clean while your development tools (git, compilers, and zypper ecosystem) stay updated.

## 🚀 Key Features

* **XDG Compliance:** Respects `XDG_CONFIG_HOME` and `XDG_DATA_HOME`, falling back to `~/.config` and `~/.local/share` if unset.
* **Hybrid Persistence:** 
    * **Shared Auth:** LLM keys and global settings are shared across all sandboxes via `${XDG_CONFIG_HOME}/opencode-sandbox`.
    * **Isolated Workspace Data:** Project history, indexing, and caches are isolated per project in `${XDG_DATA_HOME}/opencode-sandbox/ws-<hash>`.
* **Template Resolution:** Flexible `Dockerfile.template` resolution (Highest to Lowest priority):
    1.  `${XDG_CONFIG_HOME}/opencode-sandbox/Dockerfile.template`
    2.  `config/Dockerfile.template` (Parent config directory)
* **Sidecar Configuration:** Define your environment needs (base image, extra packages) per project using a `.opencode-sandbox.json` file.
* **Symlink-Safe:** Designed to be installed in a tool directory and symlinked to your `~/bin` or `/usr/local/bin`.
* **Rootless Podman:** No daemon, no root privileges, and native file permission mapping using `--userns=keep-id`.

---

## 🛠️ Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/youruser/opencode-sandbox.git ~/workspace/opencode-sandbox
    ```

2.  **Make the script executable:**
    ```bash
    chmod +x ~/workspace/opencode-sandbox/scripts/opencode-sandbox
    ```

3.  **Symlink to your Path:**
    ```bash
    mkdir -p ~/bin
    ln -s ~/workspace/opencode-sandbox/scripts/opencode-sandbox ~/bin/opencode-sandbox
    ```

---

## 📂 Project Configuration

Add a `.opencode-sandbox.json` file to any project directory to customize your sandbox:

```json
{
  "base_image": "opensuse/tumbleweed:latest",
  "install": [
    "cmake",
    "ninja",
    "gcc-c++",
    "python3-pip"
  ],
  "mounts": [
    { "host": "~/.gitconfig", "container": "/home/developer/.gitconfig" },
    { "host": "~/.ssh", "container": "/home/developer/.ssh" }
  ]
}
```

---

## 📖 Usage

Run the sandbox from any directory:

```bash
opencode-sandbox
```

### Options:
* `--rebuild`: Force an image rebuild.
* `--root`: Run as root (default: developer user).
* `--update`: Check for and install OpenCode updates within the workspace.
* `--include-dir`: Include additional directory (HostPath:ContainerPath or just HostPath for auto-mount in /mnt).
* `--debug`: Show debug information (paths, config, generated Dockerfile).
* `--dry-run`: Show the podman command without executing it.

---

## 🐚 Custom Commands & Shell Access

By default, running `opencode-sandbox` starts an interactive **OpenCode** session. However, you can pass any command to the container to override this behavior.

### Run a specific OpenCode command:
To run a specific message directly (non-interactive):
```bash
opencode-sandbox opencode run "Summarize this project"
```

### Access a Bash shell:
If you need to explore the environment or run manual commands:
```bash
opencode-sandbox /bin/bash
```

### Mount extra directories via CLI:
```bash
opencode-sandbox --include-dir ~/projects/shared-libs:/mnt/libs
# Or auto-mounts to /mnt/my-data:
opencode-sandbox --include-dir ~/my-data
```

### Run as Root:
To perform system-wide changes or install packages:
```bash
opencode-sandbox --root zypper install -y htop
```

### Run other tools inside the sandbox:
```bash
opencode-sandbox zypper info cmake
opencode-sandbox git status
```

> **Note:** When passing custom commands, if you need the environment from `~/.bashrc` (like the `PATH` for OpenCode), it's recommended to run them through a login shell:
> `opencode-sandbox /bin/bash --login -c "your-command"`
