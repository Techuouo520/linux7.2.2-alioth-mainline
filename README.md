# 适用于小米 alioth 的 Linux 7.2.2

[简体中文](README.md) | [English](README_EN.md)

适用于 Redmi K40 / POCO F3（alioth）的 Linux 7.2.2 主线内核移植。

启动链：原厂 ABL → Android boot.img（header v2：Image.gz + DTB + initramfs）→ userdata 分区上的 Ubuntu 26.04 rootfs → systemd → GNOME。

## 依赖

```bash
sudo apt install bc bison clang cmake device-tree-compiler dwarves flex libelf-dev libssl-dev lld llvm make mkbootimg android-sdk-libsparse-utils qemu-user qemu-user-binfmt
```

## 编译

```bash
git clone https://github.com/Techuouo520/linux7.2.2-alioth-mainline.git
cd linux7.2.2-alioth-mainline
cmake -S . -B build
cmake --build build --target kernel
```

| 目标 | 作用 | 产物 |
| --- | --- | --- |
| `kernel` | 编译 Image.gz、模块、DTB | `build/kernel/arch/arm64/boot/` |
| `modules` | 安装模块到暂存目录 | `build/modules/lib/modules/7.2.2-alioth` |
| `rootfs` | 构建/刷新 Ubuntu rootfs 与 initramfs（需要 sudo，依赖本地 alioth/ 脚本，不在仓库中） | `build/rootfs.simg`、`build/boot/initrd.img` |
| `bootimg` | 打包 boot.img | `build/boot/boot.img` |

## 刷入

写入 userdata 会清空安卓数据。先写 rootfs，再用 `fastboot boot` 临时启动验证：

```bash
fastboot flash userdata build/rootfs.simg
```

```bash
fastboot boot build/boot/boot.img
```

确认能启动后再固化 boot：

```bash
fastboot flash boot_a build/boot/boot.img
```

## 许可证

GPL-2.0-only
