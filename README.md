# 适用于小米 alioth 的 Linux 7.2.2

[简体中文](README.md) | [English](README_EN.md)

适用于 Redmi K40 / POCO F3（alioth）的 Linux 7.2.2 主线内核移植。

## 编译

安装依赖：

```bash
sudo apt install bc bison clang cmake device-tree-compiler dwarves flex libelf-dev libssl-dev lld llvm make
```

克隆仓库：

```bash
git clone https://github.com/Techuouo520/linux7.2.2-alioth-mainline.git
cd linux7.2.2-alioth-mainline
```

编译：

```bash
mkdir build
cd build
cmake ..
cmake --build .
```

使用 `arch/arm64/configs/alioth_defconfig`。

编译产物：

```text
build/kernel/arch/arm64/boot/Image
build/kernel/arch/arm64/boot/dts/qcom/sm8250-xiaomi-alioth.dtb
```

## 许可证

GPL-2.0-only
