
# Table of contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Table of contents](#table-of-contents)
- [Foreword](#foreword)
- [Installation](#installation)
  - [Install RPMfusion](#install-rpmfusion)
  - [Install Flathub (Flatpak support for Gnome Software)](#install-flathub-flatpak-support-for-gnome-software)
- [Tips and tricks](#tips-and-tricks)
  - [dnf package manager](#dnf-package-manager)
    - [History](#history)
    - [Comparison overview](#comparison-overview)
    - [Pimp my dnf](#pimp-my-dnf)
    - [Remove old kernels](#remove-old-kernels)
  - [Bluetooth](#bluetooth)
    - [AirPods](#airpods)
    - [TLP vs. Bluetooth](#tlp-vs-bluetooth)
    - [Notebooks using single module for Bluetooth and 802.11b wifi](#notebooks-using-single-module-for-bluetooth-and-80211b-wifi)
  - [Graphics](#graphics)
    - [Nvidia - Render gnome-shell above 60Hz](#nvidia-render-gnome-shell-above-60hz)
  - [CMake](#cmake)
    - [Most used libraries](#most-used-libraries)

<!-- /code_chunk_output -->

# Foreword

The following tips apply to a Fedora installation but will apply to other distributions as well.

Most issues and their solutions have been tested on a T440p, T495 and desktop PC using Nvidia graphics.

Last updated: Fedora 31

# Installation

## Install RPMfusion

RPMfusion provides helpful packages that do not meet the criteria of the official repositories (like open, but not correctly licensed codecs and propritery software)

https://rpmfusion.org/Configuration

Install free and nonfree via Gnome Software or dnf.

## Install Flathub (Flatpak support for Gnome Software)

https://flatpak.org/setup/Fedora/

Install via Gnome Software.

Perform flatpak update and refresh Gnome Software afterwards.

# Tips and tricks

## dnf package manager

### History

yum was the previously supported package manager. The command is still available for legacy reasons. **Use dnf**.

### Comparison overview

https://wiki.archlinux.org/index.php/Pacman/Rosetta

### Pimp my dnf

Set/add parameters to /etc/dnf/dnf.conf

|Parameter   |Option   | Explanation |
|---|---|---|
|installonly_limit   | 2  | dnf will keep the last 3 kernels by default. Reduces the number to 2.  |
|fastestmirror   | True  | Saves a new mirrorlist based on a mirror speed test all 24 hours to determine the fastest mirror.  |
|deltarpm   | True  | Supported packages will use delta packages instead of downloading the complete package. Hint: Trades bandwidth for processing power. |

### Remove old kernels

```c
sudo dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q)
```

## Bluetooth

### AirPods

pulseaudio-bluetooth does not support the proprietary LE feature of Apple AirPods. The feature has to be disabled to allow pairing and successful usage.

Limit the ControllerMode of your bluetooth module to bredr. Uncomment ControllerMode and set to bredr.

```c
/etc/bluetooth/main.conf
ControllerMode = bredr
```

Restart bluetooth service or reboot afterwards.

### TLP vs. Bluetooth

If you're experiencing cutoffs or unexpected lose of connection configure TLP to skip suspend management for Bluetooth devices.

```c
/etc/default/tlp
USB_BLACKLIST_BTUSB=1
```

### Notebooks using single module for Bluetooth and 802.11b wifi

If you're facing cutoffs of the bluetooth connection while using wifi try:

```c
/etc/modprobe.d/iwlwifi.conf
options iwlwifi bt_coex_active=0
```

Modprobe or reboot afterwards.

The coexistence feature will be disabled and the wifi performance severly limited (tested on T495). If this setting solves the bluetooth cutoffs consider switching to 5Ghz wifi.

## Graphics

### Nvidia - Render gnome-shell above 60Hz

```c
/etc/environment
__GL_SYNC_DISPLAY_DEVICE=DP-x
```

Replace DP-x with your Display device.

The setting will only apply after starting nvidia-settings. Consider adding it to startup applications.

## CMake

### Most used libraries

```c
find_package(GLEW REQUIRED)
include_directories(${GLEW_INCLUDE_DIRS})
link_libraries(${GLEW_LIBRARIES})

find_package(GLUT REQUIRED)
include_directories(${GLUT_INCLUDE_DIRS})
link_libraries(${GLUT_LIBRARIES})

find_package(glfw3 REQUIRED)
include_directories(${glfw3_INCLUDE_DIRS})
link_libraries(glfw)

find_package(OpenGL REQUIRED)
link_libraries(${OPENGL_LIBRARIES})

find_package(GTest REQUIRED)
include_directories(${GTest_INCLUDE_DIRS})
link_libraries(gtest)
link_libraries(gmock)
```
