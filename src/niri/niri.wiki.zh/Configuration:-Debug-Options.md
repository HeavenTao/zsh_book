### 概述

Niri有几个选项仅对调试有用，或者处于实验阶段且有已知问题。
它们不适用于正常使用。

> \[!CAUTION]
> 这些选项**不**受[配置破坏性变更政策](./Configuration:-Introduction#breaking-change-policy)的保护。
> 它们可能在任何时间以很少的通知更改或停止工作。

这是所有选项的概览：

```kdl
debug {
    preview-render "screencast"
    // preview-render "screen-capture"
    enable-overlay-planes
    disable-cursor-plane
    disable-direct-scanout
    restrict-primary-scanout-to-matching-format
    render-drm-device "/dev/dri/renderD129"
    ignore-drm-device "/dev/dri/renderD128"
    ignore-drm-device "/dev/dri/renderD130"
    force-pipewire-invalid-modifier
    dbus-interfaces-in-non-session-instances
    wait-for-frame-completion-before-queueing
    emulate-zero-presentation-time
    disable-resize-throttling
    disable-transactions
    keep-laptop-panel-on-when-lid-is-closed
    disable-monitor-names
    strict-new-window-focus-policy
    honor-xdg-activation-with-invalid-serial
    skip-cursor-only-updates-during-vrr
    deactivate-unfocused-windows
    keep-max-bpc-unchanged
}

binds {
    Mod+Shift+Ctrl+T { toggle-debug-tint; }
    Mod+Shift+Ctrl+O { debug-toggle-opaque-regions; }
    Mod+Shift+Ctrl+D { debug-toggle-damage; }
}
```

### `preview-render`

使niri以与屏幕录制或屏幕捕获相同的方式渲染显示器。

用于预览`block-out-from`窗口规则。

```kdl
debug {
    preview-render "screencast"
    // preview-render "screen-capture"
}
```

### `enable-overlay-planes`

启用直接扫描输出到覆盖平面。
在某些硬件上的某些动画期间可能导致帧丢失（这就是为什么它不是默认设置）。

直接扫描输出到主平面始终启用。

```kdl
debug {
    enable-overlay-planes
}
```

### `disable-cursor-plane`

禁用光标平面的使用。
光标将与帧的其余部分一起渲染。

用于解决特定硬件上的驱动程序错误。

```kdl
debug {
    disable-cursor-plane
}
```

### `disable-direct-scanout`

禁用直接扫描输出到主平面和覆盖平面。

```kdl
debug {
    disable-direct-scanout
}
```

### `restrict-primary-scanout-to-matching-format`

将直接扫描输出到主平面限制在窗口缓冲区与合成交换链格式完全匹配时。

此标志可能在合成和扫描输出之间转换时防止意外的带宽变化。
计划在我们实现告诉客户端合成交换链格式的方法时使其成为默认设置。
目前，它可能阻止某些客户端（我机器上的mpv）扫描输出到主平面。

```kdl
debug {
    restrict-primary-scanout-to-matching-format
}
```

### `render-drm-device`

覆盖niri将用于所有渲染的DRM设备。

您可以将其设置为使niri使用与默认GPU不同的主GPU。

```kdl
debug {
    render-drm-device "/dev/dri/renderD129"
}
```

### `ignore-drm-device`

<sup>自：下一个版本</sup>

列出niri将忽略的DRM设备。
在GPU透传时，当您不希望niri打开特定设备时有用。

```kdl
debug {
    ignore-drm-device "/dev/dri/renderD128"
    ignore-drm-device "/dev/dri/renderD130"
}
```

### `force-pipewire-invalid-modifier`

<sup>自：25.01</sup>

强制PipeWire屏幕录制使用无效修饰符，即使DRM提供更多的修饰符。

用于测试那些不支持修饰符的驱动程序会遇到的无效修饰符代码路径。

```kdl
debug {
    force-pipewire-invalid-modifier
}
```

### `dbus-interfaces-in-non-session-instances`

使niri创建其D-Bus接口，即使它没有作为`--session`运行。

用于在无需重新登录的情况下测试屏幕录制更改。

主niri实例*目前*不会在您关闭测试实例时取回接口，因此您最终需要重新登录以使屏幕录制再次工作。

```kdl
debug {
    dbus-interfaces-in-non-session-instances
}
```

### `wait-for-frame-completion-before-queueing`

在将每个帧交给DRM之前等待它完成渲染。

用于诊断某些同步和性能问题。

```kdl
debug {
    wait-for-frame-completion-before-queueing
}
```

### `emulate-zero-presentation-time`

模拟DRM返回的零（未知）呈现时间。

这是NVIDIA专有驱动程序上的一个问题，所以此标志可用于测试niri在这些系统上不会严重崩溃。

```kdl
debug {
    emulate-zero-presentation-time
}
```

### `disable-resize-throttling`

<sup>自：0.1.9</sup>

禁用发送到窗口的调整大小事件的节流。

默认情况下，当快速调整大小（例如交互式地）时，窗口只有在为之前请求的大小进行提交后才会接收下一个大小。
这对于调整大小事务正常工作是必需的，它还有助于某些不批处理来自合成器的传入调整大小的客户端。

禁用调整大小节流将尽可能快地向窗口发送调整大小，这可能非常快（例如，对于1000 Hz鼠标）。

```kdl
debug {
    disable-resize-throttling
}
```

### `disable-transactions`

<sup>自：0.1.9</sup>

禁用事务（调整大小和关闭）。

默认情况下，必须一起调整大小的窗口，确实一起调整大小。
例如，列中的所有窗口必须同时调整大小以保持组合列高度等于屏幕高度，并保持相同的窗口宽度。

事务使niri等待所有窗口完成调整大小，然后在同步帧中将它们全部显示在屏幕上。
为了使它们正常工作，调整大小节流不应被禁用（使用前面的调试标志）。

```kdl
debug {
    disable-transactions
}
```

### `keep-laptop-panel-on-when-lid-is-closed`

<sup>自：0.1.10</sup>

默认情况下，当笔记本电脑盖子关闭时，niri将禁用内置笔记本电脑显示器。
此标志关闭此行为并将保持内置笔记本电脑显示器开启。

```kdl
debug {
    keep-laptop-panel-on-when-lid-is-closed
}
```

### `disable-monitor-names`

<sup>自：0.1.10</sup>

禁用制造商/型号/序列号显示器名称，就像niri无法从EDID读取它们一样。

使用此标志解决0.1.9和0.1.10中连接两个具有匹配制造商/型号/序列号的显示器时出现的崩溃。

```kdl
debug {
    disable-monitor-names
}
```

### `strict-new-window-focus-policy`

<sup>自：25.01</sup>

禁用新窗口的启发式自动聚焦。
只有使用有效xdg-activation令牌激活自己的窗口才会被聚焦。

```kdl
debug {
    strict-new-window-focus-policy
}
```

### `honor-xdg-activation-with-invalid-serial`

<sup>自：25.05</sup>

像Discord和Telegram这样的广泛使用的客户端在点击其托盘图标或通知时会制作新的xdg-activation令牌。
大多数时候，这些新令牌将有无效序列号，因为应用程序需要被聚焦才能获得有效序列号，而如果用户点击托盘图标或通知，通常是因为应用程序*没有*被聚焦，用户想要聚焦它。

默认情况下，niri忽略具有无效序列号的xdg-activation令牌，以防止窗口随机窃取焦点。
此调试标志使niri遵守此类令牌，使上述广泛使用的应用程序在点击其托盘图标或通知时获得焦点。

有趣的是，点击通知会向应用程序发送一个完全有效的激活令牌，来自通知守护程序，但这些应用程序似乎只是忽略它。
也许将来这些应用程序/工具包（Electron，Qt）会被修复，使此调试标志变得不必要。

```kdl
debug {
    honor-xdg-activation-with-invalid-serial
}
```

### `skip-cursor-only-updates-during-vrr`

<sup>自：25.08</sup>

在变量刷新率激活时跳过光标输入的屏幕重绘。

用于游戏内部不绘制光标的情况，以防止对光标移动的不稳定VRR变化。

请注意，当前实现有一些问题，例如当没有重绘屏幕时（比如游戏），渲染将看起来完全冻结（因为光标移动不会导致重绘）。

```kdl
debug {
    skip-cursor-only-updates-during-vrr
}
```

### `deactivate-unfocused-windows`

<sup>自：25.08</sup>

某些客户端（特别是基于Chromium和Electron的，如Teams或Slack）错误地使用Activated xdg窗口状态而不是键盘焦点来处理如决定是否为新消息发送通知，或选择在哪里显示IME弹出窗口等事情。
Niri在未聚焦的工作区和不可见的标签窗口上保持Activated状态（以减少不必要的动画），暴露了这些应用程序中的错误。

设置此调试标志以解决这些问题。
它将导致niri丢弃所有未聚焦窗口的Activated状态。

```kdl
debug {
    deactivate-unfocused-windows
}
```

### `keep-max-bpc-unchanged`

<sup>自：25.08</sup>

连接显示器时，niri将它们的max bpc设置为8，以减少显示带宽并可能允许同时连接更多显示器。
将bpc限制为8不是问题，因为我们尚未支持HDR或颜色管理，无法真正利用更高的bpc。

显然，将max bpc设置为8会破坏一些由AMDGPU驱动的显示器。
如果这对您发生，请设置此调试标志，这将防止niri更改max bpc。
AMDGPU错误报告：https://gitlab.freedesktop.org/drm/amd/-/issues/4487。

```kdl
debug {
    keep-max-bpc-unchanged
}
```

### 键绑定

这些不是调试选项，而是键绑定。

#### `toggle-debug-tint`

将所有表面染成绿色，除非它们正在被直接扫描输出。

用于检查直接扫描输出是否正常工作。

```kdl
binds {
    Mod+Shift+Ctrl+T { toggle-debug-tint; }
}
```

#### `debug-toggle-opaque-regions`

<sup>自：0.1.6</sup>

将标记为不透明的区域染成蓝色，将渲染元素的其余部分染成红色。

用于检查Wayland表面和内部渲染元素如何将其部分标记为不透明，这是一种渲染性能优化。

```kdl
binds {
    Mod+Shift+Ctrl+O { debug-toggle-opaque-regions; }
}
```

#### `debug-toggle-damage`

<sup>自：0.1.6</sup>

将损坏的区域染成红色。

```kdl
binds {
    Mod+Shift+Ctrl+D { debug-toggle-damage; }
}
```