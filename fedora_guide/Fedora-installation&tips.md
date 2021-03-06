
# Table of contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Table of contents](#table-of-contents)
- [Foreword](#foreword)
- [Installation](#installation)
  - [Install RPMfusion](#install-rpmfusion)
  - [Install Flathub (Flatpak support for Gnome Software)](#install-flathub-flatpak-support-for-gnome-software)
  - [TLP](#tlp)
  - [Known issues](#known-issues)
    - [Unexpected long boot times](#unexpected-long-boot-times)
      - [1.) Disable plymouth](#1-disable-plymouth)
      - [2.) Disable NetworkManager waiting for connection](#2-disable-networkmanager-waiting-for-connection)
- [Tips and tricks](#tips-and-tricks)
  - [dnf package manager](#dnf-package-manager)
    - [History](#history)
    - [Comparison overview](#comparison-overview)
    - [Pimp my dnf](#pimp-my-dnf)
    - [Remove old kernels](#remove-old-kernels)
  - [Multimedia](#multimedia)
    - [Minimal configuration](#minimal-configuration)
    - [Complete configuration](#complete-configuration)
    - [Nautilus: Thumbnails for video files](#nautilus-thumbnails-for-video-files)
  - [Modern shell: Fish](#modern-shell-fish)
    - [Installation](#installation-1)
    - [Changing your default shell](#changing-your-default-shell)
    - [Configuration](#configuration)
  - [Bluetooth](#bluetooth)
    - [AirPods](#airpods)
    - [Sony LDAC, aptX, aptX HD, AAC support](#sony-ldac-aptx-aptx-hd-aac-support)
    - [TLP vs. Bluetooth](#tlp-vs-bluetooth)
    - [Notebooks using single module for Bluetooth and 802.11b wifi](#notebooks-using-single-module-for-bluetooth-and-80211b-wifi)
  - [Graphics](#graphics)
    - [Nvidia - Render gnome-shell above 60Hz](#nvidia-render-gnome-shell-above-60hz)
  - [Filesystems](#filesystems)
    - [Exfat support](#exfat-support)
  - [Font configuration](#font-configuration)
    - [Better font rendering](#better-font-rendering)
    - [E-Books (epub, mobi, ...)](#e-books-epub-mobi)
      - [Optional: Font recommendation](#optional-font-recommendation)
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

## TLP

Fedora does not come with TLP pre-installed. Fedora claims that the onboard power-tools make TLP obsolete for most use cases (citation needed). This could not be reproduced on a T440p and T495.

```sh
dnf install tlp
systemctl mask systemd-rfkill.socket
systemctl reboot
```

The defaults of TLP can be kept as is for most Thinkpad notebooks.
Exception see [TLP vs. Bluetooth](#tlp-vs-bluetooth).

## Known issues

### Unexpected long boot times

Fedora (confirmed on F31) is affected by a bug in plymouth (the animated boot screen) which extends the boot time by up to 50%. The blocking service `plymouth-quit-wait.service` will wait longer than necessary and keep the graphical target from showing.

Note your boot times before and after applying the proposal using:

```sh
systemd-analyze time

# Optional:
systemd-analyze blame
systemd-analyze plot > boot.svg
```

#### 1.) Disable plymouth

Add `rd.plymouth=0 plymouth.enable=0` to your boot parameters.

```sh
/etc/default/grub
-----------------
GRUB_CMDLINE_LINUX="[...] rhgb quiet rd.plymouth=0 plymouth.enable=0"
```

Regenerate your grub configuration.

```sh
# For EFI systems
grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
# For BIOS systems
grub2-mkconfig -o /boot/grub2/grub.cfg
```

#### 2.) Disable NetworkManager waiting for connection

Check `systemd-analyze blame` for `NetworkManager-wait-online.service`. If it's taking up more than a few seconds consider the following:

The service is intended for applications which rely on a web connection on bootup. Disabling the wait does not affect most desktop users.

```sh
systemctl disable NetworkManager-wait-online.service
```

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

```sh
dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q)
```

## Multimedia

### Minimal configuration

Requirement: RPMFusion enabled.

```sh
dnf install ffmpeg-libs
```

### Complete configuration

Requirement: RPMFusion enabled.

```sh
dnf groupupdate Multimedia
```

### Nautilus: Thumbnails for video files

Requirement: RPMFusion enabled.

```sh
dnf install gstreamer1-libav
rm -rf ~/.cache/thumbnails/fail
nautilus -q
```

## Modern shell: Fish

### Installation

```sh
dnf install fish
```

### Changing your default shell

```sh
chsh -s /bin/fish
```

Hint: Most terminal emulators require a complete restart of the terminal emulator for the new default shell to take effect. If in doubt do a reboot.

### Configuration

Fish is configured using an web interface.

```sh
fish_config
```

A http server will be started and the default browser should open a page pointing to the configuration website. Remember to hit save on every page to persist your changes.

## Bluetooth

### AirPods

pulseaudio-bluetooth does not support the proprietary LE feature of Apple AirPods. The feature has to be disabled to allow pairing and successful usage.

Limit the ControllerMode of your bluetooth module to bredr. Uncomment ControllerMode and set to bredr.

```sh
/etc/bluetooth/main.conf
------------------------
ControllerMode = bredr
```

Restart bluetooth service or reboot afterwards.

### Sony LDAC, aptX, aptX HD, AAC support

Requirement: RPMFusion enabled.

```sh
dnf install pulseaudio-module-bluetooth-freeworld
```

Restart pulseaudio or reboot afterwards.

### TLP vs. Bluetooth

If you're experiencing cutoffs or unexpected lose of connection configure TLP to skip suspend management for Bluetooth devices.

```sh
/etc/default/tlp
----------------
USB_BLACKLIST_BTUSB=1
```

### Notebooks using single module for Bluetooth and 802.11b wifi

If you're experiencing cutoffs of the bluetooth connection while using wifi try:

```sh
/etc/modprobe.d/iwlwifi.conf
----------------------------
options iwlwifi bt_coex_active=0
```

Modprobe or reboot afterwards.

The coexistence feature will be disabled and the wifi performance severly limited (tested on T495). If this setting solves the bluetooth cutoffs consider switching to 5Ghz wifi. Do not forget to undo the modification.

## Graphics

### Nvidia - Render gnome-shell above 60Hz

```sh
/etc/environment
----------------
__GL_SYNC_DISPLAY_DEVICE=DP-x
```

Replace DP-x with your Display device.

The setting will only apply after starting nvidia-settings. Consider adding it to startup applications.

## Filesystems

### Exfat support

```sh
dnf install fuse-exfat
```

## Font configuration

### Better font rendering

TODO

### E-Books (epub, mobi, ...)

- Install foliate

```sh
dnf install foliate
```

#### Optional: Font recommendation

- Download Bitter OTF font from **[Bitter OTF](https://www.huertatipografica.com/en/fonts/bitter-ht)**

- Open & install each .otf seperatly using Gnome Font Viewer

- Configure foliate to use Bitter. Recommended font size: 18
![foliate-font-config](images/foliate_font_config.png)

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
