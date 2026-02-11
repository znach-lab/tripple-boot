# MACOS TRIPLE BOOT GUIDE

## Device: MacBook Pro (Retina, 15-inch, Late 2013) – Intel

This guide describes how I ran the following three operating systems on a single disk in a stable triple boot configuration using only OpenCore on a MacBook Pro 2013 Late 15 (Intel):

1. macOS Sequoia

2. Windows 11

3. Manjaro

Goal:

- Without using reFind

- Without experiencing boot flag confusion

- Preventing Windows from messing up the EFI order

- Managing all systems through a single OpenCore chain

---

# Disk Configuration

Before starting the installation, the disk is manually partitioned. My setup is as follows:

- 128GB Windows 11

- 128GB Manjaro

    - 8GB swap

    - 60GB root (/)

    - 60GB home (/home)

- 1GB EFI (Special space reserved for OpenCore)

- 128GB macOS Sequoia

### Why is 1GB of EFI Space Reserved?

This space is reserved for the OpenCore boot chain. When macOS starts, other operating systems are launched through this EFI. Using a separate EFI prevents Windows from disrupting the boot order and provides a permanent structure.

---

# macOS Sequoia Installation

Initially, macOS Big Sur was installed on the system.

## OpenCore Legacy Patcher Installation

1. Download the latest version of OpenCore Legacy Patcher.

2. Install it.

3. Download the Sequoia image using the "Install macOS" option.

Once the download is complete, the USB writing screen will appear.

⚠ The USB will be formatted. Everything on it will be erased.

It is recommended not to erase this USB. It saves lives if the boot chain is corrupted.

The USB writing process may take some time.

---

## Sequoia Installation

1. Turn off the computer.

2. Press the Option key to open the boot menu.

3. Boot from the USB.

4. Install macOS to the 128GB partition you allocated.

After the installation is complete, you need to install OpenCore to the disk without removing the USB.

---

## Installing OpenCore to Disk

1. Select “Install to disk” from the OpenCore notification.

2. Select the main disk.

3. Select the 1GB EFI partition you allocated.

4. Complete the After Installation steps.

After this stage, macOS Sequoia will work stably.

---

# Manjaro Installation

We start the installation by placing the ISO file on a USB drive prepared with Ventoy.

## ⚠ WiFi Driver Problem

The Broadcom WiFi driver may not be automatically installed on the MBP 2013 model.

My solution:

1. I created a Manjaro VM on another computer using libvirt.

2. I established internet access via NAT connection.

3. I ran the WiFi driver script.

4. I transferred the driver packages to the MBP via USB.

5. I installed it offline.

---

## Manual Partition

Manual partitioning must be selected during Manjaro installation.

- Select the 128GB allocated space

- Create an 8GB swap

- 60GB root (/)

- 60GB home (/home)

- Select the 1GB OpenCore EFI you previously allocated as EFI

The installation is complete.

---

# Windows 11 Installation

Since it's a Mac with an Intel processor, a Windows 11 bootable USB can be prepared using Boot Camp Assistant.

1. After preparing the USB, boot from it using Option.

2. The Windows installation screen will open.

## TPM / Secure Boot Obstacle

Due to the old hardware, Windows 11 will give an incompatibility error during installation.

For Bypass:

### Open CMD
Shift + F10

### Open Registry
```
regedit
```

### Navigate to this path
```
HKEY_LOCAL_MACHINE\SYSTEM\Setup
```

Create a new key under Setup:

```
LabConfig
```
Create the following DWORD (32-bit) values ​​in LabConfig:

- BypassTPMCheck = 1

- BypassSecureBootCheck = 1

- BypassRAMCheck = 1

Close Registry → Back → Install

Installation continues.

---

# Post-Windows GRUB Problem

Windows changes the EFI order after installation.

Manjaro cannot boot and the GRUB rescue screen appears.

---

## Temporary Access from Rescue Mode

List partitions:

```
ls
```
After finding the partition containing root and /boot:

```
set root=(hdX,gptY)
set prefix=(hdX,gptY)/boot/grub
insmod normal
normal
```

Manjaro starts.

---

# Permanent GRUB Fix

After Manjaro starts:

```
lsblk -f
```

Check the EFI mount point.

Then:

```
sudo grub-install \
--target=x86_64-efi \
--efi-directory=/boot/efi \
--bootloader-id=GRUB \
--recheck
```
Check:

```
ls /boot/efi/EFI
```

Finally:

```
sudo update-grub
```

This process writes a clean and permanent GRUB EFI to the EFI where OpenCore is located.

---

# Result

- OpenCore runs as the main boot chain.

- macOS Sequoia is stable.

- Windows does not change the boot flag.

- Manjaro GRUB is permanent.

- A simple and controlled triple boot is achieved without using reFind.

---

# Boot Architecture

```
UEFI
↓
OpenCore (1GB EFI)
↓
├── macOS Sequoia
├── Windows 11
└── GRUB → Manjaro
```

Thanks to this architecture, no operating system disrupts the boot order of another, and the system remains stable in the long term.
