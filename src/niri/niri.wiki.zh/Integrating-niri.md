此页面包含将niri集成到发行版中的各种有用信息。
首先，要创建niri软件包，请参阅[打包](./Packaging-niri)页面。

### 配置

Niri将从`$XDG_CONFIG_HOME/niri/config.kdl`或`~/.config/niri/config.kdl`加载配置，回退到`/etc/niri/config.kdl`。
如果这些文件都缺失，niri将在`$XDG_CONFIG_HOME/niri/config.kdl`创建[默认配置文件](https://github.com/YaLTeR/niri/blob/main/resources/default-config.kdl)的内容，这些内容在构建时嵌入到niri二进制文件中。

这意味着您可以通过创建`/etc/niri/config.kdl`来自定义您的发行版默认配置。
当此文件存在时，niri*不会*在`~/.config/niri/`自动创建配置，因此您需要指导用户如何自行创建。

请记住，我们在新版本中更新默认配置，因此如果您有自定义的`/etc/niri/config.kdl`，您可能也希望检查并应用相关更改。

目前还不支持将niri配置文件拆分为多个文件或包含。

### Xwayland

运行X11应用程序和游戏需要Xwayland，Orca屏幕阅读器也是如此。

<sup>自：25.08</sup> Niri开箱即用地与[xwayland-satellite](https://github.com/Supreeeme/xwayland-satellite)集成。
集成需要xwayland-satellite >= 0.7在`$PATH`中可用。
请考虑让niri依赖（或至少推荐）xwayland-satellite软件包。
如果您有一个自定义配置，手动启动`xwayland-satellite`并设置`$DISPLAY`，您应删除这些自定义设置以使自动集成生效。

您可以使用[`xwayland-satellite`顶级选项](./Configuration:-Miscellaneous#xwayland-satellite)更改niri查找xwayland-satellite的路径。

### 键盘布局

<sup>自：25.08</sup> 默认情况下（除非[手动配置](./Configuration:-Input#layout)），niri通过D-Bus从systemd-localed的`org.freedesktop.locale1`读取键盘布局设置。
确保您的系统安装程序通过systemd-localed设置键盘布局，niri应该会拾取它。

### 自动启动

Niri与正常的systemd自动启动配合工作。
默认的[niri.service](https://github.com/YaLTeR/niri/blob/main/resources/niri.service)同时启动`graphical-session.target`和`xdg-desktop-autostart.target`。

要使程序在niri启动时运行而不编辑niri配置，您可以将其.desktop文件链接到`~/.config/autostart/`，或使用带有`WantedBy=graphical-session.target`的.service文件。
请参阅[示例systemd设置](./Example-systemd-Setup)页面查看一些示例。

如果这不方便，您也可以在niri配置中添加[`spawn-at-startup`](./Configuration:-Miscellaneous#spawn-at-startup)行。

### 屏幕阅读器

<sup>自：25.08</sup> Niri与[Orca](https://orca.gnome.org)屏幕阅读器配合工作。
请参阅[无障碍](./Accessibility)页面了解详细信息和面向无障碍的发行版建议。

### 桌面组件

您很可能至少需要运行通知守护进程、门户和认证代理。
这在[重要软件](./Important-Software)页面中有详细说明。

除此之外，您可能希望预配置一些桌面外壳组件以使体验不那么简陋。
Niri的默认配置启动[Waybar](https://github.com/Alexays/Waybar)，这是一个很好的起点，但您可能希望考虑更改其默认配置，使其不那么复杂，并添加`niri/workspaces`模块。
您可能还需要一个桌面背景工具（[swaybg](https://github.com/swaywm/swaybg)或[swww](https://github.com/LGFae/swww)），以及一个更好的屏幕锁（与默认的`swaylock`相比），如[hyprlock](https://github.com/hyprwm/hyprlock/)。

或者，一些桌面环境和外壳与niri配合工作，可以在一个包中提供更连贯的体验：

*   [LXQt](https://lxqt-project.org/)官方支持niri，详情请参见[他们的wiki](https://github.com/lxqt/lxqt/wiki/ConfigWaylandSettings#general)了解设置方法。
*   许多[XFCE](https://www.xfce.org/)组件可以在Wayland上工作，包括niri。详情请参见[他们的wiki](https://wiki.xfce.org/releng/wayland_roadmap#component_specific_status)。
*   有基于Quickshell的完整桌面外壳支持niri，例如[DankMaterialShell](https://github.com/AvengeMedia/DankMaterialShell)和[Noctalia](https://github.com/noctalia-dev/noctalia-shell)。
*   您可以使用[cosmic-ext-extra-sessions](https://github.com/Drakulix/cosmic-ext-extra-sessions)在niri上运行[COSMIC](https://system76.com/cosmic/)会话。