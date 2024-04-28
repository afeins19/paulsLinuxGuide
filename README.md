# arch-hypr-dots
Dotfiles for Arch Linux Hyprland Installation

## Arch Installation Guide

### Download ISO image

Go to the download page of the official Arch Linux webpage and download the ISO image from one of the mirrors: https://archlinux.org/download/.

### Prepare an installation medium

Check this Arch Wiki article to prepare an installation medium, e.g. a USB flash drive or an optical disc: https://wiki.archlinux.org/title/installation_guide#Prepare_an_installation_medium

### Connect to the web [WiFi]

To connect to the web we use _iwctl_:

To start _iwctl_ run the following command:

```
$ iwctl
```

We can then look for the device name:

```
[iwd]# device list
```

Then we can use the device name to scan for networks _(Note: This command won't output anything!)_:

```
[iwd]# station <device-name> scan
```

Now we can list all available networks:

```
[iwd]# station <device-name> get-networks
```

Finally, we connect to a network:

```
[iwd]# station <device-name> connect <SSID>
```

Check if you successfully established a connection by pinging the Google server:

```
$ ping 8.8.8.8
```

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

### Package install

For a minimal system download and install these packages:

```
$ pacstrap -K /mnt base base-devel linux linux-firmware e2fsprogs networkmanager git nano man-db man-pages texinfo grub pacman git tar gzip unzip bzip2 xz efibootmgr
```

⚠️ If you get errors due to key then do the following:

1. Initialize _pacman_ keys and populate them:

```
pacman-key --init
pacman-key --populate
```

2. Synchronize Arch keyring:

```
archlinux-keyring-wkd-sync
```

### Last steps

#### Generate fstab file

```
$ genfstab -U /mnt >> /mnt/etc/fstab
```

#### Change root into new system

```
$ arch-chroot /mnt
```

#### Set time zone

```
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ hwclock --systohc
```

#### Localization

Edit _/etc/locale.gen_ and uncomment _en_US.UTF-8 UTF-8_ and other needed locales. Generate the locales by running:

```
$ locale-gen
```

Create _/etc/locale.conf_ and set the _LANG_ variable according to your preferred language:

```
LANG=en_US.UTF-8
```

#### Network configurations

Create _/etc/hostname_ and type any name you wish as your hostname:

```
arch
```

#### Root password

Set a new password for root:

```
$ passwd
```

#### Bootloader

##### UEFI

Run the following command:

```
$ grub-install --efi-directory=/boot --bootloader-id=GRUB
```

Then create a **GRUB** config file:

```
$ grub-mkconfig -o /boot/grub/grub.cfg
```

#### Final step

Exit out of the chroot environment by typing `exit` or pressing <kbd>Ctrl</kbd>+<kbd>d</kbd>.

Unmount all the partitions:

```
$ umount -R /mnt
```

Then type `poweroff` and remove the installation disk.

## System-related Configurations

### Enable network connection

To use _pacman_ you first have to have a working internet connection by enabling _NetworkManager_:

```
$ systemctl start NetworkManager
$ systemctl enable NetworkManager
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

#### Programming Languages

These languages are needed for _Mason_, the LSP package manager in _Neovim_:

```
$ sudo pacman -S nodejs npm rust go ruby rubygems php composer lua luarocks python python-pip dotnet-runtime dotnet-sdk julia java-runtime-common java-environment-common jdk-openjdk
```

#### CLI utilities

```
$ sudo pacman -S tldr fzf wget curl tar unzip gzip htop neofetch
```

```
$ yay -S pfetch
```

- _tldr_: Commands cheat sheet
- _fzf_: Fuzzy finder
- _wget_: Fetching packages from the web
- _curl_: Fetching packages from the web
- _tar_: Enzipping/Unzipping
- _unzip_: Enzipping/Unzipping
- _gzip_: Enzipping/Unzipping
- _htop_: CLI task manager
- _neofetch_: System information
- _pfetch_: More concise system information

#### Alternatives to traditional commands

```
$ sudo pacman -S fd ripgrep bat lsd tree-sitter tree-sitter-cli
```

- _fd_: Alternative to _find_ command
- _ripgrep_: Alternative to _grep_ command
- _bat_: Alternative to _cat_ command
- _lsd_: Alternative to _ls_ command
- _tree-sitter_ & _tree-sitter-cli_: Real syntax highlighting in Neovim

### Reboot

When done installing the necessary packages, run the `sudo reboot` command.
