# 第五人格分支（feat/identityv）说明

本分支把 SSMT4 改成面向《第五人格》的可用形态。三个外部来源各走各的通道，互不混装：

| 用途 | 来源 | 交付方式 | 在本分支中的位置 |
| --- | --- | --- | --- |
| 主程序 | `Perxenic-Acid/SSMT4` | 仓库 | 本仓库（基于 `main` 的 `f5d6b7b`） |
| 3Dmigoto 包体（`d3dx.ini`） | `ququ8191/IdentityV-Package` | 按预设从包体仓库下载 | `GAME_PRESET_GITHUB_REPO_MAP.IdentityV` + `getRepoSourcePackage` |
| DX11 DLL（加载器侧） | `StarBobis/ProjectBunny` | release 资产，唯一来源 | `resolve_project_bunny_runtime_source` |
| 模型数据类型 | 用户项目 IDMT 的 `type_identityv2.rs` | 随程序发布 | `src-tauri/src/gametype/identityv2.rs` |

主仓库里既没有 `d3dx.ini` 也没有 `d3d11.dll`：`resources/Games/IdentityV/` 只有 `Icon.png`。
这一点与上游一致 —— 包体和运行时 DLL 都是独立下载的东西。

## 1. 数据类型

`type_identityv2.rs` 原文用 `mmt_core::` 路径引用核心类型，其余内容（5 个游戏类型、
元素顺序、ByteWidth）与 SSMT4 的 `D3D11Element` / `D3D11GameType` 完全兼容。适配只做了
一处：把三条 `use mmt_core::...` 换成 `use crate::...`。

运行期仍然只读 `resources/GameType/IdentityV/*.json`（仓库既有架构决定：类型库随程序发布，
避免用户目录残留旧类型）。新增的 `identityv_json_matches_type_source` 测试把 Rust 定义与
JSON 钉在一起，谁只改一边都会红。实测两者 5 个类型逐元素一致。

## 2. 包体

上游的包体通道是：`GAME_PRESET_GITHUB_REPO_MAP` 把预设映射到一个**包体仓库**，
`get3DMigotoLatestRelease` 取该仓库最新 release 的 zip 资产，`install3DMigotoUpdate`
下载后解压进 3Dmigoto 目录（`extract_zip_archive`，并保留 Runtime DLL）。

第五人格的包体仓库 `ququ8191/IdentityV-Package` 已发布 `v0.0.1`，附件为
`IdentityV-Package-V0.0.1.zip`（d3dx.ini、LICENSE、README.md 都在压缩包根目录）。
所以本分支只做了两处改动：

1. `IdentityV` 从通用的 `StarBobis/MinBase-Package` 改为 `ququ8191/IdentityV-Package`；
2. 该预设用 `^IdentityV-Package-.*\.zip$` 精确挑选发布附件，其余资产（如单文件
   `d3dx.ini`）不会被误当包体；解压仍走通用 `extract_zip_archive`，包内 DLL 被跳过。

旧测试版曾把 `source-<提交哈希>` 当作版本号。升级到正式 Release 时，`isRemoteVersionNewer`
会把"本地 source 哈希、远端语义版本"判为需要更新，避免老用户卡在旧版；同版本、正常升降级
仍按数字段比较。安装成功后 `packageVersion` 写入 Release 标签（如 `v0.0.1`）。

`[Loader]` 段不用手改：SSMT 启动前由 `MigotoManager.patchD3dxForLaunch` 删掉上游的
`loader` 允许列表，写入 `module = d3d11.dll` 与 `target = <用户填的进程路径>`。
第五人格 NeoX3 客户端的进程名是 `dwrg.exe`。

实测：Release ZIP 33228 字节，sha256 `9ea7bd2d…5f8b`；解压后 `d3dx.ini` 位于包体根目录，
包内不含任何 DLL。

## 3. DLL

第五人格的 Runtime DLL **只有 Project Bunny 一个来源**，不做随包回退、不切别的运行时。
`ssmt_runtime.rs` 里这条通道：

- 取 `StarBobis/ProjectBunny` 的 `releases/latest`；
- 只接受名为 `d3d11.dll` 的资产，要求下载地址严格等于
  `https://github.com/StarBobis/ProjectBunny/releases/download/<tag>/d3d11.dll`，大小不超过 64 MB；
- 校验 x64 PE（`MZ`、`PE\0\0`、machine `0x8664`、DLL 特征位）、可读取文件版本、
  SHA-256；当 GitHub 提供资产摘要时校验其一致性；
- 从同一 release 的 `ProjectBunny-PACKAGE-<tag>.zip` 根目录取出 `d3dcompiler_47.dll`
  （只取这一个文件，**不提取** 包里的 `d3dx.ini`——那是 DX12 配置），同样校验大小、
  摘要与 x64 PE。启动器会先准备这个配套编译器，缺它时根本到不了 DLL 部署；
- 缓存到 `<全局配置>/Runtime/project-bunny/x64/<sha256>.dll` + `current.json`，
  写入走原子替换，失败不破坏上一份已验证缓存；DLL 与编译器必须成对通过校验，
  旧格式缓存会触发一次重新获取；
- **缓存拿不到就返回错误**，由 `MigotoManager` 把这轮注入标记为 `unavailable`；
  不会退回随包 DLL。刷新失败但缓存仍有效时，把刷新错误放进 `warning` 提示，
  实际注入的仍然是 Project Bunny。

命令 `resolve_project_bunny_runtime_source` 新增可选 `force`：为 `true` 时跳过 15 分钟检查间隔，
供设置页"检查并更新 Project Bunny"按钮使用；普通启动仍用节流检查。
前端按配置的 `gamePreset`（而非可自定义的配置名称）选择它，其他游戏继续走原来的 SSMT Runtime 通道。第五人格不沿用锁定已有 DLL 和 UPX 缓存分支，以免引入未经验证的其他 DLL。

## 4. 验证

```powershell
# 类型定义 ↔ JSON 一致性（离线可跑，见 .tmp/idv2verify）
cargo run --offline --manifest-path .tmp/idv2verify/Cargo.toml -- `
  identityv-ssmt/SSMT4/src-tauri/resources/GameType/IdentityV
```

期望最后一行：`SUMMARY json_types=5 source_types=5 mismatches=0`。

```powershell
# 包体通道与 DLL 通道的静态检查（不联网部分）
python .tmp/verify_integration.py
```

```powershell
# Rust 语法检查（整包编译需要 crates.io，见“已知边界”）
rustfmt --edition 2021 --check src-tauri/src/commands/ssmt_runtime.rs `
  src-tauri/src/gametype/mod.rs src-tauri/src/gametype/identityv2.rs
```

仓库内的 `cargo test gametype::` 也会跑同一条一致性断言。

## 5. 已知边界

- 前端 `npm run build` 已通过。后端 `cargo check --locked` 在检查专用配置
  `TAURI_CONFIG={"bundle":{"resources":[]}}` 下通过；当前检出缺少正常配置要求的
  `resources/d3d11.dll`、`Run.exe` 等构建资源，因此这不是完整发行包或游戏启动验证。
- Project Bunny 的实际注入效果、`dwrg.exe` 的进程名与 DLL 兼容性需要真实游戏 + FrameAnalysis
  验证；仓库内没有可离线复现的条件。
- 源文件模式只覆盖 `d3dx.ini` 一个文件。如果 IdentityV-Package 以后加入 `Mods/` 或其他配置，
  需要改成下载仓库归档并按目录还原，否则只会更新 ini。
