# Secure Drive Wipe Utility (`wipe-disk`)

A safe, interactive Bash script designed to sanitize block storage devices on Linux systems. It automatically detects the underlying storage medium (NVMe, SATA SSD, USB Flash, or HDD) and applies the most appropriate, hardware-level or software-level sanitization method.

## Features
* **Multi-Method Wiping Logic** :
	* **NVMe** : Uses native NVMe controller formatting (`nvme format -s 1`) for rapid cryptoweave/user data erase.
	* **SATA SSD** : Uses ATA Secure Erase via `hdparm` (with automatic fallback to shred if locked in `frozen` mode by system BIOS/UEFI).
	* **USB Flash / External SSD** : Executes a pseudo-random pass via `shred` followed by zero-filling partition header sectors.
	* **HDD (SATA/USB)** : Full zero-fill overwrite using block-aligned `dd`.

* **Safety & Verification Checks** :
	* Root privilege verification.
	* Filters out system loops, zram, and optical drives (`/dev/sr*`) from list displays.
	* Unmounts active partitions automatically before initiating the wipe.
	* Explicit user confirmation (`YES`) required to proceed.

## Prerequisites
Ensure the required system tools are installed on your distribution :
```Bash
# Debian / Ubuntu / Linux Mint
sudo apt update && sudo apt install -y coreutils util-linux hdparm nvme-cli

# Fedora / RHEL
sudo dnf install -y coreutils util-linux hdparm nvme-cli
```

## Usage
1. **Make the script executable** :
```Bash
chmod +x wipe-disk
```
    
2. **Run as root (or with `sudo`)** :
```Bash
sudo ./wipe-disk
```

3. **Follow the interactive prompts** :
* Inspect the output table to verify device attributes (Model, Size, Interface, Rotational status).
 * Input the target device node name (e.g., `sdb` or `nvme0n1`). Do not select your primary system drive (typically `sda` or `nvme0n1`).
 * Confirm the operation by typing `YES`.

## Wiping Methods Summary

| Device Type | Detection Criteria | Sanitization Execution |
| ----------- | ------------------ | ---------------------- |
| **NVMe** | Device name starts with `nvme`	| `nvme format /dev/nvmeXn1 -s 1 -f` |
| **USB HDD** | `TRAN=usb` & `ROTA=1`	| `dd if=/dev/zero of=/dev/sdX bs=4M conv=fdatasync` |
| **USB Flash / SSD** | `TRAN=usb` & `ROTA=0` | `shred -v -n 1 /dev/sdX` + trailing zero-pass |
| **SATA SSD** | `ROTA=0` | ATA Secure Erase (`hdparm`) or `shred` fallback |
| **Internal HDD** | `ROTA=1` | Full zero-fill via `dd` |

**Warning** : *This operation causes permanent and irreversible loss of all partition tables, file systems, and stored data on the specified target block device. Ensure proper backups exist prior to execution.*