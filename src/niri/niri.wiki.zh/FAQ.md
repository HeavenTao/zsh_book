### 如何禁用客户端装饰/使窗口矩形化？

取消注释配置顶层的[`prefer-no-csd`设置](./Configuration:-Miscellaneous#prefer-no-csd)，然后重新启动您的应用程序。
然后niri将要求窗口省略客户端装饰，并通知它们正在被平铺（这使得一些窗口矩形化，即使它们无法省略装饰）。

请注意，目前这将阻止边缘窗口调整大小手柄显示。
您仍然可以通过按住<kbd>Mod</kbd>和鼠标右键来调整窗口大小。

### 为什么透明窗口被着色？/为什么边框/焦点环透过半透明窗口显示？

取消注释配置顶层的[`prefer-no-csd`设置](./Configuration:-Miscellaneous#prefer-no-csd)，然后重新启动您的应用程序。
Niri将在同意省略其客户端装饰的窗口*周围*绘制焦点环和边框。

默认情况下，焦点环和边框渲染为窗口后面的纯色背景矩形。
也就是说，它们将透过半透明窗口显示。
这是因为使用客户端装饰的窗口可以具有任意形状。

您还可以使用[`draw-border-with-background`窗口规则](./Configuration:-Window-Rules#draw-border-with-background)覆盖此行为。

### 如何为所有窗口启用圆角？

在配置中放入此窗口规则：

```kdl
window-rule {
    geometry-corner-radius 12
    clip-to-geometry true
}
```

有关更多信息，请查看[`geometry-corner-radius`窗口规则](./Configuration:-Window-Rules#geometry-corner-radius)。

### 如何隐藏启动时的"重要热键"弹出窗口？

在配置中放入以下内容：

```kdl
hotkey-overlay {
    skip-at-startup
}
```

### 如何运行X11应用程序如Steam或Discord？

要运行X11应用程序，您可以使用[xwayland-satellite](https://github.com/Supreeeme/xwayland-satellite)。
查看[Xwayland wiki页面](./Xwayland)获取说明。

请记住，您可以通过传递正确的标志在Wayland上原生运行许多Electron应用程序，例如`code --ozone-platform-hint=auto`

### 为什么niri不像其他合成器那样集成Xwayland？

多种因素的组合：

*   集成Xwayland需要相当多的工作，因为合成器需要实现X11窗口管理器的部分功能。
*   您需要满足X11的窗口概念，而对niri来说，我希望拥有最好的Wayland代码。
*   niri没有X11所需的良好全局坐标系统。
*   您往往会遇到无休止的X11错误流，这些错误会占用更多时间和精力去做其他任务。
*   现如今实际上没有那么多仅X11的客户端，而xwayland-satellite很好地处理了其中的大部分。
*   niri不是一个必须支持所有用例的大型严肃桌面环境（并且由某家公司支持）。

总的来说，这种情况有利于避免Xwayland集成。

<sup>自：25.08</sup> niri具有无缝的内置xwayland-satellite集成，基本上与其他合成器中的内置Xwayland一样工作，解决了手动设置的障碍。

如果将来xwayland-satellite成为新合成器集成Xwayland的标准方式，我不会太惊讶，因为它承担了大部分繁琐的工作，并将合成器与行为不当的客户端隔离。

### 我能启用半透明窗口后面的模糊效果吗？

还没有，关注/点赞[这个问题](https://github.com/YaLTeR/niri/issues/54)。

还有一个[PR](https://github.com/YaLTeR/niri/pull/1634)向niri添加模糊效果，您可以手动构建和运行。
请记住，这是一个实验性实现，可能存在问题和性能问题。

### 我能将窗口设置为粘性/固定/总在顶部/在所有工作区显示吗？

还没有，关注/点赞[这个问题](https://github.com/YaLTeR/niri/issues/932)。

您可以使用脚本来模拟此行为，该脚本使用niri IPC。
例如，[nirius](https://git.sr.ht/~tsdh/nirius)似乎有此功能（`toggle-follow-mode`）。

### 如何使Firefox中的Bitwarden窗口以浮动方式打开？

Firefox似乎首先以通用的Firefox标题打开Bitwarden窗口，然后才将窗口标题更改为Bitwarden，因此您无法有效地使用`open-floating`窗口规则定位它。

您需要使用脚本，例如[这个](https://github.com/YaLTeR/niri/discussions/1599)或其他脚本（在niri问题和讨论中搜索Bitwarden）。

### 我能直接在当前列中打开窗口/在另一个窗口的同一列中打开窗口吗？

不能，但您可以使用[niri IPC](./IPC)脚本实现您想要的行为。
监听新窗口打开的事件流，然后调用像`consume-or-expel-window-left`这样的操作。

直接在niri中添加此功能具有挑战性：

*   "直接在某列中打开窗口"这一行为本身就相当复杂。Niri需要计算给定列中其他窗口如何响应调整大小时新窗口的确切初始大小。这种逻辑存在，但不能直接插入到计算新窗口大小的代码中。然后，它需要处理各种边缘情况，如列消失，或在目标窗口有机会出现之前向列中添加新窗口。
*   您如何指示新窗口应该在现有列中生成（以及在哪个列中），而不是在新列中？不同的人在这里似乎有不同的需求（包括基于父PID等的非常复杂的规则），在设计上非常不清楚实际需要什么样的（简单）设置以及什么设置会有用。另请参见https://github.com/YaLTeR/niri/discussions/1125。

### 为什么将鼠标移动到显示器边缘会聚焦下一个窗口，但有时不会？

这可能发生在[`focus-follows-mouse`](./Configuration:-Input#focus-follows-mouse)中。
使用客户端装饰时，窗口应该在几何体外有一些边距用于鼠标调整大小手柄。
这些边距"从显示器边缘探出"，因为它们在窗口几何体外，而`focus-follows-mouse`在鼠标穿过它们时触发。

它并不总是发生：

*   一些工具包不会在窗口几何体外放置调整大小手柄。然后没有输入区域，所以`focus-follows-mouse`无处触发。
*   如果当前窗口有自己的调整大小边距，并且它一直延伸到显示器边缘，那么`focus-follows-mouse`不会触发，因为鼠标永远不会离开当前窗口。

要解决此问题，您可以：

*   使用`focus-follows-mouse max-scroll-amount="0%"`，这将防止`focus-follows-mouse`在会导致滚动时触发。
*   设置`prefer-no-csd`，这通常会导致客户端移除那些调整大小边距。

### 如何从死锁屏幕/红屏中恢复？

当您的屏幕锁死时，您将留下红屏。
这是niri锁定会话的背景。

您可以通过生成新的屏幕锁从这种情况中恢复。
一种方法是切换到不同的TTY（使用快捷键如<kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F3</kbd>），然后生成屏幕锁到niri的Wayland显示，例如`WAYLAND_DISPLAY=wayland-1 swaylock`。

另一种方法是在屏幕锁绑定上设置`allow-when-locked=true`，然后您可以在红屏上按下它以获得新的屏幕锁。

```kdl
binds {
    Super+Alt+L allow-when-locked=true { spawn "swaylock"; }
}
```

### 如何根据连接的显示器更改输出配置？

如果您需要根据连接的输出而使用不同的输出配置，则可以使用[Kanshi](https://gitlab.freedesktop.org/emersion/kanshi)。

Kanshi有自己的简单配置，并通过IPC与niri通信。您可能希望从niri config.kdl启动kanshi，例如`spawn-at-startup "/usr/bin/kanshi"`

例如，如果您希望在连接外部显示器时以不同方式缩放笔记本电脑显示器，您可能会使用这样的Kanshi配置：

    profile {
    	output eDP-1 enable scale 1.0
    }

    profile { 
    	output HDMI-A-1 enable scale 1.0 position 0,0
    	output eDP-1 enable scale 1.25 position 1920,0
    }