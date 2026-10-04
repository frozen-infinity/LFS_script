# LFS 13.1 systemd desktop installer

This is an **experimental source-build installer**, assembled from the official
BLFS 13.1 book with jhalfs. Its 547 recipe steps and wrapper have been checked
for Bash syntax, and known generated configuration problems have been corrected.
It has **not been built and boot-tested on LFS**. A clean, normal desktop is the
intended result, but this is not a verified turnkey distribution installer.
Some build failures may require editing a recipe. jhalfs itself documents this
limitation: https://www.linuxfromscratch.org/alfs/

## Download from GitHub

If Wget is already installed on LFS, download all three files:

```bash
wget https://raw.githubusercontent.com/frozen-infinity/LFS_script/main/lfs-desktop-13.1.sh
wget https://raw.githubusercontent.com/frozen-infinity/LFS_script/main/SHA256SUMS
wget https://raw.githubusercontent.com/frozen-infinity/LFS_script/main/README.md
sha256sum -c SHA256SUMS
bash lfs-desktop-13.1.sh --check
```

A clean LFS installation may have neither Wget nor Curl. In that case, download
these files on another system and copy them to LFS. The installer bundles the
sources needed to build Wget after it has been transferred.

## Run

Copy `lfs-desktop-13.1.sh` onto the LFS machine. Run from a root console in the
booted LFS system, with a working internet connection:

```bash
bash lfs-desktop-13.1.sh --check
bash lfs-desktop-13.1.sh --install --user yourname
```

Use a normal account name instead of `yourname`. An existing account can be used;
its password is set interactively at the end. Do not run in the host OS or a
chroot. The default architecture profile is x86_64. The install is large: reserve
100 GiB free, preferably 16 GiB or more RAM, and expect many hours to several
days of compilation. The enforced minima are 60 GiB free and 8 GiB RAM. Some
packages do not respect the common parallelism limit and may need more memory
or swap. Keep the machine powered and use its local console.

The self-extracting script includes Wget and Sudo source archives, a Mozilla CA
bundle, DejaVu fonts, all build recipes, and their build order. It builds the
bootstrap packages first, then downloads and builds the rest. This handles the
absence of wget/curl on a blank LFS system. It never disables TLS verification.
Bootstrap payloads and firmware use SHA-256; BLFS recipes retain the checksums
published in the book, mostly MD5. Not every supplementary patch/download in
the book has a published checksum. HTTPS is used where specified by upstream;
this is not a fully signed, authenticated supply-chain installation system.

## What is installed

KDE Plasma 6.7.4 and KWin Wayland, KDE Frameworks 6.29.0, Qt 6.11.2, SDDM,
Mesa, Wayland, XWayland, and Xorg for the SDDM greeter. Firefox, Konsole,
Dolphin, Kate, desktop portals, PAM, Polkit, NetworkManager/wpa_supplicant,
PipeWire/WirePlumber, Bluetooth, removable disk and NTFS support, printing,
power management, certificates, fonts, Git, and Curl are included.
Dependencies include additional libraries, services and build tools; see
`plan.txt` inside the extracted bundle or `/var/lib/lfs-desktop/plan.txt`.

Systemd and Shadow are rebuilt with PAM for graphical sessions. The existing
kernel, bootloader, partitioning, hostname and locale defaults are retained.
The script adds an en_US.UTF-8 locale for builds but does not select a new
system language, keyboard layout, timezone, or disk encryption scheme.

Linux-firmware 20260916 is installed by default (a 632 MB compressed download).
Use `--no-firmware` if you deliberately manage firmware separately. Firmware
installation does not regenerate an existing initramfs: if your storage or GPU
driver needs firmware in an initramfs, rebuild it using your boot setup's tools.
CPU microcode updates are not configured by this installer.

## Kernel and graphics requirements

This script **does not build a replacement kernel**. The kernel from the
completed LFS book is retained. A minimal kernel without desktop drivers must
be reconfigured and rebuilt first. Check your kernel configuration for:

- DEVTMPFS and its mount support, TMPFS and TMPFS_POSIX_ACL;
- firmware loader support, INPUT, EVDEV, DRM and DRM_KMS_HELPER;
- your GPU's driver (I915/XE, AMDGPU, NOUVEAU, or a VM graphics driver);
- SND, SND_PCM and the ALSA driver for your sound hardware;
- your Ethernet/Wi-Fi drivers, CFG80211, MAC80211 where needed, and RFKILL;
- Bluetooth and its adapter driver if needed;
- namespaces, cgroups, SECCOMP and the systemd options from the LFS book.

Driver requirements are hardware-specific. This profile uses Mesa's automatic
driver selection. Proprietary NVIDIA drivers are not installed. Intel, AMD,
Nouveau and virtual GPU support still depend on the correct kernel drivers and
firmware. Consult the BLFS hardware guidance:
https://www.linuxfromscratch.org/blfs/view/13.1-systemd/x/mesa.html
https://www.linuxfromscratch.org/blfs/view/13.1-systemd/postlfs/firmware.html

## Privileges, configuration and reboot

The launcher runs as root. Source builds use the locked `lfsdesktopbuild`
account, with temporary passwordless sudo installation privileges. Those
privileges are removed on normal exit or failure. SIGKILL or power loss can
leave `/etc/sudoers.d/lfs-desktop-build` behind; remove it if abandoning the
installation. The desktop user receives password-protected sudo through wheel.

An initial copy of `/etc` is saved under `/var/lib/lfs-desktop`; this is **not a
full system rollback**. Take a filesystem/VM snapshot before installation.
Package installs write directly into `/usr`, `/opt` and `/etc`, including
rebuilding authentication components. No automatic rollback is implemented.

The script configures graphical boot but does not reboot automatically. At the
end, reboot from a local console and choose **Plasma (Wayland)** at SDDM. SDDM's
greeter uses X11; the desktop uses Wayland. NetworkManager takes over networking
at reboot. An Ethernet connection normally uses DHCP. Connect Wi-Fi through the
Plasma network menu. Existing static networkd settings are not translated into
NetworkManager connections; set those up separately if needed. The existing
resolver configuration is backed up before handoff. Root SSH access is not
required, and LDAP server startup is removed from the dependency recipe.

## Failures and maintenance

Rerun the same command to resume. Completed recipes are tracked by SHA-256 in
`/var/lib/lfs-desktop/done`. Logs are in `/var/lib/lfs-desktop/logs`, downloads
and build trees in `/var/cache/lfs-desktop`. If a package fails, inspect its log
and corresponding editable script in `/var/lib/lfs-desktop/recipes`. Correct
the script and rerun. The failed recipe starts over from its source archive.
Changes to a completed recipe rebuild that recipe; dependent packages are not
automatically rebuilt. Retrying is not transactional.

The completion checks verify essential binaries and PAM configuration, but do
not prove that hardware, a graphical login, sound or networking work at runtime.
There is no distribution package repository, automatic update service, or
general package uninstaller. Ongoing security updates remain your responsibility
through LFS/BLFS. Installing KDE Discover does not create a binary package
repository; it is not included as a replacement for package management.

## Provenance

The book versions, source commits, recipe changes and payload checksums are
included in `provenance.txt`, `recipe-changes.txt`, and `bootstrap/SHA256SUMS`.
Official book: https://www.linuxfromscratch.org/blfs/view/13.1-systemd/
Generator: https://git.linuxfromscratch.org/jhalfs.git

Build recipes are derived from BLFS shell instructions under the MIT license
specified by the book. Bootstrap archives retain their upstream licenses.

## Bootstrap correction

This revision uses Shadow's `su` to run builds as the locked build account.
The original revision incorrectly required `runuser`, which the LFS 13.1
Util-linux instructions deliberately disable. You do not need to rebuild
Util-linux to address that error. Argument quoting and all bundled shell
scripts were checked; this revision still has not been installed or boot-tested
on LFS.
