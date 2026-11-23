# IDMT

> [!IMPORTANT]
> IDMT 是基于 [Perxenic-Acid/SSMT4](https://github.com/Perxenic-Acid/SSMT4) 的**非官方第三方分支**，与 SSMT4 作者无关，也不是官方发布。
> 本项目遵循上游的 **GNU GPL v3** 许可证（见 [LICENSE](LICENSE)），并保留上游版权与许可声明。

IDMT 面向**第五人格（Identity V）**的模型提取、Mod 制作与注入，在上游基础上补充了该游戏的预设、包体与运行时来源。
界面、配置目录、缓存、临时目录与应用更新均为独立实现，可与原版 SSMT4 **同时安装、同时运行**，默认不共享数据。

## 与上游 SSMT4 的关系

| 项目 | 说明 |
| --- | --- |
| 上游 | [Perxenic-Acid/SSMT4](https://github.com/Perxenic-Acid/SSMT4)（GPL-3.0） |
| 本仓库 | IDMT，基于上游源码修改，独立维护 |
| Blender 插件 | **与 SSMT4 共用**，不单独维护：[Perxenic-Acid/TheHerta4](https://github.com/Perxenic-Acid/TheHerta4)；更多蓝图特性的版本见 [xuhuan9102/TheHerta4](https://github.com/xuhuan9102/TheHerta4) |
| 文档 | 上游文档：<https://perxenic-acid.github.io/SSMT4-Documents/> |

Blender 侧插件请直接使用上面的上游项目，IDMT 不修改、不重新分发其二进制。

## 本分支的改动

- 新增第五人格游戏预设、图标、默认背景与模型数据类型；首次运行默认勾选第五人格。
- 包体来源：`ququ8191/IdentityV-Package` 的 Release ZIP，按 `IdentityV-Package-*.zip` 精确匹配资产。
- 运行时（`d3d11.dll`）唯一来源：`StarBobis/ProjectBunny`，含配套 `d3dcompiler_47.dll`。
- 修复贴图预览把坐标尺度不同的模型整体判定为异常并叠红的问题（仅对第五人格放宽判定）。
- 旧版 `source-<提交哈希>` 版本号可正确迁移到正式 Release 并提示更新。
- 应用标识改为 `com.idmt.app`；配置目录 `IDMTGlobalConfigs`；默认缓存 `IDMTCachedFolder`；可执行文件 `IDMT.exe`。
- 关闭上游应用自更新通道（包体与运行时更新不受影响），避免把 SSMT4 的更新装进 IDMT。
- 全局唤醒快捷键改为 `Alt+Shift+F`，避免与并存运行的 SSMT4 冲突。
- 关于页的发布页、使用文档、反馈入口指向本仓库；界面残留的 SSMT 名称已清理，`SSMT Runtime` 等确指上游运行时的名称保留。

详细接入说明见 [docs/identityv-integration.md](docs/identityv-integration.md)。

## 下载

安装包见 [Releases](https://github.com/ququ8191/IDMT/releases)。安装包未签名，Windows 可能提示 SmartScreen。

## 与 SSMT4 并存

- 两者的安装目录、卸载注册项、WebView 数据、全局配置与临时文件均独立。
- 若手动把缓存、3Dmigoto 目录或游戏目标指向同一位置，双方仍会共享该目录；请为 IDMT 单独选择。

## 构建

```powershell
npm install
npm run build
npx tauri build
```

需要 Rust 工具链、Node.js 与 Tauri 的 NSIS 打包依赖。构建产物为 `IDMT.exe` 与 `IDMT_x.y.z_x64-setup.exe`。

## 许可证

本项目以 **GNU GPL v3** 发布，全文见 [LICENSE](LICENSE)。
修改自 SSMT4，原始版权归其作者所有。分发二进制时须同时提供对应源代码，并保留本声明与许可证。

## 免责声明

本项目会向游戏进程注入 DLL，可能触发反作弊或账号风控；请自行评估风险。
仅供个人学习与 Mod 制作使用，请勿用于商业用途或破坏游戏公平性。
