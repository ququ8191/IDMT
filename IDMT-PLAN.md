# IDMT 独立化计划

目标：IDMT 与 SSMT 可并存，默认不共享安装目录、应用标识、全局配置、缓存、插件、运行时或应用更新通道；不自动引用或迁移旧配置。

1. [完成] 前后端名称统一为 IDMT（含 identifier `com.idmt.app`、`IDMTGlobalConfigs`、`IDMTCachedFolder`、临时目录、界面文案）；`npx vue-tsc --noEmit` 退出 0，`cargo check --locked --lib` 退出 0。
2. [完成] 应用二进制改为 `IDMT.exe`（Cargo `[[bin]]` + `default-run` + `launch-dev.ps1`）；NSIS 产出 `IDMT_4.2.2_x64-setup.exe`，sha256 `F1E871CC...BF811E1`，安装后为 `IDMT.exe`。
3. [完成] 并存实测：静默安装到独立目录后，`IDMT.exe`（PID 49624，标题 IDMT）与 `ssmt4.exe`（PID 47652，标题 SSMT4）同时运行。
4. [完成] 数据隔离实测：`%LOCALAPPDATA%\IDMTGlobalConfigs`、`%LOCALAPPDATA%\com.idmt.app`（WebView）独立创建；第五人格 `Background.png` 与 `Icon.png` 正常落入 IDMT 配置目录；卸载注册键分别为 `IDMT` 与 `ssmt4`。
5. [完成] SSMT 侧未被写入：`SSMT4GlobalConfigs` 仅在 SSMT 自身启动时更新了自己的 `settings.json`，无新增文件。IDMT 新建的 `settings.json` 中 `DBMTWorkFolder` 为空，未继承 SSMT 缓存路径。
6. [未验证] 双向卸载（先卸 IDMT 或先卸 SSMT 后另一方可否继续使用）尚未执行。
7. [限制] 若手动选择同一个缓存、3Dmigoto 目录或游戏目标，两应用仍可能共享这些明确选择的目录；当前仅提示，未强制阻止。
8. [待清理] 早期误命名产生的 `%LOCALAPPDATA%\IDMIGlobalConfigs`、`%LOCALAPPDATA%\com.idmi.app` 与测试安装目录仍在磁盘上，IDMT 不再引用；未自动删除。
