This document outlines key architectural decisions made during the Arch Linux installation and explains the reasoning behind each choice.

## Filesystem for the Root (`/`) Partition
**Decision:** Use `ext4` as the filesystem for the root partition.

**Rationale:**
- `ext4` is the **default and most widely supported filesystem** for Linux systems.
- It is **stable, well-tested, and performant** for general-purpose workloads.
- It requires **no additional packages or kernel modules** during installation.
- Recovery tools and documentation for `ext4` are widely available.

Alternative filesystems such as **ZFS** were considered but not used because:
- ZFS requires **additional packages and kernel modules** not included in the base Arch installation.
- Installing ZFS increases **system complexity** during the initial setup.
- ZFS is better suited for systems requiring **advanced features such as snapshots, deduplication, or storage pools**, which are unnecessary for this environment.
    

For a minimal Arch installation, `ext4` provides the best balance of **simplicity, reliability, and compatibility**.

---
## Time Synchronization

**Decision:** Use `systemd-timesyncd` for Network Time Protocol (NTP) synchronization.

**Rationale:**
- `systemd-timesyncd` is **included with systemd**, which is the default init system for Arch Linux.
- It provides **lightweight NTP synchronization** without requiring additional packages.
- It is simpler to configure than alternatives such as `chrony` or `ntpd`.

---
## Bootloader

**Decision:** Use `systemd-boot` as the system bootloader.

**Rationale:**
- `systemd-boot` is **simple and minimal**, designed specifically for UEFI systems.
- Configuration is straightforward and uses **plain text configuration files**.
- It integrates well with **systemd-based distributions such as Arch Linux**.
- It has **fewer configuration layers** compared to more complex bootloaders.