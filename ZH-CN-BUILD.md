# RHI 简体中文自建版 · 使用与维护说明

## 直接下载（不用自己编）

<https://github.com/DC1024/RHI/releases>（因为是 prerelease，`/releases/latest` 会 404，要点进去选最新的 `v*-zh-cn-*`）。
每个版本两个包，都附同名 `.sha256` 校验文件：

- `RHI-zh-CN-<tag>-win-x64.zip`（约 35MB）—— 完整绿色版
- `RHI-zh-CN-patch-<tag>.zip`（约 34MB）—— 中文补丁包，见下节

产物由 `.github/workflows/build-zh-cn.yml` 在打 `v*` 标签时自动构建发布。

发新版只需：

```bash
git tag -a v2.8.1-zh-cn-3 -m "说明"
git push <remote> v2.8.1-zh-cn-3     # CI 自动构建 + 发 Release
```

## 中文补丁包（给已装官方版的用户）

每次 Release 同时产出 `RHI-zh-CN-patch-<tag>.zip`，**只含一个 `RHI.exe`**。
覆盖进官方安装目录（默认 `C:\Program Files\RHI`）即可变中文，不用重装。

说明文档：`.github/patch/README-patch.md`（随补丁包一起打包）。

### 为什么是 exe 而不是 dll + pri（踩过的坑）

**官方 RHI 是 .NET 单文件发布**（`PublishSingleFile` + `WindowsAppSDKSelfContained`，
见 `RenoDXCommander.csproj` 里 `SelfContained=false` + `WindowsAppSDKSelfContained=true`
以及各 Content 项的 `ExcludeFromSingleFile=true`）。`RHI.dll` 和 `resources.pri`
**都打包在 `RHI.exe` 内部**；安装目录里散落的那两份只是打包残留，**运行时不加载**。

实测证据（官方 `C:\Program Files\RHI\RHI.exe`，104,738,462 字节）：exe 内可搜到 UTF-16 的
`Back to Games` / `Component Updates`（来自 pri 的 XAML）以及 `RenoDXCommander` / `MainWindow`
（来自 RHI.dll），并记录了 `resources.pri`、`RHI.dll` 的 bundle 条目名。
早先只比对磁盘上 dll/pri 的 UTF-16 字串得出的「2 文件补丁」结论是错的，已作废。

因此补丁必须与官方同形态：单文件 exe，用
`dotnet publish -r win-x64 --self-contained false -p:PublishSingleFile=true -p:WindowsAppSDKSelfContained=true`
产出（本地产出 104,896,579 字节，与官方 104,738,462 基本吻合）。

注意：单文件 publish 不会复制 `ExcludeFromSingleFile=true` 的内容文件（7z、ini、图标等），
所以**完整绿色版仍用 portable 目录发布**（`PUBLISH_DIR`），单文件产物只用于补丁包。

## 手工构建

下面是手工构建的方式，CI 用的也是同一套命令。

本目录是把社区 PR [RankFTW/RHI#20](https://github.com/RankFTW/RHI/pull/20)（作者 HexBen123）的简体中文方案，
rebase 到 `RankFTW/RHI` main 分支 v2.8.1 Beta 3 之后的本地构建。官方仓库至今没有合并任何中文 PR。

- 分支：`zh-cn`（本地仓库 `C:\Users\15657.DC-PC\WorkBuddy\2026-10-03-23-22-31\RHI`）
- 基准提交：`1f8b36d`（Patch notes: add NVAPI session lock freeze fix）
- 产物：`artifacts/publish/RHI-win-x64-portable/RHI.exe`（202 个文件，不要单独拷 exe）

## 怎么切中文

启动后 → **Settings（设置）** → 第一张卡片 **Language（语言）**：

| 选项 | 行为 |
| --- | --- |
| Automatic (system language) | 跟随 Windows 显示语言，非 zh-CN 回退英文 |
| English | 强制英文 |
| 简体中文 | 强制中文 |

切换即时生效（会触发 `LocalizationService.LanguageChanged` → 重新遍历整棵可视树），偏好写入 `%LOCALAPPDATA%\RHI\settings.json` 的 `Language` 键。

## 汉化实现

`RenoDXCommander/Services/LocalizationService.cs`：

- **581 条静态词条**的英→中词典，另有 ~73 条正则规则处理运行时动态文案（如 "Downloading X..."、"Updated N file(s)."）。
- 用 `VisualTreeHelper` 递归遍历可视树，按**英文原文**做键匹配，翻译 TextBlock / Run / TextBox 占位符 / ComboBox Header / ToggleSwitch 的 On-Off 文案 / ToolTip / MenuFlyoutItem。
  → XAML 里写的 UI 不需要逐个加 `x:Uid`，后续新增界面大概率自动就有中文。
- 技术名词刻意保留原文：Unreal / Unity / Streamline / RenoDX / ReShade / DXVK / OptiScaler / Lilium HDR。

改动面：`SettingsViewModel`（持久化）、`SettingsHandler`（下拉同步）、`MainWindow.xaml`（语言卡片）、
`MainWindow.UISync`（刷新入口）、以及各 ViewModel 的 `L()` 包装。

## 还没翻的地方（已知缺口）

| 位置 | 原因 |
| --- | --- |
| DetailPanelBuilder 的 Extras / NeuralRendering / NvidiaProfile / DofFix 等模块 | 2026-05 之后上游新增，PR 没碰；新的英文串不在 581 条词典里 |
| 部分组合文案（如 UE-Extended 的更新按钮） | 已补到词典，但类似组合串可能还有漏网的 |
| 安装引导界面 | 走的是 Inno Setup；要做中文安装界面需参考 PR #53 的 `Installer/Languages/ChineseSimplified.isl` |

补翻译只需往 `SimplifiedChinese` 词典加一行 `["英文原文"] = "中文"`，重编译即可。

## 重新构建

本机 .NET 8 SDK 装在用户目录（非全局），构建前先注入环境：

```bash
export PATH="/c/Users/15657.DC-PC/.dotnet:$PATH"
export DOTNET_ROOT="C:\\Users\\15657.DC-PC\\.dotnet"
export DOTNET_CLI_TELEMETRY_OPTOUT=1
```

```bash
dotnet restore RenoDXCommander/RenoDXCommander.csproj
dotnet build RenoDXCommander/RenoDXCommander.csproj -c Release -p:Platform=x64 --no-restore
dotnet test  RenoDXCommander.Tests/RenoDXCommander.Tests.csproj --filter "FullyQualifiedName~Localization"
dotnet publish RenoDXCommander/RenoDXCommander.csproj -c Release -p:Platform=x64 -r win-x64 \
    --self-contained false -o artifacts/publish/RHI-win-x64-portable
```

验证状态：`build 0 error`，`test 10/10 passed`，进程可正常启动。

> 注意：不要用 `PublishSingleFile=true`。实测单文件发布不会把 `ReShade.ini` / `FEATURES.txt` 等
> `ExcludeFromSingleFile=true` 的内容文件复制到输出目录，只能跑便携版（整个目录）。

## 跟进上游

```bash
git checkout zh-cn
git fetch upstream main          # upstream = https://github.com/RankFTW/RHI.git
git merge upstream/main         # 冲突多半落在 LocalizationService 周边，按"保留上游逻辑 + 补翻译调用"处理
```

上游若有大重构，优先保留他们的新结构，翻译可以事后用遍历机制自动覆盖。
