获取niri的最简单方法是安装其中一个发行版软件包。
以下是一些软件包：[Fedora COPR](https://copr.fedorainfracloud.org/coprs/yalter/niri/) 和 [nightly COPR](https://copr.fedorainfracloud.org/coprs/yalter/niri-git/)（由我自己维护），[NixOS Flake](https://github.com/sodiboo/niri-flake)，以及下面repology中的更多软件包，包括用于基于Debian的发行版的[pacstall软件包](https://pacstall.dev/packages/niri/)。
如果您想自己编译niri，请参阅[构建](#构建)部分；如果您想打包niri，请参阅[打包niri](./Packaging-niri)页面。

[![打包状态](https://repology.org/badge/vertical-allrepos/niri.svg)](https://repology.org/project/niri/versions)

安装后，从显示管理器（如GDM）启动niri。
按<kbd>Super</kbd><kbd>T</kbd>运行终端（[Alacritty]）和<kbd>Super</kbd><kbd>D</kbd>运行应用程序启动器（[fuzzel]）。
要退出niri，按<kbd>Super</kbd><kbd>Shift</kbd><kbd>E</kbd>。

如果您不使用显示管理器，应该从TTY运行`niri-session`（systemd/dinit）或`niri --session`（其他）。
`--session`标志将使niri将其环境变量全局导入到系统管理器和D-Bus中，并启动其D-Bus服务。
`niri-session`脚本还将额外将niri作为systemd/dinit服务启动，这将启动图形会话目标，一些服务（如门户）需要该目标。

您也可以在现有的桌面会话中运行`niri`。
然后它将以窗口形式打开，您可以在这里试用它。
请注意，这种窗口模式主要用于开发，因此有些bug（特别是热键方面的问题）。

接下来，请参阅[重要软件列表](./Important-Software)，了解正常桌面使用所需的软件，如通知守护进程和门户。
此外，请查看[配置介绍](./Configuration:-Introduction)页面开始配置niri。
在那里您可以找到链接到其他页面，其中包含所有选项的详细文档和示例。
最后，[Xwayland](./Xwayland)页面解释了如何在niri上运行X11应用程序。

### 桌面环境

一些桌面环境和shell可以与niri配合使用，提供更开箱即用的体验：

*   [LXQt](https://lxqt-project.org/)官方支持niri，详情请参见[他们的wiki](https://github.com/lxqt/lxqt/wiki/ConfigWaylandSettings#general)了解设置方法。
*   许多[XFCE](https://www.xfce.org/)组件可以在Wayland上工作，包括niri。详情请参见[他们的wiki](https://wiki.xfce.org/releng/wayland_roadmap#component_specific_status)。
*   有基于Quickshell的完整桌面shell支持niri，例如[DankMaterialShell](https://github.com/AvengeMedia/DankMaterialShell)和[Noctalia](https://github.com/noctalia-dev/noctalia-shell)。
*   您可以使用[cosmic-ext-extra-sessions](https://github.com/Drakulix/cosmic-ext-extra-sessions)在niri上运行[COSMIC](https://system76.com/cosmic/)会话。

### NVIDIA

NVIDIA驱动程序目前存在由于堆重用特性导致的高VRAM使用问题。
如果您在NVIDIA GPU上运行niri，建议应用[此处](./Nvidia)记录的手动修复。

NVIDIA GPU在运行niri时可能会遇到问题（例如，从TTY启动时屏幕保持黑色）。
有时，这些问题可以解决。
您可以尝试以下方法：

1.  更新NVIDIA驱动程序。您需要足够新的GPU和驱动程序来支持GBM。
2.  确保启用了内核模式设置。这通常涉及在内核命令行中添加`nvidia-drm.modeset=1`。查找并按照您发行版的指南操作。其他Wayland合成器的指南可能会有所帮助。

### Asahi、ARM和其他kmsro设备

在一些这样的系统上，niri无法正确检测主渲染设备。
如果您在TTY上启动niri时遇到黑屏，可以尝试手动设置设备。

首先，查找您拥有的设备：

    $ ls -l /dev/dri/
    drwxr-xr-x@       - root 14 мая 07:07 by-path
    crw-rw----@   226,0 root 14 мая 07:07 card0
    crw-rw----@   226,1 root 14 мая 07:07 card1
    crw-rw-rw-@ 226,128 root 14 мая 07:07 renderD128
    crw-rw-rw-@ 226,129 root 14 мая 07:07 renderD129

您可能有一个`render`设备和两个`card`设备。

打开位于`~/.config/niri/config.kdl`的niri配置文件，并按如下方式放置您的`render`设备路径：

```kdl
debug {
    render-drm-device "/dev/dri/renderD128"
}
```

保存，然后再次尝试启动niri。
如果您仍然遇到黑屏，请尝试使用每个`card`设备。

### Nix/NixOS

mesa驱动程序与niri不同步是一个常见问题，因此请确保您的系统mesa版本与niri的mesa版本匹配。
当这种情况发生时，您通常会在尝试从TTY启动niri时看到黑屏。

此外，在Intel显卡上，您可能需要[此处](https://wiki.nixos.org/wiki/Intel_Graphics)描述的解决方法。

### 虚拟机

要在虚拟机中运行niri，请确保启用3D加速。

## 主要默认热键

在TTY上运行时，Mod键是<kbd>Super</kbd>。
在窗口中运行时，Mod键是<kbd>Alt</kbd>。

一般系统是：如果热键切换到某处，那么添加<kbd>Ctrl</kbd>会将焦点窗口或列移动到那里。

| 热键 | 描述 |
| ------ | ----------- |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>/</kbd> | 显示重要的niri热键列表 |
| <kbd>Mod</kbd><kbd>T</kbd> | 启动`alacritty`（终端） |
| <kbd>Mod</kbd><kbd>D</kbd> | 启动`fuzzel`（应用程序启动器） |
| <kbd>Super</kbd><kbd>Alt</kbd><kbd>L</kbd> | 启动`swaylock`（屏幕锁） |
| <kbd>Mod</kbd><kbd>Q</kbd> | 关闭焦点窗口 |
| <kbd>Mod</kbd><kbd>H</kbd>或<kbd>Mod</kbd><kbd>←</kbd> | 聚焦左侧列 |
| <kbd>Mod</kbd><kbd>L</kbd>或<kbd>Mod</kbd><kbd>→</kbd> | 聚焦右侧列 |
| <kbd>Mod</kbd><kbd>J</kbd>或<kbd>Mod</kbd><kbd>↓</kbd> | 聚焦列中的下方窗口 |
| <kbd>Mod</kbd><kbd>K</kbd>或<kbd>Mod</kbd><kbd>↑</kbd> | 聚焦列中的上方窗口 |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>H</kbd>或<kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>←</kbd> | 将焦点列向左移动 |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>L</kbd>或<kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>→</kbd> | 将焦点列向右移动 |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>J</kbd>或<kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>↓</kbd> | 将焦点窗口在列中向下移动 |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>K</kbd>或<kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>↑</kbd> | 将焦点窗口在列中向上移动 |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>H</kbd><kbd>J</kbd><kbd>K</kbd><kbd>L</kbd>或<kbd>Mod</kbd><kbd>Shift</kbd><kbd>←</kbd><kbd>↓</kbd><kbd>↑</kbd><kbd>→</kbd> | 聚焦侧边显示器 |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>H</kbd><kbd>J</kbd><kbd>K</kbd><kbd>L</kbd>或<kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>←</kbd><kbd>↓</kbd><kbd>↑</kbd><kbd>→</kbd> | 将焦点列移动到侧边显示器 |
| <kbd>Mod</kbd><kbd>U</kbd>或<kbd>Mod</kbd><kbd>PageDown</kbd> | 切换到下方工作区 |
| <kbd>Mod</kbd><kbd>I</kbd>或<kbd>Mod</kbd><kbd>PageUp</kbd> | 切换到上方工作区 |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>U</kbd>或<kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>PageDown</kbd> | 将焦点列移动到下方工作区 |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>I</kbd>或<kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>PageUp</kbd> | 将焦点列移动到上方工作区 |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>U</kbd>或<kbd>Mod</kbd><kbd>Shift</kbd><kbd>PageDown</kbd> | 将焦点工作区向下移动 |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>I</kbd>或<kbd>Mod</kbd><kbd>Shift</kbd><kbd>PageUp</kbd> | 将焦点工作区向上移动 |
| <kbd>Mod</kbd><kbd>,</kbd> | 将右侧窗口合并到焦点列中 |
| <kbd>Mod</kbd><kbd>.</kbd> | 将焦点列中的底部窗口分离到自己的列中 |
| <kbd>Mod</kbd><kbd>\[</kbd> | 将焦点窗口向左合并或分离 |
| <kbd>Mod</kbd><kbd>]</kbd> | 将焦点窗口向右合并或分离 |
| <kbd>Mod</kbd><kbd>R</kbd> | 在预设列宽度之间切换 |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>R</kbd> | 在预设列高度之间切换 |
| <kbd>Mod</kbd><kbd>F</kbd> | 最大化列 |
| <kbd>Mod</kbd><kbd>C</kbd> | 将列居中显示 |
| <kbd>Mod</kbd><kbd>-</kbd> | 将列宽度减少10% |
| <kbd>Mod</kbd><kbd>=</kbd> | 将列宽度增加10% |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>-</kbd> | 将窗口高度减少10% |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>=</kbd> | 将窗口高度增加10% |
| <kbd>Mod</kbd><kbd>Ctrl</kbd><kbd>R</kbd> | 将窗口高度重置为自动 |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>F</kbd> | 在焦点窗口上切换全屏 |
| <kbd>Mod</kbd><kbd>V</kbd> | 在浮动布局和堆叠布局之间移动焦点窗口 |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>V</kbd> | 在浮动布局和堆叠布局之间切换焦点 |
| <kbd>PrtSc</kbd> | 截取区域截图。用鼠标选择要截图的区域，然后按空格键保存截图，或按Esc取消 |
| <kbd>Alt</kbd><kbd>PrtSc</kbd> | 将焦点窗口的截图复制到剪贴板并保存到`~/Pictures/Screenshots/` |
| <kbd>Ctrl</kbd><kbd>PrtSc</kbd> | 将焦点显示器的截图复制到剪贴板并保存到`~/Pictures/Screenshots/` |
| <kbd>Mod</kbd><kbd>Shift</kbd><kbd>E</kbd>或<kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>Delete</kbd> | 退出niri |

## 构建

首先，为您的发行版安装依赖项。

*   Ubuntu 24.04:

    ```sh
    sudo apt-get install -y gcc clang libudev-dev libgbm-dev libxkbcommon-dev libegl1-mesa-dev libwayland-dev libinput-dev libdbus-1-dev libsystemd-dev libseat-dev libpipewire-0.3-dev libpango1.0-dev libdisplay-info-dev
    ```

*   Fedora:

    ```sh
    sudo dnf install gcc libudev-devel libgbm-devel libxkbcommon-devel wayland-devel libinput-devel dbus-devel systemd-devel libseat-devel pipewire-devel pango-devel cairo-gobject-devel clang libdisplay-info-devel
    ```

接下来，获取最新稳定版Rust：https://rustup.rs/

然后，使用`cargo build --release`构建niri。

检查Cargo.toml以获取构建特性的列表。
例如，您可以使用`cargo build --release --no-default-features --features dinit,dbus,xdp-gnome-screencast`将systemd集成替换为dinit集成。

> \[!WARNING]
> 不要使用`--all-features`构建！
>
> 一些特性仅用于开发用途。
> 例如，其中一个特性启用了将分析数据收集到内存缓冲区中，该缓冲区将无限增长直到您耗尽内存。

### NixOS/Nix

我们有一个社区维护的flake，提供了包含所需依赖项的开发shell。使用`nix build`构建niri，然后运行`./results/bin/niri`。

如果您不在NixOS上，您可能需要[NixGL](https://github.com/nix-community/nixGL)来运行生成的二进制文件：

```sh
nix run --impure github:guibou/nixGL -- ./results/bin/niri
```

### 手动安装

如果直接安装而没有软件包，推荐的文件目标位置略有不同。
在这种情况下，请将文件放在下表所示的目录中。
这些可能因您的发行版而异。

不要忘记确保niri.service中的`niri`路径正确。
默认为`/usr/bin/niri`。

| 文件 | 目标位置 |
| ---- | -------- |
| `target/release/niri` | `/usr/local/bin/` |
| `resources/niri-session` | `/usr/local/bin/` |
| `resources/niri.desktop`  | `/usr/local/share/wayland-sessions/` |
| `resources/niri-portals.conf` | `/usr/local/share/xdg-desktop-portal/` |
| `resources/niri.service` (systemd) | `/etc/systemd/user/` |
| `resources/niri-shutdown.target` (systemd) | `/etc/systemd/user/` |
| `resources/dinit/niri` (dinit) | `/etc/dinit.d/user/` |
| `resources/dinit/niri-shutdown` (dinit) | `/etc/dinit.d/user/` |

[Alacritty]: https://github.com/alacritty/alacritty

[fuzzel]: https://codeberg.org/dnkl/fuzzel