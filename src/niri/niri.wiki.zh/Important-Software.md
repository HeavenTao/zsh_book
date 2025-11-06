由于niri不是一个完整的桌面环境，您很可能需要运行以下软件以确保其他应用程序正常工作。

### 通知守护进程

许多应用程序需要一个。例如，[mako](https://github.com/emersion/mako)效果很好。使用[systemd设置](./Example-systemd-Setup)或[`spawn-at-startup`](./Configuration:-Miscellaneous#spawn-at-startup)。

### 门户

这些为应用程序提供跨桌面API，用于各种用途，如文件选择器或UI设置。Flatpak应用程序特别需要工作门户。

门户**需要**[以会话方式运行niri](./Getting-Started)，这意味着通过`niri-session`脚本或从显示管理器运行。您需要安装以下门户：

*   `xdg-desktop-portal-gtk`：实现大部分基本功能，这是"默认后备门户"。
*   `xdg-desktop-portal-gnome`：屏幕录制支持所需。
*   `gnome-keyring`：实现Secret门户，某些应用程序工作所需。

然后systemd应该按需自动启动它们。这些特定门户在`niri-portals.conf`中配置，该配置[必须安装](./Getting-Started#manual-installation)在正确位置。

由于我们使用`xdg-desktop-portal-gnome`，Flatpak应用程序将读取GNOME UI设置。例如，要启用深色样式，运行：

    dconf write /org/gnome/desktop/interface/color-scheme '"prefer-dark"'

请注意，如果您使用提供的`resources/niri-portals.conf`，您还需要安装`nautilus`文件管理器以使文件选择对话框正常工作。这是必要的，因为从版本47.0开始，xdg-desktop-portal-gnome默认使用nautilus作为文件选择器。

如果您不想安装`nautilus`（比如您使用`nemo`），您可以在`niri-portals.conf`中设置`org.freedesktop.impl.portal.FileChooser=gtk;`以使用GTK门户进行文件选择对话框。

### 认证代理

当应用程序需要请求root权限时需要。像`plasma-polkit-agent`这样的程序效果很好。使用[systemd](./Example-systemd-Setup)或[`spawn-at-startup`](./Configuration:-Miscellaneous#spawn-at-startup)启动它。

请注意，在Fedora上使用systemd启动`plasma-polkit-agent`，您需要覆盖其systemd服务以添加正确的依赖关系。运行：

    systemctl --user edit --full plasma-polkit-agent.service

然后添加`After=graphical-session.target`。

### Xwayland

要运行X11应用程序如Steam或Discord，您可以使用[xwayland-satellite]。
查看[Xwayland wiki页面](./Xwayland)获取说明。

[xwayland-satellite]: https://github.com/Supreeeme/xwayland-satellite