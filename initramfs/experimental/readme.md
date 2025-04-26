# Overlay Snapshot RW Boot (Arch Linux)

Boot directly into a **specific Btrfs snapshot** and **make permanent changes** to that snapshot, without affecting the original root filesystem.

This system is designed for users who want a clean recovery, rollback, or testing environment on Btrfs, similar to what openSUSE and other snapshot-capable distros offer.

---
## ⚠️ WARNING — Experimental Feature

This setup is **experimental** and should be used **with caution**.  
Improper use **can permanently modify snapshots** and may **lead to system instability** or **data loss** if mishandled.

You are strongly advised to:
- **Test in a virtual machine first**
- **Backup your important data**
- **Understand how Btrfs subvolumes and snapshots work**

> **You have been warned! Proceed carefully.**

---

## ✨ Features

- **No Overlayfs or tmpfs** involved.
- **Snapshot is remounted read-write** during early boot.
- **All changes are made directly** into the snapshot you booted into.
- **Root (@) filesystem remains untouched** unless manually changed.
- **Fast and minimal** impact on boot time.

---

## 🛠️ Installation

1. **Clone or Download** this repository.

2. **Copy the Hook and Install Files**:
   ```bash
   sudo cp hooks/overlay_snap_rw.hook /usr/lib/initcpio/hooks/
   sudo cp hooks/overlay_snap_rw.install /usr/lib/initcpio/install/
   ```

3. **Edit `/etc/mkinitcpio.conf`**:
   Add `overlay_snap_rw` **after** `filesystems` in the `HOOKS=()` array:
   ```bash
   HOOKS=(base udev autodetect modconf block filesystems overlay_snap_rw keyboard fsck)
   ```

4. **Rebuild the initramfs**:
   ```bash
   sudo mkinitcpio -P
   ```

5. **Update your GRUB configuration** if necessary:
   - Ensure you have menu entries that allow booting into specific Btrfs snapshots (subvolumes like `@snapshots/XYZ`).
   - You can use `btrfs subvolume list /` to see available snapshots.

6. **Reboot** and select the snapshot you want to boot into.

---

## 📋 Notes

- The snapshot must be a valid Btrfs subvolume and must be bootable.
- If the snapshot is already mounted read-write, this hook does nothing.
- This setup does **not** use any temporary RAM overlay: changes are **persistent** immediately.
- It is your responsibility to ensure you are modifying the correct snapshot.
- Consider making manual backups before testing!

---

## 💬 Help

During boot, you can check the logs:
```bash
journalctl -b | grep overlay_snap_rw
```
This shows whether the snapshot was detected and whether remounting succeeded.
