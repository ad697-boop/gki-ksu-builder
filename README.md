# SukiSU/ReSukiSU 内核构建（Xiaomi 12 cupid）

在 GitHub Actions 上，用 ROM 同源的 LineageOS SM8450 内核源码编译内置 SukiSU/ReSukiSU 的内核，
用原厂 boot.img 重打包成可刷入的 boot 镜像。支持 LineageOS / AviumUI / DerpFest。

> 接手者必读：[AGENTS.md](AGENTS.md)；细节见 docs/01 构建流程、02 踩坑记录、03 术语表、04 本地状态。

## 为什么不能刷 GKI（先读）

cupid 的类原生 ROM 用的都是**高通 OSS 内核**，版本号 `5.10.256-gki-ge682ed2de56f` 里的 `gki-`
只是 `CONFIG_LOCALVERSION` 伪装，为了让 vendor 模块过 vermagic 校验。真 GKI 内核（官方/ShirkNeko/自编）
刷入**必无限重启**；管理器 LKM 修补也不生效（vermagic + modversions CRC 过不了）。
唯一可行：把 SukiSU **编译进这颗高通内核**，版本字符串与原厂完全一致。

## 使用（Actions 页手动运行）

只需选两个下拉框：

1. **`rom_preset`**（ROM）：`lineage-23.2` / `aviumui-16.2.1` / `derpfest-16.2` / `custom`
   - DerpFest 用仓库内 `stock_boot`，无需外链；`custom` 需填 `rom_url` 直链
2. **`preset`**（配置）：`suki-susfs`（推荐）/ `suki-stable` / `resuki-susfs` / `custom`
   - 预设自动填充 root_impl / ksu_ref / ksu_version / susfs / ksu_commit / ReKernel-X 等参数
   - 其它参数仅 `preset=custom` 时手动，平时不动

构建约 40-90 分钟，产物 `boot-sukiSU-<rom>-5.10.256-gki-ge682ed2de56f.img`：

```bash
fastboot flash boot boot-sukiSU-<rom>-5.10.256-gki-ge682ed2de56f.img
fastboot reboot
```

## 产物

- `boot-sukiSU-<rom>-*.img`：成品（只换内核，ramdisk/header 保持原厂）
- `Image`：原始内核（备用）
- `rom_out/boot.img`：原厂 boot 备份
- `stock.config` / `built.config`：原厂/构建配置对照
- `ReKernel-X-*.zip` + `rekernel_x.ko`：墓碑支持模块（必须重新编译，官方 ko vermagic 不匹配）

## 其它 ROM

同一内核（`android_kernel_xiaomi_sm8450`）的类原生 ROM：`rom_preset=custom` + `rom_url` 直链 +
`kernel_commit` 换成该 ROM 实际 commit（`uname -r` 里 `-g<sha>`）。GKI 设备勿用本 workflow。

## 注意

- 刷前备份原厂 boot（产物里有 `rom_out/boot.img`）。
- 默认预设 `suki-susfs` 已开 SUSFS；刷后管理器显示驱动 40796。
