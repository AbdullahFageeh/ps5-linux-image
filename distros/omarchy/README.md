# omarchy

[Omarchy](https://omarchy.org) — DHH's opinionated Arch + Hyprland desktop —
built on PS5 hardware with our `linux-ps5` kernel.

## How it is built

Omarchy's official installer is an ISO wizard (`archinstall` + the `install/`
scripts from [omacom/omarchy](https://github.com/omacom/omarchy)); there is no
supported way to run it headlessly against a pre-made rootfs, so this distro
does **not** reuse their artifacts. Instead `distros/omarchy/image.yaml` mirrors
the runtime half of Omarchy's package lists
(`install/omarchy-base.packages` + `install/omarchy-other.packages`) onto the
same Arch `distro builder` recipe the Arch image uses:

1. Arch bootstrap → pacman installs the Omarchy package set (Hyprland, Quickshell,
   waybar-style bar tooling, Chromium, foot, Neovim, PipeWire, Docker, …).
2. Drop `linux-ps5` (`pacman -U`) and rebuild the initramfs without `autodetect`.
3. tty1 autologin as `ps5` → `exec Hyprland`.

Left out on purpose:

- `linux-omarchy` / `linux-omarchy-headers` / `linux-t2` — the PS5 image ships
  its own patched kernel.
- DKMS trees for other machines (`nvidia*`, `broadcom-wl-dkms`, `asusctl`,
  `tuxedo-drivers`, `t2fanrd`, `macbook12-spi-driver-dkms`) — they cannot build
  against `linux-ps5`, and pulling them in makes the pacman transaction fail.
- Boot/OS-management pieces of the ISO image (`limine`, `plymouth`, `snapper`,
  `btrfs-progs`) — PS5 images boot through kexec with a FAT32 EFI partition and
  an ext4 root.
- Intel/Apple-only firmware and tools.

## Build locally

```bash
./build_image.sh --distro omarchy
```

Image size defaults to the shared 12 GB. The `.img.xz` lands in
`output/ps5-omarchy.img.xz`.

## Caveats

- `omarchy`'s own theme/config repository (`~/.local/share/omarchy`) is not
  installed; this is the package-level desktop (Hyprland + the apps), not the
  full Omarchy dotfiles experience. Running Omarchy's installer on the booted
  PS5 should layer its configs on top, as it would on any Arch install.
- The PS5 Oberon GPU has no working hardware video decode in mainline Mesa;
  Chromium will software-decode.
- Like every PS5 image here, default credentials are `ps5` / `ps5` with
  passwordless sudo. Change them before exposing the machine to a network.
