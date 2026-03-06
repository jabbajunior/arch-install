The purpose of this document is to explain how to reproduce this system.

# Pre-installation
- Downloaded Arch Linux ISO (`2026.02.01 x86_64`)
- Verified ISO using s SHA256 checksum (`GetFileHash` on Windows 11)
- Created a virtual machine in the Promxox homelab environment with the following configuration:
- Machine type: `q35`
- Firmware: `OVMF (UEFI)`
- Storage `64 GB` virtual disk
- CPU: `2` cores
- Memory: `4 GB` RAM
- Network: `vmbr0` bridge interface

I ran into an issue since I left `pre-enroll-keys` checked. This requires secure boot and does not work with arch linux since the iso is not digitally signed. Had to redo the VM creation to fix.

Took a snapshot here in case I need to later.

## *1.6* Verify the boot mode
Run `cat /sys/firmware/efi/fw_platform_size` to determine the boot mode
- returned `64`

Means my system is booted into 64-bit UEFI mode.
## *1.9* Partition the disks
Following the steps described in the Arch Wiki:
https://wiki.archlinux.org/title/EFI_system_partition#Create_the_partition

Launch `fdisk`
First run `fdisk` on the virtual disk:
`fdisk /dev/sda`

### ### Create the EFI System Partition
Create a **1 GB EFI boot partition**:
1. Press `n` to create a new partition
2. Select `p` for a primary partition
3. Press `Enter` to accept the default first sector
4. Enter `+1G` to allocate 1 GB

Then change the partition type:
1. Press `t` to change the partition type
2. Enter `uefi` to assign the EFI System Partition type
### Create the Swap Partition
Create a **4 GB swap partition** using the same process:
1. Press `n` to create a new partition
2. Accept the default first sector
3. Enter `+4G` for the partition size

Then change the type:
1. Press `t` to change the type
2. Enter `swap` to  assign the swap partition type
### Create the Root Partition
Create the root partition using the remaining disk space:
1. Press `n` to create a new partition
2. Accept the default first sector
3. Press `Enter` again to use the remaining space

Set the partition type to:
`linux`
### Final Partition Layout

| Device    | Size | Type             |
| --------- | ---- | ---------------- |
| /dev/sda1 | 1G   | EFI System       |
| /dev/sda2 | 4G   | Linux swap       |
| /dev/sda3 | 59G  | Linux filesystem |

## *1.10* Format the partitions
After creating the partitions, format them with the appropriate filesystems.
### Root Partition
Format the root partition as `ext4`:
	`mkfs.ext4 /dev/sda3`
- **ext4** is used for the root filesystem
### Swap Partition
Initialize the swap partition:
	`mkswap /dev/sda2`
- **swap** provides virtual memory if RAM gets exceeded
### EFI System Partition
Format the EFI partition as FAT32:
	`mkfs.fat -F 32 /dev/sda1`
- **FAT32** is required for the EFI system partition

## *1.11* Mount the filesystems
After formatting the partitions, mount them so the Arch installation can be written to the disk.
### Mount the Root Partition
Mount the root filesystem to `/mnt`:
	`mount /dev/sda3 /mnt`

### Mount the EFI System Partition
For a UEFI system, mount the EFI System Partition at `/boot` inside the root filesystem:
	`mount --mkdir /dev/sda1 /mnt/boot`
- The `--mkdir` flag creates the mount point if it does not already exist.
### Enable Swap
Activate the swap partition:
	`swapon /dev/sda2`

## *2.2* Installing essential packages
After mounting the filesystem, install essential packages
	`pacstrap -K /mnt base linux`
	-  We omit the firmware package since we are on a virtual machine