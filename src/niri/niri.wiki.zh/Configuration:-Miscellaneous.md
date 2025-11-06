此页面记录所有没有专门页面的顶级选项。

以下是所有这些选项的概览：

```kdl
spawn-at-startup "waybar"
spawn-at-startup "alacritty"
spawn-sh-at-startup "qs -c ~/source/qs/MyAwesomeShell"

prefer-no-csd

screenshot-path "~/Pictures/Screenshots/Screenshot from %Y-%m-%d %H-%M-%S.png"

environment {
    QT_QPA_PLATFORM "wayland"
    DISPLAY null
}

cursor {
    xcursor-theme "breeze_cursors"
    xcursor-size 48

    hide-when-typing
    hide-after-inactive-ms 1000
}

overview {
    zoom 0.5
    backdrop-color "#262626"

    workspace-shadow {
        // off
        softness 40
        spread 10
        offset x=0 y=10
        color "#00000050"
    }
}

xwayland-satellite {
    // off
    path "xwayland-satellite"
}

clipboard {
    disable-primary
}

hotkey-overlay {
    skip-at-startup
    hide-not-bound
}

config-notification {
    disable-failed
}
```

### `spawn-at-startup`

添加类似这样的行以在niri启动时启动进程。

`spawn-at-startup`接受程序二进制文件的路径作为第一个参数，后跟程序的参数。

此选项的工作方式与[`spawn`键绑定操作](./Configuration:-Key-Bindings#spawn)相同，因此请在那里阅读其所有细节。

```kdl
spawn-at-startup "waybar"
spawn-at-startup "alacritty"
```

请注意，作为systemd会话运行niri支持开箱即用的xdg-desktop-autostart，这可能更方便使用。
多亏了这一点，在GNOME中配置为自动启动的应用程序在niri中也会"正常工作"，无需任何手动`spawn-at-startup`配置。

### `spawn-sh-at-startup`

<sup>自：25.08</sup>

添加类似这样的行以在niri启动时运行shell命令。

参数是单个字符串，原样传递给`sh`。
您可以像预期的那样使用shell变量、管道、`~`扩展等。

详细描述请参见[`spawn-sh`键绑定操作](./Configuration:-Key-Bindings#spawn-sh)的文档。

```kdl
// 将所有参数放在同一个字符串中。
spawn-sh-at-startup "qs -c ~/source/qs/MyAwesomeShell"
```

### `prefer-no-csd`

此标志将使niri要求应用程序省略其客户端装饰。

如果应用程序特别要求CSD，将遵守该请求。
此外，客户端将被告知它们是平铺的，移除一些圆角。

设置`prefer-no-csd`后，通过xdg-decoration协议协商服务器端装饰的应用程序将在其周围绘制焦点环和边框，*没有*纯色背景。

> \[!NOTE]
> 与大多数其他选项不同，更改`prefer-no-csd`不会完全影响已运行的应用程序。
> 它会使一些窗口变成矩形，但不会移除标题栏。
> 这主要是因为niri解决了SDL2中的一个[bug](https://github.com/libsdl-org/SDL/issues/8173)，该bug阻止SDL2应用程序启动。
>
> 更改配置中的`prefer-no-csd`后重新启动应用程序以完全应用它。

```kdl
prefer-no-csd
```

### `screenshot-path`

设置截图保存的路径。
前面的`~`将扩展到主目录。

路径使用`strftime(3)`格式化以提供截图的日期和时间。

如果路径的最后一个文件夹不存在，Niri将创建它。

```kdl
screenshot-path "~/Pictures/Screenshots/Screenshot from %Y-%m-%d %H-%M-%S.png"
```

您也可以将此选项设置为`null`以禁用将截图保存到磁盘。

```kdl
screenshot-path null
```

### `environment`

覆盖niri启动的进程的环境变量。

```kdl
environment {
    // 像这样设置变量：
    // QT_QPA_PLATFORM "wayland"

    // 使用null作为值来移除变量：
    // DISPLAY null
}
```

### `cursor`

更改光标的主题和大小，并设置`XCURSOR_THEME`和`XCURSOR_SIZE`环境变量。

```kdl
cursor {
    xcursor-theme "breeze_cursors"
    xcursor-size 48
}
```

#### `hide-when-typing`

<sup>自：0.1.10</sup>

如果设置，按下键盘上的键时会隐藏光标。

> \[!NOTE]
> 此设置可能会干扰在原生Wayland模式下运行的Wine游戏，这些游戏使用鼠标观察，如第一人称游戏。
> 如果您同时按下一个键并移动鼠标时角色的视角跳下来，请尝试禁用此设置。

```kdl
cursor {
    hide-when-typing
}
```

#### `hide-after-inactive-ms`

<sup>自：0.1.10</sup>

如果设置，光标将在最后一次光标移动后经过指定的毫秒数时自动隐藏。

```kdl
cursor {
    // 在一秒钟不活动后隐藏光标。
    hide-after-inactive-ms 1000
}
```

### `overview`

<sup>自：25.05</sup>

[概览](./Overview)的设置。

#### `zoom`

控制概览中工作区的缩放程度。
`zoom`范围从0到0.75，值越低会使一切越小。

```kdl
// 在概览中使工作区比正常小四倍。
overview {
    zoom 0.25
}
```

#### `backdrop-color`

设置概览中工作区后面的背景颜色。
在切换工作区时，背景在工作区之间也是可见的。

此颜色的alpha通道将被忽略。

```kdl
// 使背景变亮。
overview {
    backdrop-color "#777777"
}
```

您还可以在[输出配置中](./Configuration:-Outputs#backdrop-color)按输出设置颜色。

#### `workspace-shadow`

控制在概览中可见的工作区后面的阴影。

这里的设置镜像布局部分中的正常[`shadow`配置](./Configuration:-Layout#shadow)，所以请查看那里的文档。

工作区阴影配置为标准化到1080像素高的工作区大小，然后与工作区一起缩小。
实际上，这意味着您需要比窗口阴影更大的展开、偏移和柔和度。

```kdl
// 在概览中禁用工作区阴影。
overview {
    workspace-shadow {
        off
    }
}
```

### `xwayland-satellite`

<sup>自：25.08</sup>

与[xwayland-satellite](https://github.com/Supreeeme/xwayland-satellite)集成的设置。

当检测到足够新的xwayland-satellite时，niri将创建X11套接字并设置`DISPLAY`，然后在X11客户端尝试连接时自动启动`xwayland-satellite`。
如果Xwayland死亡，niri将继续监视X11套接字并根据需要重新启动`xwayland-satellite`。
这与内置Xwayland在其他合成器中的工作方式非常相似。

`off`禁用集成：niri不会创建X11套接字，也不会设置`DISPLAY`环境变量。

`path`设置`xwayland-satellite`二进制文件的路径。
默认情况下，它只是`xwayland-satellite`，所以它像任何其他非绝对程序名一样被查找。

```kdl
// 使用自定义构建的xwayland-satellite。
xwayland-satellite {
    path "~/source/rs/xwayland-satellite/target/release/xwayland-satellite"
}
```

### `clipboard`

<sup>自：25.02</sup>

剪贴板设置。

设置`disable-primary`标志以禁用主要剪贴板（中键单击粘贴）。
切换此标志仅适用于之后启动的应用程序。

```kdl
clipboard {
    disable-primary
}
```

### `hotkey-overlay`

"重要热键"覆盖的设置。

#### `skip-at-startup`

如果您不想在niri启动时看到热键帮助，请设置`skip-at-startup`标志。

```kdl
hotkey-overlay {
    skip-at-startup
}
```

#### `hide-not-bound`

<sup>自：25.08</sup>

默认情况下，即使没有绑定到任何键，niri也会显示最重要的操作，以防止混淆。
如果您想隐藏所有未绑定到任何键的操作，请设置`hide-not-bound`标志。

```kdl
hotkey-overlay {
    hide-not-bound
}
```

您可以使用[`hotkey-overlay-title`属性](./Configuration:-Key-Bindings#custom-hotkey-overlay-titles)自定义热键覆盖显示的绑定。

### `config-notification`

<sup>自：25.08</sup>

配置创建/失败通知的设置。

设置`disable-failed`标志以禁用"无法解析配置文件"通知。
例如，如果您有自定义的通知。

```kdl
config-notification {
    disable-failed
}
```