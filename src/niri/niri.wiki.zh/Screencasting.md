### 概述

niri提供的主要屏幕录制接口是通过门户和pipewire。
它受到[OBS]、Firefox、Chromium、Electron、Telegram和其他应用程序的支持。
您可以录制显示器和单个窗口。

为了使用它，您需要一个工作的D-Bus会话、pipewire、`xdg-desktop-portal-gnome`和[以会话方式运行niri](./Getting-Started)（即通过`niri-session`或从显示管理器）。
在广泛使用的发行版上，这些都应该"正常工作"。

或者，您可以使用依赖`wlr-screencopy`协议的工具，niri也支持该协议。

niri中有几个为屏幕录制设计的功能。
让我们来看看！

### 屏蔽窗口

您可以从屏幕录制中屏蔽特定窗口，将它们替换为纯黑色矩形。
这对于密码管理器或消息窗口等很有用。

![显示窗口正常可见，但在OBS中被屏蔽的截图。](./img/block-out-from-screencast.png)

这通过`block-out-from`窗口规则控制，例如：

```kdl
// 从屏幕录制中屏蔽密码管理器。
window-rule {
    match app-id=r#"^org\.keepassxc\.KeePassXC$"#
    match app-id=r#"^org\.gnome\.World\.Secrets$"#

    block-out-from "screencast"
}
```

您同样可以使用层规则屏蔽层表面：

```kdl
// 从屏幕录制中屏蔽mako通知。
layer-rule {
    match namespace="^notifications$"

    block-out-from "screencast"
}
```

查看[相应wiki部分](./Configuration:-Window-Rules#block-out-from)获取更多详细信息和示例。

### 动态屏幕录制目标

<sup>自：25.05</sup>

Niri提供了一个特殊的屏幕录制流，您可以动态更改它。
它在屏幕录制窗口对话框中显示为"niri动态录制目标"。

![显示niri动态录制目标的屏幕录制对话框。](https://github.com/user-attachments/assets/e236ce74-98ec-4f3a-a99b-29ac1ff324dd)

当您选择它时，它将以空的透明视频流开始。
然后，您可以使用以下绑定更改它显示的内容：

*   `set-dynamic-cast-window`录制聚焦窗口。
*   `set-dynamic-cast-monitor`录制聚焦显示器。
*   `clear-dynamic-cast-target`返回空流。

您也可以从命令行使用这些操作，例如交互式选择要录制的窗口：

```sh
$ niri msg action set-dynamic-cast-window --id $(niri msg --json pick-window | jq .id)
```

<video controls src="https://github.com/user-attachments/assets/c617a9d6-7d5e-4f1f-b8cc-9301182d9634">

https://github.com/user-attachments/assets/c617a9d6-7d5e-4f1f-b8cc-9301182d9634

</video>

如果录制目标消失（例如目标窗口关闭），流将返回空。

所有动态录制共享相同目标，但新录制开始时空，直到下次更改（以避免意外和错误分享敏感内容）。

### 指示录制窗口

<sup>自：25.02</sup>

[`is-window-cast-target=true`窗口规则](./Configuration:-Window-Rules#is-window-cast-target)匹配由正在进行的窗口屏幕录制目标的窗口。
您使用它与特殊边框颜色一起清晰指示录制窗口。

这也适用于动态屏幕录制目标的窗口。
但是，它不会适用于恰好在全显示器屏幕录制中可见的窗口。

```kdl
// 用红色指示录制窗口。
window-rule {
    match is-window-cast-target=true

    focus-ring {
        active-color "#f38ba8"
        inactive-color "#7d0d2d"
    }

    border {
        inactive-color "#7d0d2d"
    }

    shadow {
        color "#7d0d2d70"
    }

    tab-indicator {
        active-color "#f38ba8"
        inactive-color "#7d0d2d"
    }
}
```

示例：

![用红色边框和阴影指示的录制窗口。](https://github.com/user-attachments/assets/375b381e-3a87-4e94-8676-44404971d893)

### 窗口化（假/分离）全屏

<sup>自：25.05</sup>

当录制基于浏览器的演示文稿如Google幻灯片时，您通常希望隐藏浏览器UI，这需要使浏览器全屏。
这并不总是方便，例如如果您有超宽显示器，或只是想将浏览器作为较小的窗口留下，而不占用整个显示器。

`toggle-windowed-fullscreen`绑定有助于此。
它告诉应用程序它已全屏，而实际上将其保留为您可以调整大小并放在任何地方的普通窗口。

```kdl
binds {
    Mod+Ctrl+Shift+F { toggle-windowed-fullscreen; }
}
```

请记住，并非所有应用程序都对全屏做出反应，因此有时看起来绑定没有做任何事情。

以下是一个窗口化全屏Google幻灯片[演示文稿](https://youtu.be/Kmz8ODolnDg)的示例，以及演示者视图和会议应用程序：

![窗口化的Google幻灯片演示文稿，另一个窗口显示演示者视图，另一个窗口显示Zoom UI录制演示文稿。](https://github.com/user-attachments/assets/b2b49eea-f5a0-4c0a-b537-51fd1949a59d)

### 屏幕镜像

对于演示文稿，将输出镜像到另一个输出可能很有用。
目前，niri没有内置的输出镜像，但您可以使用第三方工具[`wl-mirror`](https://github.com/Ferdi265/wl-mirror)将输出镜像到窗口。
请注意，下面的命令需要安装[`jq`](https://jqlang.org/download/)。

```kdl
binds {
    Mod+P repeat=false { spawn-sh "wl-mirror $(niri msg --json focused-output | jq -r .name)"; }
}
```

聚焦您想要镜像的输出，按<kbd>Mod</kbd><kbd>P</kbd>并将`wl-mirror`窗口移动到目标输出。
最后，将`wl-mirror`窗口全屏（默认情况下，<kbd>Mod</kbd><kbd>Shift</kbd><kbd>F</kbd>）。

[OBS]: https://obsproject.com/