# LuxenOS — Debian Live Build

LuxenOS is a Debian-based live OS for x86_64 laptops. It pairs the Sway
Wayland desktop with Waydroid so Android APKs, Aurora Store apps, and optional
Google Play Store apps can run alongside regular Linux applications.

## Release: 2026.07 improved live-build layout

This release focuses on making the project buildable and easier to maintain:

- Live-build configuration now lives in `auto/config`, where `lb config` expects
  it.
- Package lists now live under `config/package-lists/`, where live-build picks
  them up during chroot creation.
- The Waydroid installation hook now lives under `config/hooks/live/`, matching
  live-build hook discovery.
- The Waydroid hook is safer on Debian `/bin/sh`, validates downloads, retries
  transient network failures, and installs an Android setup helper for first
  boot.

## Target

- Hardware: Dell Latitude E6440 and similar amd64 laptops
- Base: Debian 13 "Trixie"
- Desktop: Sway + Waybar
- Android runtime: Waydroid with GApps image initialization and Aurora Store
  fallback
- Boot mode: UEFI via `grub-efi-amd64`

> If you need legacy BIOS boot, remove or override `--bootloader grub-efi-amd64`
> in `auto/config` before building.

## Requirements to build

```sh
sudo apt update
sudo apt install live-build curl
```

## Build steps

From the repository root:

```sh
sudo lb clean --purge
lb config
sudo lb config
sudo lb build
```

The build produces `live-image-amd64.hybrid.iso` in this directory. A full build
can take 30-90 minutes on older laptops.

## Testing the ISO with QEMU

```sh
sudo apt install qemu-system-x86 qemu-kvm
qemu-system-x86_64 -m 4096 -enable-kvm -cdrom live-image-amd64.hybrid.iso
```

## Flashing to USB

```sh
lsblk
sudo dd if=live-image-amd64.hybrid.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

Replace `/dev/sdX` with the whole USB device, not a partition. On Dell systems,
press F12 at the logo to choose the USB boot device.

## Android / Play Store setup on first boot

Waydroid downloads and initializes its Android image on first boot, so an
internet connection is required. After initialization, run:

```sh
lapdroid-setup-android
```

The helper installs Aurora Store if the bundled APK is available and prints the
optional Google Play certification steps for users who want official Play Store
sign-in.

### Aurora Store vs Play Store

| Feature | Aurora Store | Google Play Store |
| --- | --- | --- |
| No Google account needed | Yes | No |
| Works immediately | Yes | Needs device certification |
| Broad app catalogue | Yes | Yes |
| Automatic app updates | Yes | Yes |
| Google purchases/subscriptions | No | Yes |

For most users, Aurora Store is enough. Add official Play Store only if you need
Google account features such as purchases or subscriptions.

## Key bindings

| Shortcut | Action |
| --- | --- |
| Super+Return | Open terminal (Foot) |
| Super+D | App launcher (wofi) |
| Super+A | Open Waydroid / Android UI |
| Super+Shift+A | Run Android setup helper |
| Super+Shift+H | Run LuxenOS health check |
| Super+L | Lock screen |
| Super+Shift+E | Exit / logout |
| Super+F | Fullscreen |
| Super+1-4 | Switch workspaces |

## Project layout

```text
auto/config                                  live-build target settings
config/package-lists/desktop.list.chroot     Sway, Waybar, Firefox, desktop apps, polish
config/package-lists/waydroid-deps.list.chroot Waydroid runtime dependencies
config/includes.chroot/etc/skel/.config/sway/config
                                             Default Sway session config
config/includes.chroot/etc/skel/.config/waybar/
                                             Default Waybar config and style
config/hooks/live/0100-install-waydroid.hook.chroot
                                             Waydroid, GApps, Aurora setup hook
config/hooks/live/0200-configure-premium-defaults.hook.chroot
                                             Security, app, firmware, and polish defaults
config/includes.chroot/usr/local/bin/luxenos-health-check
                                             Post-boot diagnostics helper
tools/validate-release-tree                 Merge/layout validation helper
config/package-lists/desktop.list.chroot     Sway, Waybar, Firefox, desktop apps
config/package-lists/waydroid-deps.list.chroot Waydroid runtime dependencies
config/hooks/live/0100-install-waydroid.hook.chroot
                                             Waydroid, GApps, Aurora setup hook
README.md                                    Build and release documentation
```

## Next improvements

- Add a custom wallpaper in `config/includes.chroot/usr/share/backgrounds/` and
  reference it from the Sway config.
- Add a Plymouth boot splash for branded startup.
- Add Calamares for a graphical installer.
- Add the Lapdroid GTK4 shell as the default launcher once the base is stable.
