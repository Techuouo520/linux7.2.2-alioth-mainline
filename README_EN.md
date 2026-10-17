# Linux 7.2.2 for Xiaomi alioth

[简体中文](README.md) | [English](README_EN.md)

Linux 7.2.2 mainline kernel port for the Redmi K40 / POCO F3 (alioth).

Boot chain: stock ABL → Android boot.img (header v2: Image.gz + DTB + initramfs) → Ubuntu 26.04 rootfs on the userdata partition → systemd → GNOME.

## Dependencies

```bash
sudo apt install bc bison clang cmake device-tree-compiler dwarves flex libelf-dev libssl-dev lld llvm make mkbootimg android-sdk-libsparse-utils qemu-user qemu-user-binfmt
```

## Building

```bash
git clone https://github.com/Techuouo520/linux7.2.2-alioth-mainline.git
cd linux7.2.2-alioth-mainline
cmake -S . -B build
cmake --build build --target kernel
```

| Target | Purpose | Output |
| --- | --- | --- |
| `kernel` | Build Image.gz, modules and DTB | `build/kernel/arch/arm64/boot/` |
| `modules` | Install modules into a staging dir | `build/modules/lib/modules/7.2.2-alioth` |
| `rootfs` | Build/refresh the Ubuntu rootfs and initramfs (needs sudo) | `build/rootfs.simg`, `build/boot/initrd.img` |
| `bootimg` | Package boot.img | `build/boot/boot.img` |

Rootfs settings live in `alioth/rootfs.conf` (base tarball, firmware path, user name and initial password, packages). The first `rootfs` build installs the full system; later runs only refresh modules and rebuild the initramfs.

## Flashing

Writing userdata erases Android data. Flash the rootfs first, then test with a temporary `fastboot boot`:

```bash
fastboot flash userdata build/rootfs.simg
```

```bash
fastboot boot build/boot/boot.img
```

Once it boots, make boot permanent:

```bash
fastboot flash boot_a build/boot/boot.img
```

## License

GPL-2.0-only
