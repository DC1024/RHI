# RHI 简体中文补丁 · 使用说明（覆盖原版安装）

这个压缩包里**只有一个文件：`RHI.exe`**。把已安装的官方 RHI 换成它，就是中文版，
不用重新安装，游戏列表、已装模组、设置全部保留。

## 适用版本

- 官方 RHI **2.8.0 / 2.8.1 Beta 3**（GitHub 安装版 `RHI-Setup.exe`）
- Windows x64
- 系统需装有 **.NET 8 桌面运行时**（官方版同样是框架依赖，能跑官方版就能跑这个）

其它版本请先备份再试；若启动报错，按下面的「回滚」把原 exe 放回去即可。

## 操作步骤

1. **完全退出 RHI**（包括系统托盘图标，托盘里右键 → Quit）。
2. 打开 RHI 的安装目录。默认是 `C:\Program Files\RHI`。
   找不到就在桌面/开始菜单的 RHI 快捷方式上右键 → **打开文件所在的位置**。
3. **备份**原 exe：把 `RHI.exe` 原地重命名为 `RHI.exe.bak`。
4. 把压缩包里的 `RHI.exe` 复制进去**覆盖**（需要管理员权限，点「继续」）。
5. 启动 RHI → **Settings → Language → 简体中文**，界面立刻切换。
   偏好会记在 `%LOCALAPPDATA%\RHI\settings.json`，下次启动自动保持。

> 只想换回英文的话，在 Language 里选 English 即可，不必换 exe。

## 回滚

删掉覆盖进去的 `RHI.exe`，把 `RHI.exe.bak` 改回 `RHI.exe`，就是原来的官方版。

## 为什么只换 exe 这一个文件

官方 RHI 是 **.NET 单文件发布**（`PublishSingleFile` + `WindowsAppSDKSelfContained`）：
`RHI.dll`（主程序集，中文词条表和语言切换逻辑）和 `resources.pri`（编译后的 XAML，
设置页新增的 Language 下拉卡片）**都打包在 `RHI.exe` 内部**。

安装目录里散落的 `RHI.dll` / `resources.pri` 只是打包过程的残留副本，
**运行时根本不会加载** —— 所以只改那两个文件是没效果的，必须整体替换 exe。

本补丁的 exe 与官方用同样参数构建（约 100 MB，自带 WinAppSDK），
只多了中文；目录里其它文件（ReShade/OptiScaler 的 ini、7z、图标等）全部沿用官方的，一个都不动。

## 两个注意事项

- **RHI 自带自动更新**：官方一更新会把 `RHI.exe` 整个换回英文版，
  届时重新打一次这个补丁即可（或直接改用仓库 Releases 里的完整便携版）。
- **全新安装**请用完整便携版 `RHI-zh-CN-*-win-x64.zip`（约 35 MB，解压即用），
  这个补丁包是给「已经装了官方版」的人用的。

## 许可证

本补丁对应的源码在此仓库，遵循上游 **GPL-3.0**，源码与修改全部开放。
