# AGENTS.md —— 给接手 AI/开发者的总纲

## 项目

GitHub Actions 上用 ROM 同源内核源码为小米 12（cupid）编译内置 SukiSU/ReSukiSU（可选 SUSFS）的 boot 镜像，
用原厂 boot.img 重打包，`fastboot flash boot` 即用。

## 三个结论（别走弯路）

1. cupid 内核**不是 GKI**：`5.10.256-gki-ge682ed2de56f` 里的 `gki-` 是 LOCALVERSION 伪装，
   本体是高通 SM8450 OSS 内核（`LineageOS/android_kernel_xiaomi_sm8450`）。
2. 真 GKI 内核刷入**必无限重启**；管理器 LKM 修补不生效（vermagic + modversions CRC 过不了）。
3. 唯一可行：ROM 同源 commit + 同款 clang，把 SukiSU **编译进内核**，版本字符串与原厂逐字节一致。

## 快速开始（Actions 手动运行）

1. `rom_preset` 选 ROM：lineage-23.2 / aviumui-16.2.1 / derpfest-16.2 / custom
2. `preset` 选配置：suki-susfs（推荐）/ suki-stable / resuki-susfs / custom
3. 其它参数仅 `preset=custom` 时手动；下载产物刷 boot。

## 规则

- 中文回答；不刷 GKI、不用管理器 LKM 修补（本设备无效/变砖）。
- 刷前确认原厂 boot 备份存在（产物 zip 里有 `rom_out/boot.img`）。
- 改 workflow 前先读 `docs/02-踩坑记录.md`（坑已踩过）。
- SukiSU builtin **最新代码与 v4.1.3 管理器协议不匹配**（app profile v3/v4），
  SUSFS 构建必须 pin `6c13a069`（workflow 预设已处理）。

## 文档

- `docs/00` 背景与根因 · `docs/01` 构建流程 · `docs/02` 踩坑记录 · `docs/03` 术语表 · `docs/04` 本地状态与恢复
