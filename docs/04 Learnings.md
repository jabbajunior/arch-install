This document captures key concepts and lessons learned during the Arch Linux installation process.

---

## UEFI Boot and the EFI System Partition (ESP)
When using **UEFI firmware** for booting, the system requires an **EFI System Partition (ESP)**.

The ESP stores the bootloader and related boot files used by the firmware to start the operating system.

Reference:  
[https://wiki.archlinux.org/title/EFI_system_partition](https://wiki.archlinux.org/title/EFI_system_partition)

In this installation:
- The ESP was created during disk partitioning (Section **1.9**)
- It is mounted at `/boot`
- The ESP corresponds to the partition:
	`/dev/sda1`

This partition contains the **systemd-boot bootloader** and related boot configuration files.

---

## Partition Management Tools
During the installation process, several command-line tools are used to inspect and manage disks.
### lsblk
`lsblk` displays information about available block devices.

Example uses:
- View disk layout
- Confirm partitions
- Verify mount points

This command was already familiar prior to the installation.
### fdisk
`fdisk` is used to **create and modify disk partitions**.

In this installation it was used to:
- Create the **EFI System Partition**
- Create the **swap partition**
- Create the **root partition**
    

Unlike `lsblk`, `fdisk` modifies the partition table directly.

---
## Filesystem Table (`fstab`)
The file `/etc/fstab` defines which filesystems should be mounted automatically during system startup.

Without proper `fstab` entries:
- Partitions would not mount automatically
- Important directories such as `/` or `/boot` would not be available after boot


The `genfstab` command was used to generate this configuration:
	`genfstab -U /mnt >> /mnt/etc/fstab`

Using the `-U` flag ensures partitions are referenced by **UUID**, which is more reliable than device names.

---
## Root Password Requirement
A **root password must be set** in order to log into the system as the root user.

If a root password is not configured during installation, login will fail. I personally did not set one during the install and had to follow the following recovery process:

### Recovery Process
To resolve this issue:

1. Booted into the **Arch installation ISO**
2. Mounted the system partitions
    - Root partition
    - Boot partition
3. Entered the system using `arch-chroot`
4. Set the root password using:
	`passwd`
5. Exited the chroot environment
6. Rebooted the system
    
After completing these steps, it was possible to successfully log into the root account.