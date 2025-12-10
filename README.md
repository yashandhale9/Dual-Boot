# 🚀 Dual Boot Journey: Windows 11 + Ubuntu (Full Recovery & Install Story)

This repository documents my **complete real-life journey** of setting up a **dual-boot system with Windows 11 and Ubuntu**, fixing partition issues, recovering lost data, creating bootable USBs, and restoring GRUB.

This is not just a guide — it’s a full **case study** of how dual booting can go wrong and how every problem was solved step-by-step.

---

## 🧑‍💻 **System Specifications**

| Component | Details |
|----------|---------|
| Laptop | ASUS VivoBook |
| Storage | 512GB NVMe SSD |
| RAM | 8GB |
| OS (initially) | Windows 11 |
| OS (after setup) | Windows 11 + Ubuntu (Dual Boot) |

---

#  1. **Goal of the Project**
- Learn **Linux Internals**, UNIX processes, and system-level concepts.
- Install Ubuntu alongside Windows for development & Linux learning.
- Maintain Windows for daily apps and gaming.
- Explore partitioning, GRUB, EFI, file recovery, and system boot flow.

---

#  2. **Initial State**
- Windows 11 already installed.
- Laptop had **C:**, **D:**, **E:** partitions.
- Goal → Shrink a partition (E drive) → create 100GB free space → install Ubuntu.

---

#  3. **Partitioning Phase**

##  Tools Used: **GParted**
GParted is a partition editor that lets you:
- Resize partitions  
- Create/Delete partitions  
- Move partitions  
- Format storage  

### 🔹 What I Did:
1. Booted into Ubuntu Live USB.
2. Opened **GParted**.
3. Selected **E: partition** (Windows NTFS).
4. Shrunk by **100GB** to create *unallocated space*.
5. This 100GB would become Ubuntu’s root `/`.

---

#  4. **Ubuntu Installation (The Mistake 😭)**

During install, there are options:
- Erase entire disk  
- Install alongside Windows  
- Something else (manual partitions)

### ❌ **Accidentally selected: _Erase Disk and Install Ubuntu_**
This option:
- Wiped Windows Boot Manager  
- Wiped Windows partitions  
- Created new Ubuntu EFI + EXT4 + SWAP partitions  
- Completely destroyed the original C:, D:, E: partitions  

💥 **This triggered the entire recovery mission.**

---

#  5. **Bootloader Issue**
After installation:
- Windows was gone  
- Only Ubuntu booted  
- EFI entries of Windows removed  
- GRUB had no Windows entry  

---

#  6. **Data Loss Discovery & Panic Mode 😱**

The biggest issue:
- **D: drive had important personal data**
- Installation wiped the NTFS structure

So recovery was required urgently.

---

#  7. **Recovery Phase: Tools Used**

##  **TestDisk**
A powerful tool to:
- Recover lost partitions  
- Repair boot sectors  
- Rebuild partition table  

### 🔹 Results:
- Detected old partitions  
- BUT filesystem corrupted  
- Could NOT restore D: partition directly  

---

## 🔧 **PhotoRec**
When file system is destroyed, PhotoRec recovers **raw files**, not folders.

### 🔹 What It Does:
- Scans disk block-by-block  
- Finds recognizable file signatures  
- Recovers images, video, documents  
- Saves them into `recup_dir.*` folders  

### 🔹 My Results:
- ~845 folders recovered  
- Thousands of files rescued  
- Filenames lost, but data safe  

---

#  8. **Storage Issues**
Recovered data filled Ubuntu storage → GUI broke:
gdm.service failed

Switched to terminal-only login (TTY mode).

---

#  9. **Fixing GUI via Terminal**
Commands used:

```bash
sudo apt --fix-broken install
sudo apt install --reinstall gdm3
sudo reboot
```
After freeing space → GUI restored.

---

#  10. **Handling Permission Errors**

Recovered files were root-owned.

### Fix:
```
sudo chown -R $USER:$USER recup_dir.*
sudo rm -rf recup_dir.
```

#  11. **Creating Bootable Windows USB with Ventoy**
###  Ventoy

Ventoy makes a USB bootable by simply copying ISO files directly.

### Steps:
```
sudo sh Ventoy2Disk.sh -i /dev/sda
```

Then copy Windows ISO:
```
Win11_25H2_EnglishInternational_x64.iso
```

Boot → Ventoy menu → Select Windows ISO → Install Windows.

---

#  12. **Windows Installation**

Selected:

138GB unallocated space

Installed Windows successfully **without deleting Ubuntu.**

---

#  13. **Dual Boot Setup**

After Windows installation:

- Windows replaced GRUB

- Only Windows was booting

To restore Ubuntu entry:

**GRUB Reinstallation Commands:**
```
sudo mount /dev/nvme1n1p2 /mnt
sudo mount /dev/nvme1n1p1 /mnt/boot/efi
for i in /dev /proc /sys /run; do sudo mount --bind $i /mnt$i; done
sudo chroot /mnt
grub-install /dev/nvme1n1
update-grub
exit
```
Reboot → GRUB Back → Both OS visible ✔️

---

#  14. **Final Setup**

Windows boots perfectly

Ubuntu boots perfectly

Dual boot stable

All recovered data saved to external device

System restored & clean

---

# 🎓 **Key Learnings**
🔸 Bootloader Internals (GRUB vs Windows Boot Manager)

🔸 EFI Partition Importance

🔸 How Linux mounts disks (lsblk, df -h)

🔸 Data Recovery Concepts (Raw carving, MFT corruption)

🔸 Secure USB flashing with Ventoy

🔸 Partitioning safety rules

🔸 Linux permissions & root handling

---

# 📂 **Commands Cheat Sheet**
```
lsblk                # list disks
df -h                # check storage usage
sudo gparted         # partition tool
sudo testdisk        # recover partitions
sudo photorec        # recover raw files
sudo chown -R USER   # fix ownership
sudo rm -rf          # delete folders
sudo sh Ventoy2Disk.sh -i /dev/sdX   # install Ventoy
grub-install         # reinstall GRUB
update-grub
```
---

#  **Tips for Beginners**

NEVER click Erase Disk unless you want a clean wipe

Always backup partitions before dual booting

Ventoy is the best tool for multi-boot USB creation

GParted is safer than Windows Disk Management in many cases

When recovering data → do NOT write to the same disk

---

### From wiping entire disk → to recovering data → to rebuilding bootloaders → to achieving a stable dual-boot…

### This project transformed a mistake into mastery. 🔥

