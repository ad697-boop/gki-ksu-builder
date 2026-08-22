# SukiSU 内核构建（Xiaomi 12 / cupid，SM8450，LineageOS 23.2）

在 GitHub Actions 上，用 **LineageOS 官方 SM8450 内核源码**（`LineageOS/android_kernel_xiaomi_sm8450`，
`lineage-23.2` 分支）编译内置 **SukiSU Ultra** 的内核，并直接用原厂 boot.img 重打包成可刷入的 boot 镜像。

## 为什么这么做（背景）

cupid 的所有类原生 ROM（LineageOS/LMODroid 等）用的都是**高通 OSS 内核**，不是真正的 GKI 内核。
版本号里的 `5.10.256-gki-ge682ed2de56f` 是伪装出来的：

- 内核本身是高通 SM8450 内核（`ARCH_WAIPIO/CAPE/DIWALI` 等），不是 Google common 内核；
- `-gki-` 只是 `CONFIG_LOCALVERSION`，为了让 vendor_boot 里的 54 个预编译模块能通过 vermagic 校验加载；
- 所以任何“通用 GKI 内核”（官方 KSU、ShirkNeko、以及本仓库旧的 GKI workflow 产物）刷进去都会无限重启；
- 管理器“修补 boot.img”的 LKM 方式也不会生效：通用 ksu 模块过不了这颗内核的
  vermagic + `CONFIG_MODVERSIONS` 符号校验。

正确做法就是把 SukiSU **编译进这颗高通内核本身**，并保持与 ROM 完全一致的版本字符串
（`5.10.256-gki-ge682ed2de56f`），这样原厂 vendor 模块照样加载，root 也生效。

## 使用步骤

1. 进入仓库 Actions 页，手动运行 **Build SukiSU kernel (SM8450 lineage-23.2 / cupid)**。
2. 默认参数对应 `lineage-23.2-20260820-nightly-cupid-signed`：
   - `rom_url`：ROM 包直链（默认 Princeton 镜像的 20260820 nightly）；
   - `kernel_commit`：内核源码 commit（默认 `e682ed2de56f`，即该 nightly 实际使用的 commit）；
   - `ksu_ref`：SukiSU 分支（默认 `builtin`，非 GKI 内核用的内置 LSM hook 版本）。
3. 等构建完成（内核全量 LTO，约 40-90 分钟），下载产物 `boot-sukiSU-5.10.256-gki-ge682ed2de56f`。
4. 刷入：
   ```bash
   fastboot flash boot boot-sukiSU-5.10.256-gki-ge682ed2de56f.img
   fastboot reboot
   ```
   然后安装 [SukiSU Manager](https://github.com/SukiSU-Ultra/SukiSU-Ultra/releases)。

## 产物说明

- `boot-sukiSU-<版本>.img`：直接刷入 `boot` 分区的成品镜像（只换了内核，ramdisk/header 与原厂一致）；
- `Image`：编译出的原始内核（备用，可自己用 magiskboot 换进别的 boot.img）；
- `stock.config` / `built.config`：原厂内核配置 / 本次构建配置，用于核对差异；
- `boot.img`：原厂 boot 镜像（备份）。

## 工作原理

- 从 ROM 包提取原厂 boot.img，解出原厂内核和 ramdisk；
- 从原厂内核里提取**最终编译配置**（IKCONFIG）作为构建配置，保证与 ROM 完全一致；
- 解析原厂内核的 vermagic（如 `5.10.256-gki-ge682ed2de56f`），把构建版本字符串设成完全相同的值；
- 集成 SukiSU-Ultra `builtin` 分支（LSM hook，不需要 kprobes，也不需要改核心 syscall 文件），
  打开 `CONFIG_KSU=y`（本版暂不开 `KSU_SUSFS`，后续可加）；
- 用与 nightly 相同的 clang（r563880c / clang 21）编译 `Image`；
- 校验构建内核的版本字符串与原厂一致后，用 magiskboot 换入原厂 boot.img 重打包。

## 其它 ROM 怎么用

其它基于同一内核（`android_kernel_xiaomi_sm8450`）的类原生 ROM，只要：

- 把 `rom_url` 换成对应 ROM 包的直链；
- 把 `kernel_commit` 换成该 ROM 实际使用的内核 commit（可在 ROM 内核的
  `uname -r` / vermagic 里看到 `-g<12位commit>`）；

就能编出匹配该 ROM 的 KSU 内核。GKI 内核（6.1/6.6）的设备请勿使用本 workflow。

## 注意

- 旧 workflow `build.yml`（GKI 方案）已被证明在这台设备上无效（刷入无限重启），请勿再使用。
- 刷机前请先备份原厂 boot：`fastboot getvar current-slot` 后
  `fastboot flash boot_<a/b> 原厂boot.img` 可恢复。
- 本 workflow 默认不开 SUSFS（隐藏 root 需要另外给内核打 susfs 补丁），先保证 root 可用。
