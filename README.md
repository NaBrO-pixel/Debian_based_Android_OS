# LauxenOS — Debian Live Build

A custom Debian-based live OS built for laptops, using Sway as the desktop
and Waydroid to run Android APKs (including Play Store apps) alongside
normal Linux apps.

Target hardware: Dell Latitude E6440 (and similar x86_64 laptops)
Base: Debian 13 "Trixie", amd64
Desktop: Sway (Wayland) + Waybar (status bar)
Android apps: Waydroid with GApps (Play Store + Aurora Store)

Note: auto/config now forces --bootloader grub-efi-amd64. This assumes
you'll boot the E6440 in UEFI mode. If you boot in legacy BIOS mode instead,
remove that line from auto/config or the ISO may not boot.

## Requirements to build

    sudo apt update
    sudo apt install live-build curl

## Build steps

    cd lapdroid-os
    sudo lb clean --purge   # always run this before a rebuild, clears stale cached package versions
    sudo lb build

Produces live-image-amd64.hybrid.iso in this directory.
Takes 30-90 minutes on the E6440. Leave it running.

## Testing the ISO (QEMU)

Test in a VM before touching real hardware:

    sudo apt install qemu-system-x86 qemu-kvm
    qemu-system-x86_64 -m 2048 -enable-kvm -cdrom live-image-amd64.hybrid.iso

## Flashing to USB

    lsblk                          # find your USB device, e.g. /dev/sdb
    sudo dd if=live-image-amd64.hybrid.iso of=/dev/sdX bs=4M status=progress

Boot the E6440 from USB with F12 at the Dell logo.

## Key bindings

    Super+Return    Open terminal (Foot)
    Super+D         App launcher (wofi)
    Super+A         Open Waydroid / Android UI
    Super+L         Lock screen
    Super+Shift+E   Exit / logout
    Super+F         Fullscreen
    Super+1-4       Switch workspaces

## Android / Play Store setup (on first boot)

Waydroid downloads the Android image on first boot (~600MB, needs internet).
Once that finishes, run the setup helper:

    lapdroid-setup-android

This will:
1. Install Aurora Store (lets you download any Play Store app without needing
   Google certification — works immediately, no account needed)
2. Print step-by-step instructions to enable the full official Google Play
   Store if you want it (requires a Google account + one-time certification)

### Aurora Store vs Play Store

| Feature                  | Aurora Store | Google Play Store |
|--------------------------|-------------|-------------------|
| No Google account needed | YES         | No                |
| Works immediately        | YES         | Needs certification|
| Full app catalogue       | YES         | YES               |
| Automatic app updates    | YES         | YES               |
| Google account features  | No          | YES               |

For most users Aurora Store is enough. Add official Play Store only if you
need Google account features (purchases, subscriptions, etc.)

## Project layout

    config/
      package-lists/
        desktop.list.chroot           Sway, Waybar, Firefox, etc.
        waydroid-deps.list.chroot     Waydroid runtime dependencies
      includes.chroot/
        etc/skel/.config/sway/        Default Sway config (keybindings, autostart)
        etc/skel/.config/waybar/      Default Waybar config + styling
        usr/share/lapdroid-os/apks/   Aurora Store APK (pre-downloaded at build)
      hooks/live/
        0100-install-waydroid.hook.chroot   Installs Waydroid + GApps at build time
    auto/
      config                          live-build target settings (Trixie, amd64)
    README.md                         This file

## Next steps

- Add a custom wallpaper: drop a .jpg/.png into
  config/includes.chroot/usr/share/backgrounds/ and update the sway config
  output line to reference it
- Add a Plymouth boot splash for your own branding at startup
- Add Calamares installer to turn the live ISO into a "click to install" distro
- Once the base is stable, consider adding your Lapdroid GTK4 shell as the
  default launcher on top of Sway
