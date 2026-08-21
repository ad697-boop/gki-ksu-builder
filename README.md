# GKI 5.10 + KernelSU 构建（适配 LMODroid 6.2 / cupid）

用于编译 android12-5.10 分支、内置 KernelSU 的 GKI 内核。

优先尝试与 ROM 内核完全同源的 commit `g992742f9cdcc`；由于该 commit 只在维护者的私有 fork 中、无法公开获取，
找不到时自动改用官方 2025-12 GKI 发布版（同一 KMI `android12-5.10`、同一安全补丁级别 `2025-12`），
与 ROM 的 vendor 模块兼容，可正常开机。

## 使用步骤

1. 在 GitHub 上新建一个**空仓库**（Public/Private 均可，Public 免费额度更足）。
2. 把本仓库的 `.github/workflows/build.yml` 和本文件推上去（可以直接网页上传，或本地 git push）。
3. 进入仓库 Actions 页，手动运行 **Build KernelSU GKI kernel (android12-5.10 @ g992742f9cdcc)**。
4. 等待约 40-60 分钟，构建完成后在 Actions 运行页下载 `Image-ksu-5.10.245` 产物。

## 产物说明

- `Image`：未压缩内核（与本 ROM 原厂内核格式一致，直接换内核用这个）
- `Image.lz4` / `Image.gz`：压缩格式内核（备用）
- `boot.img`：GKI 通用 boot 镜像（备用，不建议直接用）

## 拿到 Image 之后

需要把 Image 换进原厂 boot.img（保留原厂 ramdisk），流程：

1. `magiskboot unpack boot.img`
2. 用构建出的 `Image` 覆盖 `kernel`
3. `magiskboot repack boot.img`，得到 `new-boot.img`
4. 先 `fastboot boot new-boot.img` 临时测试，正常后再 `fastboot flash boot new-boot.img`

原厂 boot.img 位于 `D:\codex\LMODroid-6.2-20251227-RELEASE-cupid\images\boot.img`，
magiskboot 位于 `D:\codex\magiskboot-win\magiskboot.exe`。

## 说明

- 内核 commit 固定为 `g992742f9cdcc`，与 ROM 的 `5.10.245-gki-g992742f9cdcc` 完全一致，只是加入 KernelSU（`CONFIG_KSU=y`）。
- 不添加 SUSFS/KPM/ZRAM 等额外改动，最大限度保持与原厂内核一致，避免兼容性问题。
