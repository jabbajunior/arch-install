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