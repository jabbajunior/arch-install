This document captures key concepts and lessons learned during the Arch Linux installation process.

# Summary
- [[04 Learnings#^440aa1|Partition Management Tools]]
- [[04 Learnings#^12a749|Filesystem Table]]
- [[04 Learnings#^e49768|Root Password Requirement]]
- [[04 Learnings#^8d8d80|Root Account Lockout]]
- [[04 Learnings#^374f87|Persistent Network Configuration]]
- [[04 Learnings#^7fbf37|Firewall Learnings]]

---
## Partition Management Tools

^440aa1

### Key Lesson
When working with disks, I need to clearly distinguish between **inspection tools** and **modification tools**, and intentionally choose the correct one based on the task.

### What Changed for Me
Before this project, I primarily used `lsblk` and treated it as my main way of interacting with disks. Through this installation, I learned that:
- `lsblk` is **read-only** and used for inspecting disk state
- Tools like `fdisk` are required to **actually modify partition tables**

This clarified that disk management is not a single-tool workflow, but a combination of:
1. Inspect → understand current state
2. Modify → apply changes
3. Re-inspect → verify results

### Practical Rules Going Forward
- Use `lsblk` (or similar tools) to **verify current disk layout before making changes**
- Use `fdisk` (or equivalent) when **creating or modifying partitions**
- Always re-run inspection tools after changes to confirm correctness
- Never assume a tool modifies state unless explicitly designed to do so

### Example from This Project
- Used `fdisk` to create:
    - EFI System Partition
    - Swap partition
    - Root partition
- Used `lsblk` to verify partition layout and mount points

---
## Filesystem Table (`fstab`)

^12a749
### Key Lesson
I must explicitly define how filesystems are mounted at boot, rather than assuming they will be available automatically.

### What Changed for Me
Before this project, I had used `/etc/fstab` but did not fully understand its role. I now understand that:
- `fstab` defines **persistent mount configuration across reboots**
- The system does not “remember” mounts unless they are explicitly configured
- Critical directories (e.g., `/`, `/boot`) depend on correct mount definitions to function after boot

I also learned that `fstab` represents a **declarative approach** to system configuration—defining _what should be mounted_, not manually mounting each time.
### Practical Rules Going Forward
- Always ensure required filesystems are defined in `/etc/fstab` before rebooting
- Use **UUIDs** instead of device names to avoid issues with changing device paths
- Treat mount configuration as part of system initialization, not a post-boot step
- Verify mounts after boot to confirm `fstab` is correct

### Example from This Project
- Generated `fstab` using:
	- `genfstab -U /mnt >> /mnt/etc/fstab`
- Used UUID-based entries for reliability
- Ensured root and boot partitions mount automatically on startup
---
## Root Password Requirement

^e49768

### Key Lesson
System access must be explicitly configured during setup. If no valid authentication method (e.g., root password or user account) is defined, the system can become inaccessible and require offline recovery.

### What Changed for Me
Before this, I treated setting a root password as a routine step. By intentionally skipping it, I learned that:
- A freshly installed system does not guarantee **any accessible login path**
- Missing authentication configuration can result in being completely locked out
- Recovery requires booting into an external environment and modifying the system offline

This reinforced that **access configuration is a critical part of system initialization**, not an optional step.

### Example from This Project
- Intentionally skipped setting a root password during installation
- Result: Unable to log into the system after boot
- Recovered by:
    - Booting into installation media
    - Mounting partitions
    - Using `arch-chroot` to enter the system
    - Setting the root password with `passwd`

---
## Root Account Lockout

^8d8d80

### Key Lesson
Administrative access must always be preserved through at least one valid path. Disabling or locking the root account without configuring an alternative privileged user can render the system administratively unusable.

### What Changed for Me
I intentionally locked the root account (`passwd -l`) to test whether `su` could still be used for privilege escalation. I learned that:

- Locking the root account disables authentication for root entirely
- `su` does not bypass this restriction—it still requires valid root credentials
- Without a user configured with `sudo` (or equivalent), there is **no way to perform administrative actions** within the system

This revealed that **privilege escalation depends on available authentication paths**, not just command availability.
### Example from This Project
- Locked the root account using `passwd -l`
- No `sudo`-enabled user was configured
- Result: No ability to perform administrative tasks

Recovered by:
- Modifying bootloader kernel parameters (`init=/bin/bash`)
- Booting into a root shell
- Unlocking the root account with `passwd -u root`

---
## Persistent Network Configuration

^374f87

### Key Lesson
Runtime configuration changes do not persist across reboots. For any system state that must survive restarts, I need to use a **managed, declarative configuration mechanism**, not ad-hoc commands.
### What Changed for Me
I initially assumed I could configure networking manually using `ip` commands and avoid using a network manager. Through troubleshooting, I learned that:
- Commands like `ip address add` and `ip route add` only modify the **current runtime state**
- These changes are lost on reboot unless managed by a service
- Arch Linux does not provide persistent networking by default; **it requires explicit configuration via a network service**

This clarified the distinction between:
- **Ephemeral configuration** (manual commands)
- **Persistent configuration** (managed by system services)
### Example from This Project
- Manually configured IP address and route to restore connectivity
- Lost network configuration after reboot
- Resolved by configuring a network management service for persistence

---

## Firewall Learnings

^7fbf37

### Key Lesson
A zone-based firewall (like `firewalld`) is an abstraction layer on top of a lower-level firewall engine (`nftables`). This abstraction simplifies common tasks but hides underlying complexity.

### What I Learned
Through this setup, I learned that:

- `nftables` handles the actual packet filtering
- `firewalld` provides a higher-level interface to manage those rules

Instead of manually defining rules like `ACCEPT`, `DROP`, or `FORWARD`, I can assign **services or ports to zones**, and `firewalld` translates those into the appropriate `nftables` rules behind the scenes.

---
### Tradeoffs Observed
#### Simplicity (firewalld)
- Interact using **zones and services** instead of low-level rules
- No need to manage rule ordering, chains, or hooks directly
- Easier to:
    - allow/block services quickly
    - reason about firewall behavior
    - avoid breaking access (e.g., SSH)

#### Control (nftables / iptables)
- Requires understanding:
    - chains
    - hooks
    - tables
    - rule ordering
- Easier to misconfigure or create conflicts
- Provides full control over packet filtering behavior

---
### Key Insight
There is a clear tradeoff between:
- **Simplicity and safety (firewalld)**
- **Flexibility and control (nftables)**

Using a higher-level tool reduces complexity but requires some understanding of the underlying system for debugging and advanced use cases.
