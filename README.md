# arch-hypr-dots
Dotfiles for Arch Linux Hyprland Installation

## Arch Installation Guide

### Download ISO image

Go to the download page of the official Arch Linux webpage and download the ISO image from one of the mirrors: https://archlinux.org/download/.

### Prepare an installation medium

Check this Arch Wiki article to prepare an installation medium, e.g. a USB flash drive or an optical disc: https://wiki.archlinux.org/title/installation_guide#Prepare_an_installation_medium

### Console font

```
$ setfont Lat2-Terminus16
```

### Partitioning

#### UEFI

Check the name of the hard disk:

```
fdisk -l
```

Use the name (in my case _nvme0n1_) to start the `fdisk` partitioning tool:

```
fdisk /dev/nvme0n1
```

#### UEFI with GPT

Press <kbd>g</kbd> to create a new GUID Partition Table (GPT).

We will do it according to the example layout of the Arch wiki:

| Mount point | Partition                   | Partition type | Suggested size      |
| ----------- | --------------------------- | -------------- | ------------------- |
| /boot   | /dev/_efi_system_partition_ | uefi           | At least 1 GiB    |
| [SWAP]      | /dev/_swap_partition_       | swap           | At least 4 GiB   |
| /        | /dev/_root_partition_       | linux          | Remainder of device |

##### Create boot partition

1. Press <kbd>n</kbd>.
1. Press <kbd>Enter</kbd> to select the default partition number.
1. Press <kbd>Enter</kbd> to use the default first sector.
1. Enter _+1G_ for the last sector.
1. Press <kbd>t</kbd> and choose 1 and write _uefi_.

##### Create swap partition

1. Press <kbd>n</kbd>.
1. Press <kbd>Enter</kbd> to select the default partition number.
1. Press <kbd>Enter</kbd> to use the default first sector.
1. Enter _+4G_ for the last sector.
1. Press <kbd>t</kbd> and choose 2 and write _swap_.

##### Create root partition

1. Press <kbd>n</kbd>.
1. Press <kbd>Enter</kbd> to select the default partition number.
1. Press <kbd>Enter</kbd> to use the default first sector.
1. Enter <kbd>Enter</kbd> to use the default last sector.
1. Press <kbd>t</kbd> and choose 3 and write _linux_.

⚠️\ **When you are done partitioning don't forget to press <kbd>w</kbd> to save the changes!**

After partitioning check if the partitions have been created using `fdisk -l`.

##### Partition formatting

```
$ mkfs.ext4 /dev/<root_partition>
$ mkswap /dev/<swap_partition>
$ mkfs.fat -F 32 /dev/<efi_system_partition>
```

##### Mounting the file system

```
$ mount /dev/<root_partition> /mnt
$ mount --mkdir /dev/<efi_system_partition> /mnt/boot
$ swapon /dev/<swap_partition>
```

# Follow Arch Install Wiki

## System-related Configurations

### Enable network connection

To use _pacman_ you first have to have a working internet connection by enabling _NetworkManager_:

```
$ systemctl start dhcpcd
$ systemctl enable dhcpcd
```

### Update the system

First things first: Update the system!

```
$ pacman -Syu
```

### `sudo` Command

Install the `sudo` command:

```
$ pacman -S sudo
```

### Add your personal user account

```
$ useradd -m -g users -G wheel,storage,power,video,audio,input <your username>
$ passwd <your username>
```

#### Grant root access to our user

```
$ EDITOR=nano visudo
```

Uncomment the following line in order to use the `sudo` command without password prompt:

```
%wheel ALL=(ALL) NOPASSWD: ALL
```

You can then log in as your newly created user:

```
$ su <your username>
```

If you wish to have the default XDG directories (like Downloads, Pictures, Documents etc.) do:

```
$ sudo pacman -S xdg-user-dirs
$ xdg-user-dirs-update
```

### Install AUR package manager

To install [yay](https://github.com/Jguer/yay):

```
$ cd $HOME && mkdir aur
$ cd aur
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
```

### Sound

```
$ sudo pacman -S alsa-utils alsa-plugins
$ sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber pavucontrol
```

#### Enable and Starting Services

```shell
systemtcl –user enable –now pipewire.service
systemtcl –user enable –now pipewire-pulse.service
systemtcl –user enable –now wireplumber.service
```
Configuration files for PipeWire and related services are located in `~/.config/pipewire`

### Bluetooth

```
$ sudo pacman -S bluez bluez-utils blueman
$ sudo systemctl enable bluetooth
```

### Pacman

To beautify Pacman use:

```
$ sudo nano /etc/pacman.conf
```

Uncomment `Color` and add below it `ILoveCandy`.

ℹ️ If you have a good internet connection, you can uncomment the option `ParallelDownloads = 5`.

### Enable SSD Trim

```
$ sudo systemctl enable fstrim.timer
```

### Enable Time Synchronization

```
$ sudo pacman -S ntp
```

```
$ sudo systemctl enable ntpd
```

Then enable NTP:

```
$ timedatectl set-ntp true
```

## Graphical User Interface (GUI) Settings

### Wayland

```
$ sudo pacman -S hyprland hyprpaper swayidle
```

```
$ yay -S wlogout swaylock-effects-git
```

- _hyprland_: A compositor for Wayland
- _hyprpaper_: Set wallpaper in Hyprland
- _swayidle_: DPMS, turning screen off after timeout period

- _wlogout_: Menu for logging out, rebooting, shutting down, etc
- _swaylock-effects-git_: Lockscreen

⚠️ _Caution:_ If you don't have an NVIDIA graphics card you have to delete the environment variables concerning NVIDIA in _~/.config/hyprland/hyprland.conf_ later when configuring the system!

### Drivers

**AMD**:

```
sudo pacman -S xf86-video-amdgpu mesa vulkan-radeon lib32-mesa lib32-vulkan-radeon
```

### Fonts

```
$ sudo pacman -S noto-fonts ttf-opensans ttf-firacode-nerd
```

Emojis:

```
$ sudo pacman -S noto-fonts-emoji
```

To support Asian letters:

```
$ sudo pacman -S noto-fonts-cjk
```

### Shell

```
$ sudo pacman -S zsh
```

Change default shell to zsh:

```
$ chsh -s $(which zsh)
```

### Terminal

```
$ sudo pacman -S alacritty kitty
```

### Program Launcher

```
$ sudo pacman -S wofi
```

### Status Bar

```
$ sudo pacman -S waybar
```

### File Manager

```
$ sudo pacman -S thunar
```

### Browser

```
$ sudo pacman -S vivaldi
```

### Screenshot

```
$ yay -S hyprshot
```

### Screen Recorder

```
$ yay -S obs-studio-git
```

You have to install additional packages. Please follow these instructions: https://gist.github.com/PowerBall253/2dea6ddf6974ba4e5d26c3139ffb7580

### Media Player

```
$ sudo pacman -S vlc
```

### PDF Viewer

```
$ sudo pacman -S zathura zathura-pdf-mupdf
```

### GTK Dark Theme

To make GTK applications (e.g. _nemo_) use dark theme, execute the following commands:

```
$ gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita-dark'
$ gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
```

### Other Tools

#### CLI utilities

```
$ sudo pacman -S tldr fzf wget curl tar unzip gzip bahtop neofetch
```

- _tldr_: Commands cheat sheet
- _fzf_: Fuzzy finder
- _wget_: Fetching packages from the web
- _curl_: Fetching packages from the web
- _tar_: Enzipping/Unzipping
- _unzip_: Enzipping/Unzipping
- _gzip_: Enzipping/Unzipping
- _bashtop_: CLI task manager
- _neofetch_: System information

### Reboot

When done installing the necessary packages, run the `sudo reboot` command.

### Steam
To install Steam on Arch Linux, you can use the package from the multilib repository. Steam is a popular platform for digital distribution of games and includes support for managing your games, online play, and community features. Here’s how to install Steam on Arch Linux:

#### 1. Enable Multilib Repository
First, you need to enable the multilib repository if you haven't done so already. The multilib repository contains software and libraries that are compiled for 32-bit architecture, which Steam requires.

- Open the pacman configuration file:
  ```bash
  sudo nano /etc/pacman.conf
  ```

- Find the lines for `[multilib]` in the file. Uncomment both the `[multilib]` line and the line directly below it (`Include = /etc/pacman.d/mirrorlist`).

- Save and exit the editor (in nano, press `Ctrl+O` to save and `Ctrl+X` to exit).

- Update your package database:
  ```bash
  sudo pacman -Sy
  ```

#### 2. Install Steam
Now that the multilib repository is enabled, you can install Steam:

```bash
sudo pacman -S steam
```

This command installs the Steam client and all required 32-bit libraries.

#### 3. Running Steam
- You can start Steam using your desktop environment's menu or by running `steam` in the terminal.
- On the first run, Steam will update itself to the latest version, which may take some time depending on your internet connection.

### 4. Protonup-qt

```bash
yay -S protonup-qt
```
## Installing Hyprland

### Prerequisites

```bash
sudo pacman -S wlroots cmake wayland xorg-xwayland
```

```bash
yay -S hyprland-git
```

### Configuration
Hyprland uses a configuration file located at ~/.config/hypr/hyprland.conf. If the file doesn't exist, you can create it and configure it according to your needs. Sample configuration files and a lot of documentation can be found on the Hyprland GitHub page.

### Running Hyprland
After installation, you can start Hyprland directly from a TTY or add an entry for it in your display manager. To start it from a TTY:

Logout of your current session and go to a TTY (Ctrl+Alt+F2, for example).
Log in and run:
```bash
export XDG_SESSION_TYPE=wayland
export XDG_CURRENT_DESKTOP=hyprland
hyprland
```
If you're using a display manager like SDDM or GDM, you may need to create a desktop entry for Hyprland. Create a file named `hyprland.desktop` in `/usr/share/wayland-sessions/` with the following contents:

```
[Desktop Entry]
Name=Hyprland
Comment=This session logs you into Hyprland
Exec=hyprland
Type=Application
```
