---
title: "Installing KDE Plasma Workspaces on Rocky Linux 8: A Step-by-Step Guide"
date: 2026-09-17
categories:
  - Linux
tags:
  - kde-plasma
  - rocky-linux
  - desktop-environment
  - sddm
  - dnf
---

# Installing KDE Plasma Workspaces on Rocky Linux 8

Rocky Linux ships as a minimal, server-oriented distro by default, which is great for stability but leaves you without a desktop environment out of the box. If you're running Rocky Linux 8 and want a full-featured, polished desktop, KDE Plasma is one of the best options available — and installing it is more straightforward than you might expect.

This guide walks through the complete process, from prerequisite repositories to your first login on the Plasma desktop.

## What You'll Need

Before starting, make sure your setup checks these boxes:

- Rocky Linux 8 (this guide was tested on 8.10 "Green Obsidian", x86_64)
- Active internet connectivity
- At least 3 GB of free disk space for Plasma components
- Root or sudo access
- Access to the BaseOS, AppStream, Extras, and PowerTools repositories

This walkthrough was done on a VMware virtual machine, but the same steps apply to bare metal installs.

## Step 1: Install the EPEL Repository

KDE Plasma and its dependencies pull in packages that live outside Rocky's default repos, so the first step is enabling EPEL (Extra Packages for Enterprise Linux):

```bash
dnf install epel-release
```

This pulls in `epel-release-8-22.el8` from the extras repository and unlocks a much wider package set for the steps that follow.

## Step 2: Enable the PowerTools Repository

KDE Plasma also needs some development and build tooling that lives in the PowerTools repo, which is disabled by default:

```bash
dnf config-manager --set-enabled powertools
```

With PowerTools enabled, you're ready for the main event.

## Step 3: Install KDE Plasma Workspaces and Base-X

This is the big one — installing the Plasma desktop group along with the base X11 graphics stack:

```bash
sudo dnf groupinstall "KDE Plasma Workspaces" "base-x" -y
```

Don't be surprised by the scale of this operation. On a fresh minimal install, expect:

- **~1,025 new packages**
- **~74 package upgrades**
- **~2.0 GB** total download

Grab a coffee — this step takes a while, especially on a modest connection.

## Step 4: Enable the SDDM Display Manager

SDDM (Simple Desktop Display Manager) is Plasma's default login screen. Enable it so it starts automatically:

```bash
systemctl enable sddm
```

## Step 5: Set the Graphical Target as Default

By default, a minimal Rocky Linux install boots to a text console. Switch the default boot target to graphical mode so the system lands on the login screen automatically:

```bash
systemctl set-default graphical.target
```

## Logging In

After a reboot, you'll be greeted by the SDDM login screen. Enter your credentials — whether that's your regular user or root — and Plasma will load.

Once authenticated, the full KDE Plasma desktop appears, complete with the applications menu and all the bundled software ready to use.

## What's Included

The `KDE Plasma Workspaces` and `base-x` groups bring in a lot more than just a desktop shell:

**Core components**
- KDE Plasma Desktop 5.24.7
- KWin window manager (X11 and Wayland support)
- SDDM display manager
- Plasma Workspace and utilities

**Bundled applications**
- Dolphin (file manager)
- Kate (text editor)
- Konsole (terminal)
- System Settings
- KAddressBook, KMail, KOrganizer (PIM suite)
- Spectacle (screenshots)
- Okular (document viewer)
- GIMP-related tools

**System tools**
- NetworkManager and plugins
- PulseAudio
- Bluetooth support
- Printer management
- Font management utilities

**Graphics and drivers**
- Mesa drivers
- X.Org display server and drivers
- GTK theme integration
- Breeze icon theme

## Post-Installation Steps

A few things worth doing right after your first login:

**Update the system:**
```bash
dnf update
```

**Install common applications:**
```bash
dnf install firefox thunderbird vlc
```

**Set up your network** using the NetworkManager applet in the system tray.

**Add extra language input support**, if needed — for example, Chinese Pinyin input:
```bash
dnf install ibus ibus-libpinyin
```

## Troubleshooting Common Issues

**Desktop doesn't display properly:**
1. Check the SDDM logs: `journalctl -u sddm -n 50`
2. Verify your display drivers: `lspci | grep VGA`
3. Try forcing the X11 platform: `export QT_QPA_PLATFORM=xcb`

**System feels sluggish:**
1. Disable unnecessary visual effects under *System Settings > Startup and Shutdown > Splash Screen*
2. Check available disk space: `df -h`
3. Monitor resource usage with System Monitor

**No sound:**
1. Install additional codecs: `dnf install gstreamer1-plugins-ugly`
2. Configure audio under *System Settings > Multimedia*
3. Check mixer levels: `alsamixer`

## What to Expect, Resource-Wise

- **Disk usage:** roughly 3–4 GB total for Plasma and its components
- **Memory:** around 300–500 MB of RAM for basic desktop operation
- **Boot time:** 30–60 seconds to reach the desktop, depending on hardware

## Keeping It Updated

Standard `dnf upgrade` handles Plasma updates going forward — there's no special update mechanism to worry about:

```bash
dnf upgrade
```

## Wrapping Up

Five commands and a handful of minutes later, you've gone from a bare Rocky Linux 8 minimal install to a fully functional KDE Plasma desktop with a complete application suite. From here, Plasma's modular design means you can customize nearly everything — themes, widgets, window behavior — to fit exactly how you like to work.

For deeper dives, the [KDE Project site](https://kde.org), [Rocky Linux docs](https://docs.rockylinux.org), and [KDE Community forums](https://discuss.kde.org) are all solid next stops.
