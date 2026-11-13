# 桌面图标管家（DesktopIconsTray）v1.4

一个常驻托盘、体积很小的 Windows 小工具：**启动后自动隐藏桌面图标**，右键托盘图标即可 **显示** / **隐藏**桌面图标，或 **退出**。无窗口、无控制台、无第三方依赖、不联网、不需要管理员权限。

![托盘右键菜单](test/tray-menu.png)

> v1.4 新增：**启动时自动隐藏桌面图标**（默认行为）；需要启动后保持显示时，给 exe 加 `--show` 或 `--no-autohide` 参数。
>
> v1.3 兼容性优化：多级桌面窗口查找、.NET 3.5 兼容版、应用清单、32/64 位 AnyCPU。  
> v1.2：退出程序时自动恢复显示桌面图标。  
> v1.1：优先调用资源管理器自己的显示/隐藏命令，避免桌面框选虚线异常。

## 应该使用哪个 exe？

| Windows 版本 | 推荐程序 | 说明 |
| --- | --- | --- |
| Windows 11（21H2–25H2） | `DesktopIconsTray.exe` | 系统自带 .NET Framework 4.8 |
| Windows 10（1607 及以上） | `DesktopIconsTray.exe` | 系统自带 .NET Framework 4.6+ |
| Windows 8.1 / Windows 8 | `DesktopIconsTray.exe` | 系统自带 .NET Framework 4.5 |
| Windows 7 SP1 | 已装 .NET 4.0+ 用 `DesktopIconsTray.exe`；否则用 `DesktopIconsTray_NET35.exe` | Win7 默认带 .NET 3.5 |
| 32 位 Windows | 同上 | 两个程序都是 AnyCPU，会自动以 32 位运行 |
| Windows Vista / XP | `DesktopIconsTray_NET35.exe`（未实机验证，不保证） | 需要 .NET 3.5 |

`DesktopIconsTray.exe` 需要 **.NET Framework 4.0 或更高**；`DesktopIconsTray_NET35.exe` 只需要 **.NET Framework 3.5**。Windows 8/10/11 一般用主程序即可，不要优先选 NET35 版本（部分 Win10/11 默认未启用 .NET 3.5）。

## 使用方法

1. 双击对应版本的 exe：程序会**立即隐藏桌面图标**，并进入后台，只在通知区域留下一个图标。
2. 如果托盘里看不到图标：点任务栏右下角的 **^** 溢出区，找到“桌面图标管家”，可拖到任务栏固定。
3. 右键托盘图标：
   - **显示桌面图标** —— 恢复显示桌面图标
   - **隐藏桌面图标** —— 隐藏所有桌面图标（壁纸和桌面右键菜单仍正常）
   - **退出** —— 关闭程序，并**自动恢复显示桌面图标**后再退出
4. 菜单打开时会用勾号标出当前状态。
5. 如果启动时不想自动隐藏，请从快捷方式/命令行加 `--show`（或 `--no-autohide`）参数。
6. 再次双击 exe 不会启动第二个实例，只会提示“程序已在后台运行”。

## 兼容性优化点

- 桌面窗口查找改为多级回退：
  1. `GetShellWindow()` 所在窗口整棵子树
  2. `Progman`
  3. 顶层 `WorkerW` / `Progman`（支持嵌套 WorkerW、壁纸幻灯片、多显示器布局）
  4. 其它顶层窗口兜底
- 显示/隐藏命令依次尝试：
  1. 发给 `SHELLDLL_DefView` 的 `WM_COMMAND 0x7402`（Windows 10/11 及部分 Win7/8）
  2. 发给 `Progman` 的同一命令（经典 Win7 方式）
  3. 直接 `ShowWindow` 桌面图标列表（极少数系统命令不响应时兜底）
- 嵌入应用清单（manifest）：
  - `asInvoker`：不需要管理员权限
  - 声明 Windows 7 / 8 / 8.1 / 10 / 11 兼容，避免系统兼容性虚拟化
  - `dpiAware`：高分屏下托盘菜单更清晰
- AnyCPU：同一文件即可在 32 位和 64 位 Windows 上运行
- 提供两份构建：.NET 4.x 主程序 + .NET 3.5 旧系统兼容版，覆盖更多 Windows 版本

## 可选：开机自动启动

1. `Win + R` 输入 `shell:startup` 回车，打开“启动”文件夹。
2. 把对应 exe 的快捷方式放进去。
3. 默认启动就会自动隐藏桌面图标，无需额外参数。如果希望开机启动时保留图标显示，在快捷方式“目标”末尾加参数 `--show`，例如：
   `"D:\工具\DesktopIconsTray.exe" --show`
4. 程序启动后仍会驻留托盘，可随时右键显示/隐藏或退出。

## 命令行参数

| 参数 | 作用 |
| --- | --- |
| `--show` / `--no-autohide` | 启动时保持显示（覆盖默认的启动自动隐藏）；对已运行实例则是显示命令 |
| `--hide` / `--autohide` | 隐藏桌面图标 |
| `--toggle` | 切换显示/隐藏 |
| `--exit` | 让后台程序退出 |
| `--notify` | 弹出“已在运行”的气泡提示 |

在已经运行的情况下，再启动 exe 并带参数，会把命令转给已驻留的实例，然后自己退出。

## 常见问题

**托盘里找不到图标？**  
点任务栏右下角的 `^` 打开溢出区，把“桌面图标管家”拖到任务栏固定即可。

**应该用主程序还是 NET35 版本？**  
Win8/10/11 优先用 `DesktopIconsTray.exe`。Win7 默认只有 .NET 3.5，装过 .NET 4.x 可用主程序，否则用 `DesktopIconsTray_NET35.exe`。

**提示需要安装 .NET Framework？**  
主程序需要 .NET Framework 4.0+，旧系统或未安装时请改用 `DesktopIconsTray_NET35.exe`（需要 .NET 3.5），或安装 .NET Framework 4.8。

**右键菜单点了没反应，或提示“操作失败”？**  
请确认 `explorer.exe` 正常运行；锁屏界面、安全桌面、部分全屏独占程序下无法操作桌面窗口。若同时开着其它桌面图标工具，请先退出那个工具再试。

**隐藏图标后，桌面左键拖动的虚线框也不见了？**  
这是正常现象：虚线框属于桌面图标层，隐藏图标时整个图标层也被隐藏了。点“显示桌面图标”后虚线框会恢复。

**拖动时只看到图标上有选中方块，没有虚线框？**  
Windows 的虚线框只在**按住左键拖动时**显示，松开后只保留图标的选中状态。请从桌面**空白处**按下左键再拖动；如果从图标上按下拖动，会变成拖动文件而不是框选。

**为什么一启动桌面图标就自动隐藏了？**  
v1.4 起这是默认行为。右键托盘图标选“显示桌面图标”即可恢复；如果启动时不想隐藏，用 `--show`（或 `--no-autohide`）参数启动，例如给快捷方式目标加 `--show`。

**退出程序后原来隐藏的图标又出来了？**  
这是 v1.2 起的预期行为：退出时会自动恢复显示桌面图标。如果希望保持隐藏，请不要退出程序。

**提示权限不足 / 操作无效？**  
本工具不需要管理员权限。如果 exe 被放在受限沙箱目录、网盘同步目录或 AppContainer 环境中，进程可能被降权，请把 exe 复制到桌面或 `D:\工具` 这类普通文件夹后再运行。

**Windows 提示“已保护你的电脑” / SmartScreen？**  
这是未签名的小工具，可选“更多信息 → 仍要运行”。程序不联网、不读写敏感数据。

**怎么卸载？**  
右键托盘图标 → 退出；然后删除 exe（以及启动文件夹里的快捷方式）即可。

## 文件说明

| 文件 | 作用 |
| --- | --- |
| `DesktopIconsTray.exe` | 主程序（.NET Framework 4.0+，推荐） |
| `DesktopIconsTray_NET35.exe` | 旧系统兼容版（.NET Framework 3.5） |
| `app.manifest` | 应用兼容性清单 |
| `app.ico` | 程序/托盘图标 |
| `src\*.cs` | C# 源码（WinForms 托盘 + Win32 API） |
| `build.ps1` | 一键编译两个版本 |
| `test\verify.ps1` | 核心功能自动验证（38 项） |
| `test\menu-verify.ps1` | 托盘右键菜单端到端验证 |
| `test\compat-verify.ps1` | 指定 exe 的兼容性冒烟测试（10 项） |
| `test\tray-menu.png` | 菜单截图 |

重新编译（需要 .NET Framework 4.x；如同时存在 3.5 编译器会一并生成兼容版）：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\build.ps1
```

## 实现原理（简述）

- 每次操作动态定位桌面窗口：`Progman` / `WorkerW` → `SHELLDLL_DefView` → `SysListView32`
- 优先向 `SHELLDLL_DefView` 发送资源管理器自己的“显示桌面图标”命令 `WM_COMMAND 0x7402`，保持 Explorer 内部状态一致（不影响桌面框选虚线）
- 如果 `DefView` 不响应，再尝试经典的 `Progman` 命令，最后才直接 `ShowWindow` 兜底
- 操作后轮询 `IsWindowVisible` 确认状态，并同步 `HideIcons` 注册表项
- 启动时默认执行一次隐藏；`--show` / `--no-autohide` 可覆盖为保持显示
- 退出清理阶段先恢复显示桌面图标，再释放托盘并退出
- 托盘用 WinForms `NotifyIcon` + `ContextMenuStrip`，主体为隐藏的 `ApplicationContext`；本机命名管道用于单实例转发和自动化测试

## 验证记录

Windows 版本兼容性只做了**本机 Windows 11 Build 26200（25H2）**实机验证；Windows 7/8/8.1/10 的路径基于公开稳定的 Win32 API 设计，未逐版本实机测试，使用前请先在目标机器上试运行。

在 Windows 11 Build 26200 上：

- 主程序构建成功：.NET Framework 4.x csc，无警告、无第三方依赖
- `test\verify.ps1`：**38 项全部通过**
  - **启动时自动隐藏桌面图标，`HideIcons=1`**
  - 启动驻留、托盘图标控制通道状态正确
  - 隐藏后真实窗口不可见、`HideIcons=1`、状态稳定
  - 显示后真实窗口可见、`HideIcons=0`
  - 隐藏图标后退出程序，桌面图标自动恢复显示、`HideIcons=0`
  - `--show` 启动可覆盖自动隐藏，保持图标显示
  - toggle、单实例、退出、`--hide` / `--show` / `--exit` 转发均通过
- `test\menu-verify.ps1`：托盘右键菜单实际弹出，隐藏 / 显示 / 退出全部通过
  - 启动后确认图标自动隐藏，通过菜单“显示桌面图标”恢复
  - 通过 UI Automation 在系统托盘溢出区找到图标并右键
  - 通过鼠标点击菜单项，验证桌面图标真实状态和进程退出
  - 退出前先隐藏图标，点击“退出”后确认图标自动恢复显示
- 框选虚线验证：用本工具“隐藏 → 显示”后，模拟桌面左键拖动并截屏确认虚线框正常出现
- .NET 3.5 兼容版：在本机（已启用 .NET 3.5）完成启动自动隐藏 / 显示 / 隐藏 / 退出恢复验证，10 项通过
- 32 位进程：用 x86 构建在本机 64 位 Windows 上完成同样的 10 项验证，确认 32 位系统路径可用
- 两个发布 exe 均确认：运行时版本分别为 `v4.0.30319` / `v2.0.50727`，均嵌入了含 `supportedOS` / `dpiAware` / `asInvoker` 的 manifest

> 说明：验证时通过独立 Win32 窗口查询确认 `SysListView32` 的真实可见性，而不是只相信程序自己返回的结果。
