## 使用xwayland-satellite

<sup>自：25.08</sup> Niri开箱即用地与[xwayland-satellite](https://github.com/Supreeeme/xwayland-satellite)集成。
确保安装了xwayland-satellite >= 0.7并在`$PATH`中可用。
无需进一步配置，niri将在磁盘上创建X11套接字，导出`$DISPLAY`，并在X11客户端连接时按需启动xwayland-satellite。
如果xwayland-satellite死亡，niri将自动重启它。

如果您有一个自定义配置，手动启动`xwayland-satellite`并设置`$DISPLAY`，您应删除这些自定义设置以使自动集成生效。

要检查集成是否工作，请验证niri输出是否说类似`listening on X11 socket: :0`：

```sh
$ journalctl --user-unit=niri -b
systemd[2338]: Starting niri.service - A scrollable-tiling Wayland compositor...
niri[2474]: 2025-08-29T04:07:40.043402Z  INFO niri: starting version 25.05.1 (0.0.git.2345.d9833fc1)
(...)
niri[2474]: 2025-08-29T04:07:40.690512Z  INFO niri: listening on Wayland socket: wayland-1
niri[2474]: 2025-08-29T04:07:40.690520Z  INFO niri: IPC listening on: /run/user/1000/niri.wayland-1.2474.sock
niri[2474]: 2025-08-29T04:07:40.700137Z  INFO niri: listening on X11 socket: :0
systemd[2338]: Started niri.service - A scrollable-tiling Wayland compositor.
$ echo $DISPLAY
:0
```

![xwayland-satellite运行Steam和半条命。](https://github.com/user-attachments/assets/57db8f96-40d4-4621-a389-373c169349a4)

我们使用xwayland-satellite而不是直接使用Xwayland，因为[X11非常诡异](./FAQ#why-doesnt-niri-integrate-xwayland-like-other-compositors)。
xwayland-satellite为我们承担了处理X11特殊性的大部分工作，给niri提供正常的Wayland窗口来管理。

xwayland-satellite与大多数应用程序配合良好：Steam、游戏、Discord，甚至更奇特的东西如带有wine Windows VST插件的Ardour。
然而，想要在特定屏幕坐标定位窗口或栏的X11应用程序不会正确行为，需要运行嵌套合成器。
请参见下面的章节了解如何做到这一点。

## 使用labwc Wayland合成器

[Labwc](https://github.com/labwc/labwc)是一个带有Xwayland的传统堆叠式Wayland合成器。
您可以在窗口中运行它，然后在内部运行X11应用程序。

1.  从您的发行版包中安装labwc。
2.  使用`labwc`命令在niri内运行它。
    它将作为新窗口打开。
3.  在它提供的X11 DISPLAY上运行X11应用程序，例如`env DISPLAY=:0 glxgears`

![Labwc运行X11应用程序。](https://github.com/user-attachments/assets/aecbcecb-f0cb-4909-867f-09d34b5a2d7e)

## 直接以rootful模式运行Xwayland

此方法涉及直接调用XWayland并将其作为自己的窗口运行，它还需要在其中运行额外的X11窗口管理器。

![Xwayland在rootful模式下运行。](https://github.com/YaLTeR/niri/assets/1794388/b64e96c4-a0bb-4316-94a0-ff445d4c7da7)

以下是您如何操作：

1.  运行`Xwayland`（只是二进制文件本身，不带标志）。
    这将生成一个黑色窗口，您可以调整大小和全屏（使用Mod+Shift+F）以方便。
    在较旧的Xwayland版本中，窗口将是屏幕大小且不可调整大小。
2.  在其中运行一些X11窗口管理器，例如`env DISPLAY=:0 i3`。
    这样您可以在Xwayland实例内管理X11窗口。
3.  在其中运行X11应用程序，例如`env DISPLAY=:0 flatpak run com.valvesoftware.Steam`。

在全屏Xwayland内全屏游戏，您将获得几乎正常的游戏体验。

> \[!TIP]
> 如果您不运行X11窗口管理器，Xwayland将在所有X11窗口关闭且新窗口打开时关闭并重新打开其窗口。
> 为防止此情况，请按上述方式启动X11 WM，或打开其他长期运行的X11窗口。

一个警告是，当前rootful Xwayland似乎不与合成器共享剪贴板。
对于文本数据，您可以使用[wl-clipboard](https://github.com/bugaevc/wl-clipboard)手动操作，例如：

*   `env DISPLAY=:0 xsel -ob | wl-copy` 从Xwayland复制到niri剪贴板
*   `wl-paste -n | env DISPLAY=:0 xsel -ib` 从niri复制到Xwayland剪贴板

如果需要，您也可以将这些绑定到热键：

    binds {
        Mod+Shift+C { spawn "sh" "-c" "env DISPLAY=:0 xsel -ob | wl-copy"; }
        Mod+Shift+V { spawn "sh" "-c" "wl-paste -n | env DISPLAY=:0 xsel -ib"; }
    }

## 使用xwayland-run运行Xwayland

[xwayland-run]是一个在专用Xwayland rootful服务器中运行X11客户端的辅助工具。
它负责启动Xwayland，设置X11 DISPLAY环境变量，设置xauth并使用新启动的Xwayland实例运行指定的X11客户端。
当X11客户端终止时，xwayland-run将自动关闭专用的Xwayland服务器。

它的工作方式如下：

    xwayland-run <Xwayland参数> -- your-x11-app <X11应用程序参数>

例如：

    xwayland-run -geometry 800x600 -fullscreen -- wine wingame.exe

## 使用Cage Wayland合成器

也可以在[Cage](https://github.com/cage-kiosk/cage)中运行X11应用程序，它运行一个支持Xwayland的嵌套Wayland会话，X11应用程序可以在其中运行。

与Xwayland rootful方法相比，这不需要运行额外的X11窗口管理器，可以用一个命令`cage -- /path/to/application`使用。然而，如果在Cage中启动多个窗口，也可能导致问题，因为Cage旨在用于信息亭，每个新窗口将自动全屏并接管之前打开的窗口。

要使用Cage，您需要：

1.  安装`cage`，它应该在大多数仓库中。
2.  运行`cage -- /path/to/application`并在niri上享受您的X11程序。

可选地，人们也可以修改应用程序的桌面条目并添加`cage --`前缀到`Exec`属性。例如Spotify Flatpak将看起来像这样：

```ini
[Desktop Entry]
Type=Application
Name=Spotify
GenericName=Online music streaming service
Comment=Access all of your favorite music
Icon=com.spotify.Client
Exec=cage -- flatpak run com.spotify.Client
Terminal=false
```

## Proton-GE原生Wayland

可以将一些游戏作为原生Wayland客户端运行，绕过与X11相关的问题。您可以使用Proton的自定义版本，如[Proton-GE](https://github.com/GloriousEggroll/proton-ge-custom)，通过在游戏启动参数中设置`PROTON_ENABLE_WAYLAND=1`环境变量来实现。请注意，目前这是一个实验性功能，可能不适用于每个游戏，并且可能有其自身的问题。

    PROTON_ENABLE_WAYLAND=1 %command%

## 使用gamescope

您可以使用[gamescope](https://github.com/ValveSoftware/gamescope)运行X11游戏甚至Steam本身。

与Cage类似，gamescope只显示单个、最顶层的窗口，所以不太适合运行常规应用程序。
但您可以在gamescope中运行Steam，然后从Steam启动一些游戏。

    gamescope -- flatpak run com.valvesoftware.Steam

要全屏运行gamescope，您可以传递设置必要分辨率的标志，以及以全屏模式启动的标志：

    gamescope -W 2560 -H 1440 -w 2560 -h 1440 -f  -- flatpak run com.valvesoftware.Steam

> \[!NOTE]
> 如果在gamescope中运行时Steam异常终止，后续的gamescope调用有时会无法正确启动它。
> 如果发生这种情况，请按照上面的说明在rootful Xwayland中运行Steam，然后正常退出，然后您将能够再次使用gamescope。

[xwayland-run]: https://gitlab.freedesktop.org/ofourdan/xwayland-run

[xwayland-satellite]: https://github.com/Supreeeme/xwayland-satellite