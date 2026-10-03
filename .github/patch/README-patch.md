# RHI 简体中文补丁 · 使用说明（覆盖原版安装）

这个压缩包只有 2 个文件，作用是把**已安装的官方 RHI** 直接变成中文版，
不用重新安装、不影响你已配置好的游戏和设置。

## 适用版本

- 官方 RHI **2.8.0 / 2.8.1 Beta 3**（GitHub 安装版 `RHI-Setup.exe`）
- Windows x64

其它版本请先备份再试；若启动报错，按下面的「回滚」把原文件放回去即可。

## 操作步骤

1. **完全退出 RHI**（包括系统托盘图标，托盘里右键 → Quit）。
2. 打开 RHI 的安装目录。默认是：
   `C:\Program Files\RHI`
   找不到就在桌面/开始菜单的 RHI 快捷方式上右键 → **打开文件所在的位置**。
3. **备份**这两个文件（原地重命名即可）：
   - `RHI.dll` → `RHI.dll.bak`
   - `resources.pri` → `resources.pri.bak`
4. 把压缩包里的 `RHI.dll` 和 `resources.pri` 复制进去**覆盖**同名文件
   （需要管理员权限，点「继续」）。
5. 启动 RHI → **Settings → Language → 简体中文**，界面立刻切换。
   偏好会记在 `%LOCALAPPDATA%\RHI\settings.json`，下次启动自动保持。

## 回滚

删掉覆盖进去的两个文件，把 `.bak` 改回原名，就是原来的英文版。

## 为什么只换这两个文件

- `RHI.dll`：主程序集，中文词条表（`LocalizationService`）和语言切换逻辑都在这里。
- `resources.pri`：编译后的 XAML 资源包，设置页新增的「Language」下拉卡片在这里。

其余文件（WinAppSDK 运行时、ReShade/OptiScaler 的 ini、7z 等）与官方完全一致，不需要动。

## 两个注意事项

- **RHI 自带自动更新**：官方一更新，会把 `RHI.dll` 覆盖回英文版，
  届时需要重新打一次这个补丁（或直接用仓库 Releases 里的完整便携版）。
- **不要只拷贝 exe**：这个补丁只替换 2 个文件，前提是目录里已经有官方版的其它依赖。
  全新安装请用完整便携版 `RHI-zh-CN-*-win-x64.zip`。

## 许可证

本补丁对应的源码在此仓库，遵循上游 **GPL-3.0**，源码与修改全部开放。
