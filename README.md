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

**This triggered the entire recovery mission.**

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

# **Visuals**

## ⚠️ The Moment Everything Went Wrong

During Ubuntu installation, instead of choosing **Manual Installation (Something Else)**  
I accidentally clicked **Erase Disk and Install Ubuntu**.

This tiny mistake triggered the entire chain of events —  
disk wiped → partitions gone → Windows lost → bootloader broken → full recovery mission.

<img src="https://github.com/user-attachments/assets/a5a2d4ad-66d7-4f69-8bf5-ca6a055ab2a4" width="300">

---


## 🔄 When GRUB Replaced Windows Boot Manager

After the accidental erase, I rebooted the laptop expecting the usual dual-boot menu.  
But instead, I was greeted with this screen — **only Ubuntu was left**.

Windows Boot Manager had been completely overwritten from the EFI entries.

This is when I realized:
**The bootloader was broken, and Windows wasn't booting anymore.**

<img src="https://github.com/user-attachments/assets/b11a7746-e3f7-4723-ad3a-2ac2d758d9a0" width="300">

---


## USB Failed: Missing Partition Table  
During my recovery, even the USB refused to cooperate.

Balena Etcher threw this error:

> **“Missing partition table — this does not appear to be a bootable image.”**

This confirmed that my USB had also become corrupted during the initial disk wipe and retries.  
I had to completely reformat the USB, repair its partition table, and later switch to **Ventoy** for stable multi-boot setups.

<img src="https://github.com/user-attachments/assets/df42423e-040b-49d1-8488-b9796b5870d3" width="300">

---


## 🔍 Recovering Lost Data with PhotoRec (Raw File Carving)

Once TestDisk confirmed that my partitions were corrupted,  
I moved to the next tool — **PhotoRec**, which ignores the filesystem entirely  
and recovers files by scanning raw disk sectors.

Here is the exact moment when PhotoRec started pulling my data back:

<img src="https://github.com/user-attachments/assets/574322db-4cff-40c3-a4f3-86f2374949b3" width="300">


PhotoRec recovered:

- **33,204 text files**  
- **5,093 PNGs**  
- **4,781 JPGs**  
- **2,568 EXEs**  
- **2,463 GZIP archives**  
- **809 ELF binaries**  
- **491 SQLite DBs**  
- **384 .lnk files**  
- **2,000+ other file types**

Total recovered: **59,807 files** 🤯

This step alone took hours — but it proved that even a wiped disk  
still holds recoverable data if scanned at the sector level.

---


## 📂 The Chaos of Raw File Recovery (Hundreds of recup_dir Folders)

After PhotoRec finished scanning my SSD sector-by-sector,  
it dumped **all recovered files** into hundreds of folders named `recup_dir.X`.

This is exactly how raw file carving looks when your filesystem structure is gone:

<img src="https://github.com/user-attachments/assets/d18ef499-410c-4c5e-b0d8-8f5fd05610e0" width="300">


Each folder contained random mixes of:

- text files  
- images  
- executables  
- databases  
- XMLs  
- thumbnails  
- system binaries  
- and many files without extensions

Nothing was organized.  
Nothing had filenames.  
And yet — this is REAL recovery.

Raw carving ignores the filesystem and extracts whatever bytes it can reconstruct.  
This stage taught me more about **file signatures, block recovery, and filesystem metadata**  
than any classroom ever could.

---


##  When the GUI Broke Completely (GDM Failure)

Right when I thought recovery was going well, Ubuntu suddenly refused to boot  
into the desktop environment. Instead of GNOME, I was greeted with this:


<img src="https://github.com/user-attachments/assets/f0c7b789-0651-4bb3-b048-6357a075e22d" width="300">


**GDM (GNOME Display Manager) failed to start**, which meant:

- No desktop  
- No mouse  
- No windows  
- No graphics  

Only a pure **TTY terminal** was available.

This forced me to learn real Linux internals:

- systemd services  
- gdm.service logs  
- switching TTY modes (Ctrl + Alt + F3)  
- reinstalling GPU drivers  
- fixing broken dependencies  
- repairing display manager via `systemctl`  

This moment turned my recovery into a *true* Linux debugging journey.

---


## 🧩 Partition Scan Failed — Filesystem Completely Damaged  

When I ran **TestDisk** to inspect my lost Windows partition, it gave the most painful message any dual-boot user can see:


<img src="https://github.com/user-attachments/assets/c54a1fad-4ccd-4ae3-b4eb-704ddda5580d" width="300">



### TestDisk Output:
> **Can’t open filesystem. Filesystem seems damaged.**

### 💀 What this meant:
- My **entire NTFS filesystem was corrupted**
- Neither Windows nor Ubuntu could read or mount the partition
- The partition table still existed, but the **actual filesystem content was destroyed**
- No directory structure left
- No filenames left  
- Standard recovery was impossible

This was the turning point that forced me to switch from:

**Partition Recovery (TestDisk)**  
to  
**Raw File Carving (PhotoRec)**

### 🔄 Why this step mattered:
This moment confirmed that:

- NTFS metadata = gone  
- File indexes = gone  
- Windows OS partition = unrecoverable  
- Only individual file blocks could be extracted  

This is where the REAL recovery began —  
**brute-forcing thousands of files out of raw disk sectors using PhotoRec.**

---


## 🖥️ Dropped to TTY Mode: Manual Disk Diagnosis

After GDM failed and Ubuntu refused to load the desktop,  
I logged into **TTY (Ctrl + Alt + F3)** and continued recovery entirely through the terminal.

Here’s the moment I authenticated into TTY and started inspecting my disk:


<img src="https://github.com/user-attachments/assets/dc1e59d9-461b-45c0-9e7d-45ea9519e410" width="300">


### 🧵 What I did here:
- Logged in with my username & password
- Used `lsblk` to inspect all disks and partitions
- Verified which partitions survived the crash
- Checked the EFI partition (`nvme1n1p1`)
- Identified the corrupted Ubuntu and Windows partition entries
- Prepared the system for recovery using TestDisk & PhotoRec

This step was CRITICAL because it showed me:

✔ Which partitions were valid  
✔ Which ones were corrupted  
✔ Where Windows & Ubuntu *should* have been  
✔ How badly the disk layout was damaged

From this point forward, EVERYTHING I did was low-level Linux recovery.

---


## 🚀 Ventoy to the Rescue (Bootable USB Finally Working)

After multiple failures with Etcher and a corrupted USB,
**Ventoy saved the day.**

Once I installed Ventoy on the USB, it instantly recognized the Windows ISO:

This was the first moment things started working again —
I could finally boot into the Windows installer and continue the recovery.

<img src="https://github.com/user-attachments/assets/3537eb98-62a0-41bb-95b5-fc20e0065e17" width="300">


---


## 🧩 Rebuilding the Partition Table & Reinstalling Windows

After recovering as many files as possible with TestDisk and PhotoRec,  
I reached the final stage — **reinstalling Windows on the damaged disk**.

When the Windows installer loaded, the partition table looked like this:

<img src="https://github.com/user-attachments/assets/a36188f7-dfb6-4a3c-a035-30188956dfa3" width="300">


Every partition was either *Unknown* or *Unallocated*.  
This confirmed how badly the disk structure had been wiped.

I recreated the necessary partitions and performed a clean Windows installation,  
while keeping my Ubuntu partition untouched.

This step finally restored my dual-boot system back to life.


---

### From wiping entire disk → to recovering data → to rebuilding bootloaders → to achieving a stable dual-boot…

### This project transformed a mistake into mastery. 🔥

