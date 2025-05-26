![GitHub Release](https://img.shields.io/github/v/release/vivisn00b/timesnap-btrfs)
![GitHub License](https://img.shields.io/github/license/vivisn00b/timesnap-btrfs)
![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/vivisn00b/timesnap-btrfs/super-linter.yml)
![Maintenance](https://img.shields.io/maintenance/yes/2025)

## 💻 `timesnap-btrfs` 
`timesnap-btrfs` is a lightweight shell utility packaged for Arch Linux systems using Btrfs and Timeshift; it automatically creates a Timeshift snapshot before each system upgrade and adds it to the GRUB menu, providing seamless rollback and recovery specifically tailored for Timeshift users.

#### ⚠️ Warning: Using Read-Only Snapshots Can Be Challenging
If you plan to boot from read-only snapshots, it is important that directories like `/var/log` or even the entire `/var` reside on separate subvolumes. Without this setup, your snapshots need to be writable to function properly.

This project provides its own solution to handle these challenges. For comprehensive instructions, see the [initramfs documentation](https://github.com/vivisn00b/timesnap-btrfs/blob/main/initramfs/readme.md).

- - -
### ✨ Features:
- 📸 Automatically **creates a Timeshift snapshot** before `pacman` upgrades.
- 📂 Specially made to work only with **Timeshift in Btrfs mode**.
- 🧹 Optionally **deletes older snapshots** based on your configuration.
- 🔁 Optionally **updates GRUB** after snapshot creation.
- 🛠️ Simple and **fully configurable** via config files.
- 📦 **Integrates with pacman** using a hook for fully automated snapshot management.
- 📋 **Automatically lists all existing snapshots** on the Btrfs root partition.
- 🧾 Creates matching **GRUB boot menu entries** for Timeshift snapshots using snapshot metadata like **comments, tags, and types** for meaningful labels.
- ⚙️ Automatically **regenerates** `grub.cfg` if used with the optional systems service.

- - -
### 🛠️ Installation:
#### 🔨 Manual Installation
1. Run `make install` to install the script.
2. Run `make help` to view all available options and usage modes.

#### 📦 Dependencies
Make sure the following packages are installed on your system:
- [btrfs-progs](https://archlinux.org/packages/core/x86_64/btrfs-progs/) – for managing Btrfs filesystems
- [grub](https://archlinux.org/packages/core/x86_64/grub/) – GRUB bootloader
- [bash ≥ 4](https://archlinux.org/packages/core/x86_64/bash/) – required shell
- [gawk](https://archlinux.org/packages/core/x86_64/gawk/) – for robust text processing
- [inotify-tools](https://archlinux.org/packages/extra/x86_64/inotify-tools/) – *only required when using the `grub-btrfsd` daemon*

- - -
### ⚙️ Customization:
You have the possibility to modify many parameters in `/etc/default/timesnap/timesnap-grub-btrfs.conf`.
For further information see [config file](https://github.com/vivisn00b/timesnap-btrfs/blob/main/timesnap-grub-btrfs.conf).

#### 🪛 Customization of the `timesnap-gbd` Daemon
`timesnap-btrfs` includes a daemon script called `timesnap-gbd` that **automatically updates the GRUB menu** whenever a snapshot is created or deleted in a monitored directory.

> 🔔 **Note:** You must install [`inotify-tools`](https://archlinux.org/packages/extra/x86_64/inotify-tools/) to use the daemon functionality.

You can customize the daemon's behavior using the following command-line arguments:

#### 🛠️ Available Options
| Option                  | Description                                                                 |
|-------------------------|-----------------------------------------------------------------------------|
| `-c`, `--no-color`      | Disable colored output.                                                     |
| `-l`, `--log-file`      | Specify a custom log file path for output.                                  |
| `-r`, `--recursive`     | Watch snapshot directories recursively.                                     |
| `-s`, `--syslog`        | Enable logging to system syslog.                                            |
| `-t`, `--timeshift-auto`| Auto-detect the dynamic Timeshift snapshot path (for Timeshift ≥ 22.06).    |
| `-v`, `--verbose`       | Enable detailed verbose logging.                                            |
| `-h`, `--help`          | Show help and usage information.                                            |

> 📌 **Note:** When using the `--timeshift-auto` flag, you **do not need to provide a snapshot path manually**. The daemon will dynamically detect and monitor the Timeshift snapshot directory (e.g., `/run/timeshift/$PID/backup/timeshift-btrfs`).

- - - 
### ⏳ Daemon:
`timesnap-gbd` is a daemon that monitors the Timeshift snapshot directory and **automatically updates the GRUB menu** whenever a snapshot is created or deleted. This ensures that your GRUB bootloader is always in sync with the latest Timeshift snapshots, allowing easy boot-time recovery options without manual intervention.

#### 🔧 `timesnap-gbd` Systemd Instructions
To use the daemon as a systemd service, follow the instructions below:

▶️ Start the daemon:
```bash
sudo systemctl start timesnap-gbd
```

🔁 Enable on Boot:
```bash
sudo systemctl enable timesnap-gbd
```

📝 Edit the Service:
```bash
sudo systemctl edit --full timesnap-gbd
```

🔄 Restart After Changes:
``` bash
sudo systemctl restart timesnap-gbd 
```

> 💡 **Tips:**
> You can view your changes with:
> ```bash
> systemctl cat timesnap-gbd
> ```
> To revert all changes to the default service configuration, use:
> ```bash
> systemctl revert timesnap-gbd
> ```

- - -
### 🧰 Troubleshooting:
Having issues with `timesnap-grub-btrfs`? Here are some steps and checks to help you resolve common problems.

#### 📦 Check Installed Version
When reporting bugs or asking for support, knowing which version you're using is helpful. Run one of the following:
```bash
sudo /etc/grub.d/41_snapshots-btrfs --version
```
or
```bash
sudo /usr/bin/timesnap-gbd --help
```
Include this version info when submitting issues for `timesnap-grub-btrfs`.

#### 🗣️ Run the Daemon in Verbose Mode
To debug issues with snapshot detection or GRUB updates, start the daemon with verbose logging:
```bash
sudo /usr/bin/grub-btrfsd --verbose --timeshift-auto
```

>💡 **Tip:** You can also add `--verbose` to your systemd service file for persistent logging:
>```bash
>sudo systemctl edit --full timesnap-gbd
># Add --verbose to the ExecStart line
>```

Restart the service after editing:
```bash
sudo systemctl restart timesnap-gbd
```

#### 🔒 Snapshots Not Showing on LUKS Encrypted Devices?
By default, GRUB does not load modules required for LUKS encryption. If you're using LUKS and snapshot boot entries are missing:
1. Open the grub-btrfs config file:
   ```bash
   sudo nano /etc/default/timesnap/timesnap-grub-btrfs.conf
   ```
2. Enable cryptodisk support:
   ```bash
   TIMESNAP_GRUB_BTRFS_ENABLE_CRYPTODISK="true"
   # Just remove the '#' at the beginning of the line in the config file
   ```
3. Regenerate your GRUB config:
   ```bash
   sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```
This allows GRUB to unlock your encrypted root before listing snapshots.

#### 🧪 Test Manually
You can also test manual GRUB regeneration if snapshots are not appearing:
```bash
sudo /etc/grub.d/41_snapshots-btrfs --timeshift-auto
```
Then regenerate the full GRUB config:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

#### 📝 Still Having Problems?
If issues persist, feel free to [open a new issue](https://github.com/vivisn00b/timesnap-btrfs/issues/new/choose) with detailed logs, version information, and a description of your setup.
