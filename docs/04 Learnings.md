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

---
## Root Account Lockout
If the **root account is locked** and there are **no users configured with `sudo` privileges**, it is not possible to perform administrative tasks on the system.

Locking the root account using `passwd -l` prevents authentication through methods such as `su` or direct root login. Without another privileged account, the system effectively becomes **administratively locked**.

### Recovery Process
To recover administrative access:
1. Rebooted the virtual machine and accessed the **systemd-boot menu**
2. Booted into the **UEFI shell**
3. Located the bootloader entry file:  
    `/boot/loader/entries/arch.conf`
4. Edited the boot entry and modified the kernel options line:
    
Original entry:
	`options root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

Modified entry:
	`options root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw init=/bin/bash`

5. Booted the system using the modified entry
6. The system started directly in a **root shell**
7. Remounted the root filesystem as read/write if necessary
8. Unlocked the root account using:
	`passwd -u root`

After unlocking the root account, normal system administration access was restored.
### Lesson Learned
Before locking the root account, ensure that at least one user has **privilege escalation capabilities (e.g., `sudo`)**.  

Otherwise, the system may require **bootloader modification or recovery media** to regain administrative access.

---
## Persistent Network Configuration
Manual networking configuration performed with `ip` commands does not persist across system reboots. Commands such as `ip address add` and `ip route add` modify the system’s **current runtime network state**, but they do not create permanent configuration.

While troubleshooting network connectivity, an IP address and default route were manually assigned to restore temporary LAN connectivity. However, after rebooting the system the configuration was lost because it had not been managed by a network configuration service.

Arch Linux does not automatically configure networking unless a **network management daemon** is enabled.
### Lesson Learned
Temporary networking commands are useful for **diagnosing connectivity issues**, but they should not be relied on for permanent configuration.

To maintain persistent network configuration across reboots, a network management service such as `systemd-networkd` must be used. DNS resolution also requires a resolver service such as `systemd-resolved` to be running.

---

## Pacman Usage
### Overview
`pacman` is Arch Linux's package manager. It handles installing, updating, removing, and querying packages from repositories and the local system, but functions differently from other distributions package managers.

Manpage: https://man.archlinux.org/man/pacman.8.en
Useful video: https://www.youtube.com/watch?v=-dEuXTMzRKs
### Syntax
Pacman uses the following syntax:
- Capital letters = **operations** (what you want to do)
- Lowercase letters = **options** (modify behavior)
Format:
	`pacman -operation [options]`

### Common Operations
#### Sync (`-S`)
Use for installing and updating packages from repositories.

Key options:
- `-y`
	- Refresh package databases
- `-u`
	- upgrade all packages
#### Query (`-Q`)
Used to inspect packages installed on the local system.

Key options:
- `-e`
	- explicitly installed packages (not dependencies)
- `-d`
	- packages installed as dependencies
- `-t`
	- unrequired/orphaned packages
- `-m`
	- foreign packages (not from official repos, e.g. AUR)
- `-n`
	- Native packages
#### Removal (`-R`)
Used to uninstall packages.

Key options:
- `-s`
	- Remove dependencies not required by other packages
- `-n`
	- Remove config files
---

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

---
## systemd
### Overview
`systemd` is a widely used *init system* in many Linux distributions.

It is responsible for:
- Starting and managing system processes
- Handling services such as networking, logging, and virtualization

`systemd` runs as a background *daemon* and manages resources called *units*.
### Sources
https://wiki.archlinux.org/title/Systemd
- Section 1
- Section 2.1 + 2.2
- Section 3.1 + 3.3
- Section 4
https://man.archlinux.org/man/systemd.unit.5
https://www.youtube.com/watch?v=Kzpm-rGAXos
https://www.youtube.com/watch?v=Fz8Ldw-s8_Q

### Managing systemd (`systemctl`)
#### Syntax
`systemctl COMMAND [UNIT]`

##### Common Commands
`status`
Displays the current state of a unit, including:
- Whether it is currently running
- Whether it is enabled or disabled
	`systemctl status <unit>`


`start`
Starts a unit.
	`systemctl start <unit>`


`stop`
Stops a running unit.
	`systemctl stop <unit>`

`restart`
Stops and then starts a unit.
	`systemctl restart <unit>`


`reload`
Reload a unit's configuration **without stopping it**.
	`systemctl reload <unit>`

Use when want to apply config changes **without** service interruption.


`enable`
Configure a unit to start automatically when its **target is reached**.
	`systemctl enable <unit>`


`disable`
Prevents a unit from starting automatically.
	`systemctl disable <unit>`


`list-units`
List all units currently loaded into memory.
	`systemctl list-units`

---
#### Units
`systemd` manages resources as *units*, identified by their suffix.

Types:
- `.service`
	- services (most common)
- `.mount`
	- mount points
- `.device`
	- hardware devices
- `.socket`
	- socket-based activation

**If no suffix is specified, `systemctl` assumes `.service`**
##### Unit Files
Unit files define how a unit behaves.
They typically contain three optional sections :
- [Unit]
- [Service]
	- for service units only
- [Install]
###### Unit
Defines:
- Description and metadata
- Dependencies on other units
- Startup order

Common options:
- `Wants=`
	- soft dependency on other unit
- `Requires=`
	- strong dependency on other unit
- `After=`
	- start after another unit
- `Before=`
	- start before another unit

###### Service
Defines how the service runs.

`Type=` options:
- `simple`
	- default, runs in foreground
- `forking`
	- process forks
- `oneshot`
	- runs once and exits
- `notify`
	- wait for readiness notification
- `dbus`
	- activated via D-Bus
- `idle`
	- delayed execution
###### Install
Defines when an **enabled unit starts** and is tied to system targets.

Common option:
- `WantedBy=`
	- specifies which target start this unit


Targets (Runlevels)
Targets replace traditional SysV runlevels and here is a table found on the arch wiki.

| SysV Runlevel | systemd Target    | Notes                       |
| ------------- | ----------------- | --------------------------- |
| 0             | poweroff.target   | Shut down the system        |
| 1, s, S       | rescue.target     | Single user mode (recovery) |
| 2, 4          | multi-user.target | Same as runlevel 3          |
| 3             | multi-user.target | Multi-user, non-graphical   |
| 5             | graphical.target  | Multi-user with GUI         |
| 6             | reboot.target     | Reboot the system.          |


### Logging
`systemd` provides its own logging system via `journald`.

Logs are stored in the system journal and can be queried using `journalctl`.

#### Viewing Logs
Use `journalctl` to inspect binary logs.

##### Common Options
`-e`
- Jump to the most recent entries

`-f`
- Follow logs in real time

###### Filtering Logs
**By pattern**
- `journalctl --grep=PATTERN`

**By time**
Show logs from a specific time onward:
- `journalctl --since "20 min ago`
	- can also supply a specific timestamp

**By system unit**
Filter logs for a specific service:
- `journalctl --unit <unit.service>`
	- 

**By user unit**
Filter logs for a user service:
- `journalctl --user-unit <unit>`
- User services run for a specific user, not system wide.


### Learning
Since `systemd` is the init system, it is convenient that many aspects of the system can be managed through a single interface (`systemctl`). This provides a more unified experience compared to SysV systems, where different tools with different syntaxes were required for tasks like managing services, networking, and mounting.

Compared to SysV init, `systemd` is more flexible because it does not require all processes associated with a runlevel to start sequentially. Instead, it can start units based on their dependencies, allowing for parallelization where possible. This leads to a more efficient and faster boot process.

A concern is that `systemd` appears to extend beyond the traditional responsibilities of an init system. Rather than only handling the boot process and starting services, it also manages logging, networking (in some configurations), and other system components. This makes it feel closer to a broader system management layer, potentially acting almost like a wrapper around the kernel rather than strictly adhering to the minimal role expected of an init system.

---

## Firewall
### Overview
My last modification to this system is to install a firewall. I have previous exposure to `iptables` and wanted to learn more about its modern replacement, `nftables`.

While reading the Arch Wiki, I found that a higher-level frontend called `firewalld` is commonly used (especially on RHEL-based distributions). This tool manages firewall rules more easily by abstracting away direct rule manipulation.

### Sources
- https://www.redhat.com/en/blog/beginners-guide-firewalld
- https://www.youtube.com/watch?v=JINRru-W1_s
- https://wiki.archlinux.org/title/Nftables

### Installation
Install the required packages
	`pacman -S nftables firewalld`

Enable and start firewalld:
	`systemctl enable --now firewalld`

> Note: `nftables` does not need to be started as a service.

Restart the system.
	`reboot`
### Basics
`firewalld` is a **zone-based firewall**, built on top of a **stateful firewall** (`nftables`).

Firewall rules are applied to a zone and each connection belongs to a single zone.

---
#### Zones
The default zones are:
- `drop`
    - Drops all incoming packets silently
    - Outgoing connections allowed
- `block`
    - Rejects incoming packets with ICMP response
    - Outgoing connections allowed
- `public`
    - Default untrusted network
    - Only selected incoming connections allowed
- `external`
    - Used for routed networks with masquerading
- `dmz`
    - For systems exposed to the internet
- `work`
    - Trusted work environments
- `**home**`
    - Trusted home networks
    - Only selected incoming connections allowed
- `internal`
    - Internal networks (similar to home/work)
- `trusted`
    - All traffic allowed (no filtering)

#### Zone Selection
Since I am on a home network, I will be using home for trusted services and block for all non trusted connections. 

Set the default zone to home by:
	`firewall-cmd --set-default-zone=home`

---
#### Command Alias
Also, I did set an alias for `firewall-cmd` by editing `/etc/profile` and appending the following line:
	`alias firewall='firewall-cmd'`

The remaining commands will refer to `firewall-cmd` via this alias.

---

### Setting up the firewall
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
### Learnings
From this, I learned that a **zone-based firewall is an abstraction layer** on top of a lower-level firewall engine.

In this case:
- `nftables` handles the actual packet filtering
- `firewalld` provides a more user-friendly interface to manage those rules
    
Instead of manually writing rules like `ACCEPT`, `DROP`, or `FORWARD`, I can assign **services or ports to zones**, and `firewalld` translates that into the appropriate `nftables` rules behind the scenes.

---

One of the biggest advantages I noticed is **simplicity compared to nftables/iptables**.

With `nftables` or `iptables`:
- You need to understand **chains, hooks, tables, and rule ordering**
- It’s easy to misconfigure rules or create conflicts

With `firewalld`:
- You interact at a higher level (zones and services)
- The complexity of rule ordering and chaining is handled for you

This makes it much easier to:
- quickly allow or block services
- reason about firewall behavior
- avoid breaking access (like SSH)

That said, this abstraction comes with a tradeoff:
- You lose some **fine-grained control**
- Debugging can be harder if you don’t understand the underlying `nftables` rules

---

I did not explore all of `firewalld`’s features, but based on this experience, I would likely use:
- `firewalld` or `ufw` for general system administration
- `nftables` directly for more advanced or custom firewall setups

Overall, I think zone-based firewalls provide a much more approachable way to manage firewall rules, especially when learning or managing systems where simplicity and reliability are more important than full control.