# 🍴 Fork: WinApps-Launcher (Enhanced)
This is a **fork** of the original [WinApps-Launcher](https://github.com/winapps-org/winapps) project, now maintained by `sundaysfantasy`.

The primary goal of this fork is to add **integrated application management features** and ensure persistent configuration for user-defined applications, preventing data loss during WinApps reinstalls.

![WinApps Launcher Demonstration](demo.gif)

## New Features ✨: Integrated Application Creation

This fork introduces a unified **`Create New Application`** function directly within the launcher's menu. This feature guides the user through creating a new application entry and automatically registers it with your system.

### How it Works

The function automates the creation of all necessary files across four distinct directories to ensure persistence, correct launching from the `winapps-launcher` menu, and proper desktop integration:

1.  **Persistent Data (`~/.local/bin/winapps-src/apps/<app-name>`):** Creates the master application directory containing the essential `info` file and the icon. This is the **persistent source** that survives WinApps reinstallations.
2.  **WinApps Directory Copy (`~/.local/share/winapps/apps/<app-name>`):** Creates a physical copy of the application directory in the location WinApps expects. This ensures the main `winapps` script can find the application data and launch it correctly.
3.  **Launcher Script (`~/.local/bin/<app-name>`):** Creates a dedicated Bash script used by the `winapps-launcher.sh` menu to launch the application using the `winapps explore --app <app-name>` syntax.
4.  **Desktop Entry (`~/.local/share/applications/<app-name>.desktop`):** Generates a compliant `.desktop` file, allowing the application to appear in your application menu and launch directly with argument support (`winapps <app-name> %F`).

---


# WinApps-Launcher
Taskbar Launcher for [WinApps](https://github.com/winapps-org/winapps).
Feel free to fork, submit pull requests, open issues and promote this project to grow the WinApps community!

![WinApps Launcher Demonstration](demo.gif)

## Dependencies
You should already have most required dependencies after installing WinApps, but `yad` may be missing.

### Debian / Ubuntu / Linux Mint / Zorin OS
```bash
sudo apt install yad
```

### Fedora / RHEL
```bash
sudo dnf install yad
```

### Arch
```bash
sudo pacman -Syu --needed yad
```

### openSUSE
```bash
sudo zypper install yad
```

> [!IMPORTANT]
> [GNOME no longer shows tray icons by default](https://blogs.gnome.org/aday/2017/08/31/status-icons-and-gnome/). To use WinApps Launcher with GNOME, you must install the [AppIndicator and KStatusNotifierItem Support](https://extensions.gnome.org/extension/615/appindicator-support/) shell extension.

## Installation
1.  Ensure you have already installed [`WinApps`](https://github.com/winapps-org/winapps).

2.  Identify your WinApps source directory.
    *   For a local user installation: `~/.local/bin/winapps-src`
    *   For a system-wide installation: `/usr/local/bin/winapps-src`

3.  Set a variable in your terminal pointing to this directory. **Copy the line that matches your setup.**

    ```bash
    # FOR LOCAL INSTALL:
    WINAPPS_SRC_DIR="$HOME/.local/bin/winapps-src"

    # --- OR ---

    # FOR SYSTEM-WIDE INSTALL:
    WINAPPS_SRC_DIR="/usr/local/bin/winapps-src"
    ```

4.  Clone this repository and run the launcher as a test. This will place `winapps-launcher` inside your existing `winapps-src` directory.

    ```bash
    # Clone the repository into the correct location
    git clone https://github.com/winapps-org/winapps-launcher.git "${WINAPPS_SRC_DIR}/winapps-launcher"

    # Make the script executable
    chmod +x "${WINAPPS_SRC_DIR}/winapps-launcher/winapps-launcher.sh"

    # Run the launcher as a test
    "${WINAPPS_SRC_DIR}/winapps-launcher/winapps-launcher.sh"
    ```

## Post-Installation
You can add an application menu icon for WinApps Launcher and/or configure a user service that starts WinApps Launcher automatically at login.

### Application Icon
Ensuring `WINAPPS_SRC_DIR` is still set correctly, run the following within your terminal to add a WinApps Launcher icon to your applications menu.

```bash
mkdir -p ~/.local/share/applications

cat > ~/.local/share/applications/winapps-launcher.desktop <<EOF
[Desktop Entry]
Type=Application
Name=WinApps Launcher
Comment=Taskbar Launcher for WinApps
Exec="$WINAPPS_SRC_DIR/winapps-launcher/winapps-launcher.sh"
Icon=$WINAPPS_SRC_DIR/winapps-launcher/icons/LinkIcon.svg
Terminal=false
Categories=Utility;
EOF
```

### Autostart
1. Ensuring `WINAPPS_SRC_DIR` is still set correctly, copy and paste the following code block into your terminal to create the systemd user service file.
    ```bash
    mkdir -p ~/.config/systemd/user

    cat > ~/.config/systemd/user/winapps-launcher.service <<EOF
    [Unit]
    Description=Run 'WinApps Launcher'
    After=graphical-session.target default.target
    Wants=graphical-session.target

    [Service]
    Type=simple
    Environment="PATH=%h/.local/bin:%h/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    Environment="LIBVIRT_DEFAULT_URI=qemu:///system"
    Environment="SCRIPT_PATH=$WINAPPS_SRC_DIR/winapps-launcher/winapps-launcher.sh"
    Environment="LANG=C"
    ExecStart=/bin/bash -c "\\"\$SCRIPT_PATH\\""
    ExecStopPost=/bin/bash -c 'echo "[SYSTEMD] WINAPPS LAUNCHER SERVICE EXITED."'
    TimeoutStartSec=5
    TimeoutStopSec=5
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=default.target
    EOF
    ```

2. Enable the user service.
    ```bash
    systemctl --user enable winapps-launcher --now # Enable & Start Service
    systemctl --user status winapps-launcher # Verify
    ```

> [!NOTE]
> To uninstall the WinApps Launcher user service, run the following:
> ```bash
> systemctl --user stop winapps-launcher # Stop Service
> systemctl --user disable winapps-launcher # Disable Service
> rm ~/.config/systemd/user/winapps-launcher.service # Delete Service
> ```
