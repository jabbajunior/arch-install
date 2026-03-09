# 3/6/2026
Worked on sections **1.9 – 2.2** of the Arch Linux installation process.
Tasks completed:
- **Partitioned the disk** using `fdisk` and created:
    - EFI System Partition
    - Swap partition
    - Root partition
- **Formatted the partitions**
    - `ext4` for the root filesystem
    - `FAT32` for the EFI partition
    - Initialized the swap partition
- **Mounted the filesystems**
    - Root partition mounted at `/mnt`
    - EFI partition mounted at `/mnt/boot`
    - Swap enabled with `swapon`
- **Installed the base system**
    - Used `pacstrap` to install `base` and `linux` packages

# 3/8/2026
Worked on sections **3.1 – 3.8** of the Arch Linux installation process.
Tasks completed:
- **Generated the filesystem table**
    - Created `/etc/fstab` using `genfstab`
    - Configured mounts using **UUIDs** for reliability
- **Entered the installed system**
    - Used `arch-chroot` to switch into the new system environment for configuration
- **Configured system time**
    - Set the timezone to `America/New_York`
    - Synchronized the hardware clock with `hwclock`
    - Configured **systemd-timesyncd** for NTP synchronization
    - Enabled automatic time synchronization with `timedatectl`
- **Configured system localization**
    - Generated the `en_US.UTF-8` locale
    - Set the system language in `/etc/locale.conf`
    - Configured the console keyboard layout in `/etc/vconsole.conf`
- **Configured system networking**
    - Set the system hostname via `/etc/hostname`
- **Secured the root account**
    - Set the root password using `passwd`
- **Installed and configured the bootloader**
    - Installed **systemd-boot** using `bootctl`
    - Configured `/boot/loader/loader.conf`
    - Created a boot entry in `/boot/loader/entries/arch.conf`
    - Verified the bootloader installation with `bootctl`
- **Logged into the installed system**
    - Restarted the virtual machine after completing the installation process
    - Verified that the **systemd-boot bootloader** loaded correctly
    - Successfully logged into the system using the **root account**