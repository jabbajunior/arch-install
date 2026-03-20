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

# 3/11/2026
Worked on sections **1.1** of the Arch Linux post-install configuration based on the **Arch Wiki General Recommendations**.

Tasks completed:
- **Created a standard user account**
    - Added a new user `keiran` using `useradd -m`
    - Generated a home directory at `/home/keiran`
    - Set the user password using `passwd`
- **Configured user groups**
    - Created a new group `user` using `groupadd`
    - Added the user `keiran` to the group using `usermod -aG`
    - Verified group membership with the `groups` command
- **Tested user account functionality**
    - Confirmed the ability to log into the system using the newly created user account
    - Verified that the home directory and permissions were correctly initialized
- **Investigated privilege escalation behavior**
    - Attempted to restrict direct root access by locking the root account using `passwd -l`
    - Observed that locking the root password prevents authentication via `su` and other password-based escalation methods
- **Recovered from root account lockout**
	- After exiting the system, I realized I was locked out
	- Had to remount the ESP to `/boot`
	- Used the EFI shell to manually edit the boot entry to include `rw init=/bin/bash`
    - Unlocked the root account using `passwd -u root`
- **Restored normal bootloader configuration**
    - Re-enabled boot entry editing within **systemd-boot**
    - Confirmed successful system boot and restored administrative access

# 3/12/2026
Worked on **network configuration** after discovering the system had no network connectivity following reboot.

Tasks completed:
- **Diagnosed network connectivity issue**
    - Observed that the system had no configured IP address or default route
    - Determined that the network interface was not being automatically configured
- **Tested temporary networking configuration**
    - Assigned a temporary IP address using  
        `ip address add 10.0.0.233 dev ens18`
    - Added a default route to the gateway using  
        `ip route add default via 10.0.0.1 dev ens18`
    - Confirmed that these commands restored **local network connectivity**
- **Identified persistence issue**
    - Rebooted the system and observed that the manually assigned IP address and route were removed
    - Determined that the changes were **temporary runtime configurations**
- **Implemented persistent network configuration**
    - Selected **systemd-networkd** as the network management service
    - Started the service using  
        `systemctl start systemd-networkd.service`
    - Enabled the service to start automatically at boot
- **Created network configuration file**
    - Created `/etc/systemd/network/10-wired.network`
    - Configured a **static IP address, gateway, and DNS server**
    - Set DNS to point to the **Pi-hole server** for local DNS resolution
- **Configured DNS resolution**
    - Started the **systemd-resolved** service to provide DNS resolution for the system
- **Verified network functionality**
    - Confirmed persistent IP configuration after restarting networking services
    - Verified DNS resolution through the configured Pi-hole server


# 3/17/2026
Focused on **package management** and **SSH configuration** as part of system setup and hardening.

Tasks completed:
- **Learned pacman fundamentals**
	- Reviewed official manpage: [https://man.archlinux.org/man/pacman.8.en](https://man.archlinux.org/man/pacman.8.en)
	- Watched video: https://www.youtube.com/watch?v=-dEuXTMzRKs
	- Studied basic usage and command structure (operations vs options)
- **Installed and started SSH daemon**
    - Started service using  
        `systemctl start sshd`
    - Enabled service for persistence across reboots
- **Configured SSH daemon settings**
    - Edited `/etc/ssh/sshd_config` to improve security:
        - Disabled root login
        - Changed default port from 22
        - Limited login grace time to 30 seconds
        - Restricted authentication attempts to 3
        - Disabled password authentication (key-based only)
        - Disallowed empty passwords
        - Restricted access to a specific user
- **Configured user SSH access**
    - Created `~/.ssh` directory and `authorized_keys` file
    - Added public key for authentication
- **Set correct permissions**
    - Set `.ssh` to `700` (owner only access)
    - Set `authorized_keys` to `600` (owner read/write only)
- **Verified secure access**
    - Ensured key-based login works correctly after configuration

# 3/20/2026
Focused on **systemd fundamentals**, including unit management, service control, and logging.

Tasks completed:
- **Studied systemd basics**
    - Reviewed Arch Wiki and systemd man pages
    - Learned role of `systemd` as the init system and its use of **units**
- **Reviewed** `**systemctl**` **usage**
    - Covered common commands (`status`, `start`, `stop`, `restart`, `enable`, etc.)
    - Understood key distinctions like `start` vs `enable` and `reload` vs `restart`
- **Learned unit file structure**
    - Covered `[Unit]`, `[Service]`, and `[Install]` sections
    - Reviewed dependency handling and systemd targets (vs SysV runlevels)
- **Studied systemd logging**
    - Learned use of `journalctl` and basic filtering options
- **Reflected on systemd design**
    - Noted benefits of centralized management and faster boot
    - Considered tradeoffs with systemd’s broader scope vs traditional Unix philosophy