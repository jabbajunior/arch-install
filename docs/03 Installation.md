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

## _1.6_ Verify the boot mode
Run `cat /sys/firmware/efi/fw_platform_size` to determine the boot mode
- returned `64`

Means my system is booted into 64-bit UEFI mode.
## _1.9_ Partition the disks
Following the steps described in the Arch Wiki:
https://wiki.archlinux.org/title/EFI_system_partition#Create_the_partition

Launch `fdisk`
First run `fdisk` on the virtual disk:
`fdisk /dev/sda`

### Create the EFI System Partition
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

## _1.10_ Format the partitions
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

## _1.11_ Mount the filesystems
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

# Installation
## _2.2_ Installing Essential Packages
After mounting the filesystem, install the base system packages using `pacstrap`:

`pacstrap -K /mnt base linux`
- `-K` initializes the keyring inside the new system
- The packages `base` and `linux` install the minimal Arch system and kernel
- The `linux-firmware` package is **not required** in this case since the installation is being performed inside a virtual machine
### Install a Text Editor
A text editor will be useful for editing configuration files during the installation process.  
Install **nano** with `pacman`:
`pacman -S nano`

## *3.1* Configuring `fstab`
To ensure the filesystems are mounted automatically at boot, generate an `fstab` file for the new system.

Run:
`genfstab -U /mnt >> /mnt/etc/fstab`
- `-U` instructs `genfstab` to use **UUIDs** instead of device names
- Using UUIDs ensures the correct partitions are mounted even if device names change
    
This command appends the generated configuration to `/mnt/etc/fstab`.
## _3.2_ Chroot into the New System
After generating the `fstab`, enter the installed system using `arch-chroot`.
	`arch-chroot /mnt`
	
- This changes the root directory to `/mnt`, allowing you to configure the newly installed Arch system as if it were booted normally.`

## _3.3_ Configuring Time
### Set the Timezone
Set the system timezone by creating a symbolic link to the appropriate zone file.
	`ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime`

Then synchronize the hardware clock with the system clock:
`hwclock --systohc`

---
### Configure Network Time Protocol (NTP)
To keep the system clock synchronized automatically, configure `systemd-timesyncd`.

First edit the configuration file:
	`nano /etc/systemd/timesyncd.conf`

Uncomment the following lines:
	NTP=  
	FallbackNTP=

Then configure the NTP servers:
	`NTP=0.us.pool.ntp.org 1.us.pool.ntp.org 2.us.pool.ntp.org 3.us.pool.ntp.org`
- These servers are part of the **NTP pool project**
- The `us.pool.ntp.org` servers are appropriate for my system
---
### Enable Time Synchronization
Enable automatic time synchronization:
	`timedatectl set-ntp true`

Verify that the service is running:
	`timedatectl timesync-status`

- This should display the synchronization status and the active NTP server.
## _3.4_ Localization
Localization settings define the system language and keyboard configuration.

### Generate Locales
First edit the locale generation file:
	`nano /etc/locale.gen`

Uncomment the following line:
	en_US.UTF-8 UTF-8

Then generate the locale files via:
	`locale-gen`

### Set the System Locale
Create or edit the locale configuration file:
	`nano /etc/locale.conf`

Uncomment the following line:
	`LANG=en_US.UTF-8`

- This sets the default system language.

### Configure the Console Keyboard Layout
Edit the virtual console configuration file:
	`nano /etc/vconsole.conf`

Add the following line:
	`KEYMAP=en`
	
This sets the keyboard layout for the Linux console.

## _3.5_ Network Configuration
Set the system hostname to identify the machine on the network.

Edit the hostname file:
	`nano /etc/hostname`

Add your desired hostname. I chose:
	`arch`

This name will be used to identify the system on the network and in the shell prompt.

## _3.7_ Root Password
A root password must be set in order to log into the **root user**.  
If this step is skipped, it will not be possible to log into the system as root.

To set the root password, run:
	`passwd`

You will be prompted to enter and confirm the new password.

## _3.8_ Bootloader
Configure the bootloader by following the steps in the Arch Wiki:  
[https://wiki.archlinux.org/title/Systemd-boot#Configuration](https://wiki.archlinux.org/title/Systemd-boot#Configuration)
### Install systemd-boot
Install the bootloader using `bootctl`:
	`bootctl install`

This installs **systemd-boot** to the EFI System Partition.

### Configure the Bootloader
Edit the loader configuration file:
	`nano /boot/loader/loader.conf`

Add the following configuration:
```bash
default arch.conf
timeout 4
console-mode max
editor no
```
- `default` specifies the default boot entry
- `timeout` sets the number of seconds before automatic boot
- `console-mode max` uses the highest available console resolution
- `editor no` disables editing boot parameters at boot time
### Create the Boot Entry
Create a boot entry file for the new system:
	`nano /boot/loader/entries/arch.conf`

Add the following configuration:
```bash
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw
```
- Replace the `UUID` value with the UUID of the **root (`/`) partition**
- In this setup, the root partition corresponds to `/dev/sda3`

### Verify the Configuration
Verify that the bootloader configuration has been installed correctly:
	`bootctl`

This command displays information about the bootloader installation and detected boot entries


# Post-installation
Now following these recommendations, but in the order I deem necessary:
https://wiki.archlinux.org/title/General_recommendations

## System Administration
### Users and groups
#### Create a User Account
Create a new user account for general system use.
	`useradd -m keiran`
- Creates a new user named _keiran_ 
- The `-m` flag creates a home directory at `/home/keiran`
    

Set the password for the new user:
`passwd keiran`

---

#### Create a User Group
Create a group for standard user accounts
	`groupadd user`
- Creates a new group named *user*
#### Add the User to the Group
Add the user account to the group:
	`usermod -aG user keiran`
	- `-a` appends the user to the group
	- `-G` specifies what groups to append to

## Networking
### Persistent Configuration
#### Start the Service
To ensure the networking is configured, we will use the *system-networkd* service.

Start the service via:
	`systemctl start systemd-networkd.service`

Verify the service status:
	`systemctl status systemd-networkd.service`
- Confirm the service is running
- Check that the service is enabled, so it starts automatically at boot
	- If not enabled, run `systemctl enable systemd-networkd.service`

#### Creating a Network Configuration File
Create a file named `10-wired.network` within `/etc/systemd/network`
- File must end with `.network`
- Values below `70` are typically used for the prefix

Example configuration
```INI
[Match]
Name=interface

[Network]
Address=
Gateway=
DNS=
```

Explanation
- `[Match]` determines which interface the configuration is applied to
- `Name=` match the interface name
- `Address=` defines the static IP assigned to the interface
- `Gateway=` specified the default route used to reach external networks
- `DNS=` optionally defines the DNS server used for name resolution

In this system specifically, the interface name was changing across reboots. Because of this, my configuration was updated to match **MAC Address** instead of the interface name

Example:
	`MACAddress=<interface-mac>`

After creating the configuration file, reload the network configuration:
	`networkctl reload`

#### DNS Configuration
To enable DNS resolution, the **systemd-resolved** service must be running.

Start the resolver service:
	`systemctl start systemd-resolved`

This service reads DNS settings defined in the previous `.network` configuration file.


## SSH Daemon Configuration (sshd)
### Overview
On Arch Linux, the SSH server is managed by the OpenSSH daemon, referenced by `sshd` (not `ssh` which some debian-based distros use).

### Starting SSH
	`systemctl start sshd`

### SSH Daemon Configuration
The configuration file is located at:
	`/etc/ssh/sshd_config`


#### Security Changes
The following changes improve security:
- Disable root login
- Change default port
- Limit authentication time
- Limit login attempts
- **Disabled password authentication**
- Disable empty passwords
- Restrict login to a specific user

### Applying Configuration Changes
We need to restart the ssh daemon for the changes to take effect.
	`systemctl restart sshd`

### Setting up ssh directory
We need to do the following steps in our user account's home directory (`/home/user`)
1. Create a `.ssh` directory
	1. `mkdir -p ~/.ssh`
2. Create an `authorized_keys` file
	1. `touch ~/.ssh/authorized_keys`
3. Add your **public key**
	1. Paste the public key into the authorized keys
	2. do not place your private key
#### Required permissions
SSH is very strict about permissions and incorrect settings may cause login failures
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Meaning:
- `.ssh`
	- only accessible to the user
- `authorized_keys`
	- only read/writable by the user
## Configuring firewalld
### Command Alias
Also, I did set an alias for `firewall-cmd` by editing `/etc/profile` and appending the following line:
	`alias firewall='firewall-cmd'`

The remaining commands will refer to `firewall-cmd` via this alias.

### Setting up the firewall
#### Zone Selection
Since I am on a home network, I will be using home for trusted services and block for all non trusted connections. 

Set the default zone to home by:
	`firewall-cmd --set-default-zone=home`

---
#### Listing services
`firewall --zone=home --get-services`
- If no zone is specified, the default zone is used
#### Adding services
For this system, I want to allow:
- SSH
- HTTP
- HTTPS
- HTTP/3 (QUIC over UDP)

`firewall --add-service=<service> --permanent`
- permanent flag persists this change through reboots
##### Modifying a service
Since I changed the SSH port earlier, I needed to update the service:

```
firewall --service=ssh --remove-port=22/tcp --permanent
firewall --service=ssh --add-port=<port>/tcp --permanent
```
#### Applying changes
`firewall --reload`

#### Testing
To verify firewall worked without `nftables` running, I used ICMP (ping)

ICMP does not use ports, so it must be handled separately

##### Block ICMP
`firewall --add-icmp-block=echo-request`

Test:
- Ping from another machine
	- Fails
##### Unblock ICMP
`firewall --remove-icmp-block=echo-request`

Test:
- Ping again
	- Success
### Issues Encountered
After installation, I initially tried to start both `nftables` and `firewalld` services.

- `nftables` failed because it is not required to run alongside firewalld
- `firewalld` initially threw a Python-related error

After some searching, there was a post about `nftables` needing a kernel module to be loaded.
After rebooting, both issues were resolved

---
Another issue occurred after starting the firewall
- Firewall rules allowed SSH for the default port of 22
- I had to access the system through the Proxmox console to fix it
