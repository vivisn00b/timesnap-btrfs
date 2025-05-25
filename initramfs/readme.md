# 🧊 Booting Btrfs Snapshots in Read-Only Mode with OverlayFS

Booting a **read-only Btrfs snapshot** can be tricky — especially when parts of the system (like `/var`) require write access during boot. A clean and elegant solution is to use **OverlayFS**, available since **Linux kernel 3.18**.

OverlayFS allows you to boot snapshots as **non-persistent, Live CD-like environments**:

- ✅ Snapshot remains **immutable**
- 🧠 All writes are redirected to a **RAM-backed overlay**
- 🔄 All changes are **discarded at reboot**
- 🛠️ Fixes issues with write access (e.g. `/var`, logs, temporary files)

> ⚠️ Requires modifying your **initramfs**. Snapshots that don't include this change (unless using a separate `/boot` partition) won't benefit from the feature.

---

## 📦 Installation Guide

### 🐧 Arch Linux

#### 1. Install the Hook

- Copy:
  - `overlay_snap_ro-install` → `/etc/initcpio/install/timesnap-grub-btrfs-overlayfs`
  - `overlay_snap_ro-hook` → `/etc/initcpio/hooks/timesnap-grub-btrfs-overlayfs`
- Rename both files **exactly the same**:
  - Example:
    ```text
    overlay_snap_ro-install → timesnap-grub-btrfs-overlayfs
    overlay_snap_ro-hook → timesnap-grub-btrfs-overlayfs
    ```

#### 2. Update `mkinitcpio.conf`

Edit `/etc/mkinitcpio.conf` and append the custom hook:

```bash
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck timesnap-grub-btrfs-overlayfs)
```

> Make sure the hook name matches the renamed files exactly.

#### 3. Regenerate initramfs

```bash
sudo mkinitcpio -P
```

- `-P` processes all presets in `/etc/mkinitcpio.d`.

---

### 🌀 Dracut-Based Distros (Fedora, RHEL, etc.)

Dracut users can use kernel parameters to activate OverlayFS:

- Boot **read-only** snapshot:
  ```text
  rd.live.overlay.readonly=1
  ```

- Boot snapshot in **Live CD-like mode** (changes lost at reboot):
  ```text
  rd.live.overlay.overlayfs=1
  ```

#### With `timesnap-grub-btrfs`:

1. Set the kernel parameter:

```bash
TIMESNAP_GRUB_BTRFS_SNAPSHOT_KERNEL_PARAMETERS="rd.live.overlay.overlayfs=1"
```

2. Regenerate the GRUB submenu for snapshots:

```bash
sudo /etc/grub.d/41_snapshots-btrfs
```

---

### 📚 Other Distributions

For other distros:

- Check your distribution's documentation for customizing **initramfs**.
- Feel free to contribute instructions to this README via pull request!

---

## 🧪 Behavior Summary

| Feature              | Description                                    |
|----------------------|------------------------------------------------|
| Snapshot State       | Immutable / Read-only                          |
| Overlay Storage      | RAM (tmpfs)                                    |
| Persistence          | None — all changes lost on reboot              |
| Use Case             | Safe testing, troubleshooting, temporary boot  |

---

## 🙌 Contributing

If you’ve successfully set this up on a different distribution or improved the process:
- Open a PR with your notes or script
- File an issue with your suggestions

Let’s make booting snapshots safer and simpler across all distros!
