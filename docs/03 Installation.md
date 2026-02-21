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

## 1.5 Set the console keyboard layout and font
Not sure how to organize this document
On section 1.5