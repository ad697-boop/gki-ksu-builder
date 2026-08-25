# AGENTS.md —— 给接手本项目的 AI/开发者

## 项目一句话

在 GitHub Actions 上，用 **LineageOS 官方 SM8450 内核源码**为小米 12（cupid）编译**内置 SukiSU Ultra（可选 SUSFS）**的 boot 镜像，并用原厂 boot.img 重打包成可直接 `fastboot flash boot` 的成品，同时可选编译匹配内核的 ReKernel-X（墓碑内核支持）LKM。

## 最重要的三个结论（先读，别走弯路）

1. **cupid 的类原生 ROM 内核不是 GKI 内核**。版本号 `5.10.256-gki-ge682ed2de56f` 里的 `gki-` 是 `CONFIG_LOCALVERSION` 伪装出来的，目的是让 vendor_boot 里预编译的 vendor 模块通过 vermagic 校验。内核本体是高通 SM8450 OSS 内核（`LineageOS/android_kernel_xiaomi_sm8450`）。
2. **所有"通用 GKI 内核"方案在这台设备上必死**：刷任何真 GKI KSU 内核（官方、ShirkNeko、自编 5.10.245 等）都会无限重启；管理器"修补 boot（LKM）"也不生效（通用模块过不了 vermagic + CONFIG_MODVERSIONS 符号 CRC）。
3. **唯一可行路径**：用 ROM 同源 commit 的内核源码，把 SukiSU **编译进内核**（built-in），保持内核版本字符串与原厂完全一致，只替换 boot.img 里的内核。

## 快速开始

1. 仓库 Actions 页手动运行 **Build SukiSU kernel (SM8450 cupid / lineage-23.2 & AviumUI)**。
2. 选择 `rom_preset`（lineage-23.2 / aviumui-16.2.1 / derpfest-16.2 / custom）：
   - `lineage-23.2`（默认）：`lineage-23.2-20260820-nightly-cupid-signed`
   - `aviumui-16.2.1`：`AviumUI-16.2.1-cupid-20260716-Official-GMS.zip`（官方 SourceForge 直链）
   - `derpfest-16.2`：`DerpFest-v16.2-20260723-cupid-Official-Stable`（仓库内 stock_boot boot.img.gz，
     无需外链；填 `rom_url` 可改走完整 ROM 下载）
   - `custom`：自己填 `rom_url`（同一内核的其它 ROM 也可用）
   其它参数：
   - `rom_url`：仅 custom 时生效（预设会覆盖它）
   - `kernel_commit`：`e682ed2de56fd2841ef35741c4d0f03599ffd561`（lineage-23.2 分支，即该 nightly 用的 commit）
   - `ksu_ref`：SukiSU 分支/tag（默认 `v4.1.3`）
   - `ksu_version`：驱动版本号（默认 `40796`，与 v4.1.3 管理器一致）
   - `build_rekernel_x`：默认 `yes`，编译 ReKernel-X LKM 并打包模块
   - `susfs`：默认 `no`；`yes` 时切 builtin 分支 + 打 susfs 补丁 + 开 `CONFIG_KSU_SUSFS`
3. 下载产物 zip，`fastboot flash boot boot-sukiSU-*.img`。

## 工作目录约定

- 内核源码：`kernel/`（LineageOS/android_kernel_xiaomi_sm8450，O=out 构建）
- SukiSU：`kernel/KernelSU/`，软链 `kernel/drivers/kernelsu -> ../KernelSU/kernel`
- susfs4ksu：`susfs4ksu/`（仅 susfs=yes 时克隆）
- ReKernel-X：`kernel/drivers/rekernel_x/`（LKM 必须放内核树内编译）
- 解包产物：`boot_work/`（kernel、ramdisk.cpio、boot.img、stock.config）
- ROM 解包：`rom_out/boot.img`
- 构建日志：`build.log`（内核）、`lkm_build.log`（LKM）、`susfs_patch.log`（susfs 补丁），失败时自动上传为 artifact

## 规则

- 每次回答用中文。
- 不要刷 GKI 镜像、不要用管理器 LKM 修补（本设备无效/变砖）。
- 刷机前必须确认原厂 boot 备份存在（产物 zip 里有 `rom_out/boot.img`）。
- 改 workflow 前先读 `docs/02-踩坑记录.md`，很多"坑"已经踩过并修复。

## 详细文档

- `docs/00-背景与根因.md`：设备/内核真相、为什么其它方案都失败
- `docs/01-构建流程.md`：workflow 逐步骤说明
- `docs/02-踩坑记录.md`：已修复的所有问题（含 commit）
- `docs/03-术语表.md`：vermagic、KMI、LKM、SUSFS 等
- `docs/04-本地工作区与恢复.md`：本机文件、工具、刷机/恢复流程
