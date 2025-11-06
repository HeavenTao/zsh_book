### 概述

默认情况下，niri将尝试使用其首选模式打开所有连接的显示器。
您可以使用`output`部分禁用或调整此设置。

这是写入所有属性的样子：

```kdl
output "eDP-1" {
    // off
    mode "1920x1080@120.030"
    scale 2.0
    transform "90"
    position x=1280 y=0
    variable-refresh-rate // on-demand=true
    focus-at-startup
    backdrop-color "#001100"

    hot-corners {
        // off
        top-left
        // top-right
        // bottom-left
        // bottom-right
    }

    layout {
        // ...eDP-1的布局设置...
    }

    // 自定义模式。注意：可能损坏您的显示器。
    // mode custom=true "1920x1080@100"
    // modeline 173.00  1920 2048 2248 2576  1080 1083 1088 1120 "-hsync" "+vsync"
}

output "HDMI-A-1" {
    // ...HDMI-A-1的设置...
}

output "Some Company CoolMonitor 1234" {
    // ...CoolMonitor的设置...
}
```

输出通过连接器名称（即`eDP-1`、`HDMI-A-1`）或显示器制造商、型号和序列号匹配，每个用单个空格分隔。
您可以通过运行`niri msg outputs`找到所有这些。

通常，笔记本电脑中的内置显示器将被称为`eDP-1`。

<sup>自：0.1.6</sup> 输出名称不区分大小写。

<sup>自：0.1.9</sup> 输出可以通过制造商、型号和序列号匹配。
之前，它们只能通过连接器名称匹配。

### `off`

此标志完全关闭该输出。

```kdl
// 关闭该显示器。
output "HDMI-A-1" {
    off
}
```

### `mode`

设置显示器分辨率和刷新率。

格式为`<width>x<height>`或`<width>x<height>@<refresh rate>`。
如果省略刷新率，niri将为该分辨率选择最高刷新率。

如果完全省略模式或模式不起作用，niri将尝试自动选择一个。

在niri实例中运行`niri msg outputs`以列出所有输出及其模式。
您在此设置的刷新率必须与您在`niri msg outputs`中看到的完全匹配，精确到三位小数。

```kdl
// 为此显示器设置高刷新率。
// 高刷新率显示器倾向于使用60 Hz作为其首选模式，
// 需要手动模式设置。
output "HDMI-A-1" {
    mode "2560x1440@143.912"
}

// 在内置笔记本电脑显示器上使用较低分辨率
// （例如，用于测试目的）。
output "eDP-1" {
    mode "1280x720"
}
```

#### `mode custom=true`

<sup>自：下一个版本</sup>

您可以通过设置`custom=true`来配置自定义模式（显示器未提供）。
在这种情况下，刷新率是强制的。

> \[!CAUTION]
> 自定义模式可能损坏您的显示器，特别是如果它是CRT。
> 请遵循显示器说明中的最大支持限制。

```kdl
// 为此显示器使用自定义模式。
output "HDMI-A-1" {
    mode custom=true "2560x1440@143.912"
}
```

### `modeline`

<sup>自：下一个版本</sup>

通过模型线直接配置显示器的模式，覆盖任何配置的`mode`。
模型线可以通过[cvt](https://man.archlinux.org/man/cvt.1.en)或[gtf](https://man.archlinux.org/man/gtf.1.en)等工具计算。

> \[!CAUTION]
> 超出规范的模型线可能损坏您的显示器，特别是如果它是CRT。
> 请遵循显示器说明中的最大支持限制。

```kdl
// 为此显示器使用模型线。
output "eDP-3" {
    modeline 173.00  1920 2048 2248 2576  1080 1083 1088 1120 "-hsync" "+vsync"
}
```

### `scale`

设置显示器的缩放。

<sup>自：0.1.6</sup> 如果未设置缩放，niri将根据显示器的物理尺寸和分辨率猜测适当的缩放。

<sup>自：0.1.7</sup> 您可以使用小数缩放值，例如`scale 1.5`表示150%缩放。

<sup>自：0.1.7</sup> 整数缩放不再需要点，例如您可以写`scale 2`而不是`scale 2.0`。

<sup>自：0.1.7</sup> 0以下和10以上的缩放现在将在配置解析期间失败。缩放之前无论如何都被限制在这些值。

```kdl
output "eDP-1" {
    scale 2.0
}
```

### `transform`

逆时针旋转输出。

有效值为："normal"、"90"、"180"、"270"、"flipped"、"flipped-90"、"flipped-180"和"flipped-270"。
带`flipped`的值还会翻转输出。

```kdl
output "HDMI-A-1" {
    transform "90"
}
```

### `position`

在全局坐标空间中设置输出的位置。

这会影响方向性显示器操作，如`focus-monitor-left`，以及光标移动。
光标只能在直接相邻的输出之间移动。

> \[!NOTE]
> 输出缩放和旋转必须考虑定位：输出以逻辑或缩放像素为单位进行大小调整。
> 例如，具有缩放2.0的3840×2160输出将具有1920×1080的逻辑大小，所以要在其右侧直接放置另一个输出，将其x设置为1920。
> 如果位置未设置或导致重叠，则输出将自动放置。

```kdl
output "HDMI-A-1" {
    position x=1280 y=0
}
```

#### 自动定位

每次输出配置更改时（包括显示器断开和连接），niri都会重新定位输出。
使用以下算法定位输出。

1.  收集所有连接的显示器及其逻辑大小。
2.  按名称对它们进行排序。这使得自动定位不依赖于显示器连接的顺序。这很重要，因为连接顺序在合成器启动时是非确定性的。
3.  尝试按顺序放置每个明确配置`position`的输出。如果输出与之前放置的输出重叠，则将其放置在所有之前放置的输出的右侧。在这种情况下，niri还将打印警告。
4.  将每个没有明确配置`position`的输出放置在所有之前放置的输出的右侧。

### `variable-refresh-rate`

<sup>自：0.1.5</sup>

如果输出支持，此标志启用可变刷新率（VRR，也称为自适应同步、FreeSync或G-Sync）。

您可以在`niri msg outputs`中检查输出是否支持VRR。

> \[!NOTE]
> 一些驱动程序对VRR有各种问题。
>
> 如果光标在VRR下以低帧率移动，尝试设置[`disable-cursor-plane`调试标志](./Configuration:-Debug-Options#disable-cursor-plane)并重新连接显示器。
>
> 如果显示器应该被检测为VRR-capable但没有，有时拔掉不同的显示器可以修复它。
>
> 一些显示器在启用VRR时会持续模式设置（闪黑）；我不确定是否有办法修复它。

```kdl
output "HDMI-A-1" {
    variable-refresh-rate
}
```

<sup>自：0.1.9</sup> 您还可以设置`on-demand=true`属性，这将仅在此输出显示匹配`variable-refresh-rate`窗口规则的窗口时启用VRR。
这有助于避免VRR的各种问题，因为它可以在大多数时间禁用，仅在特定窗口（如游戏或视频播放器）时启用。

```kdl
output "HDMI-A-1" {
    variable-refresh-rate on-demand=true
}
```

### `focus-at-startup`

<sup>自：25.05</sup>

niri启动时默认聚焦此输出。

如果连接多个带有`focus-at-startup`的输出，它们将按照它们在配置中出现的顺序优先。

当连接的输出中没有明确`focus-at-startup`时，niri将聚焦按名称排序的第一个（与niri中其他地方使用的输出排序相同）。

```kdl
// 默认聚焦HDMI-A-1。
output "HDMI-A-1" {
    focus-at-startup
}

// ...如果HDMI-A-1未连接，则聚焦DP-2。
output "DP-2" {
    focus-at-startup
}
```

### `background-color`

<sup>自：0.1.8</sup>

设置niri为此输出上的工作区绘制的背景颜色。
当您不使用任何背景工具（如swaybg）时可见。

<sup>直到：25.05</sup> 此颜色的alpha通道将被忽略。

<sup>自：下一个版本</sup> 此设置已弃用，请在[输出`layout {}`块](#layout-config-overrides)中设置`background-color`。

```kdl
output "HDMI-A-1" {
    background-color "#003300"
}
```

### `backdrop-color`

<sup>自：25.05</sup>

设置niri为此输出绘制的背景颜色。
在工作区之间或概览中可见。

此颜色的alpha通道将被忽略。

```kdl
output "HDMI-A-1" {
    backdrop-color "#001100"
}
```

### `hot-corners`

<sup>自：下一个版本</sup>

为此输出自定义热角。
默认情况下，热角[在手势设置中](./Configuration:-Gestures#hot-corners)用于所有输出。

当您将鼠标放在显示器的角落时，热角切换概览。

`off`将在此输出上禁用热角，写入特定角将仅在此输出上启用这些热角。

```kdl
// 在HDMI-A-1上启用左下角和右下角热角。
output "HDMI-A-1" {
    hot-corners {
        bottom-left
        bottom-right
    }
}

// 在DP-2上禁用热角。
output "DP-2" {
    hot-corners {
        off
    }
}
```

### 布局配置覆盖

<sup>自：下一个版本</sup>

您可以使用`layout {}`块自定义输出的布局设置：

```kdl
output "SomeCompany VerticalMonitor 1234" {
    transform "90"

    // 仅适用于此输出的布局配置覆盖。
    layout {
        default-column-width { proportion 1.0; }

        // ...任何其他设置。
    }
}

output "SomeCompany UltrawideMonitor 1234" {
    // 为超宽显示器设置更窄的比例和更多预设。
    layout {
        default-column-width { proportion 0.25; }

        preset-column-widths {
            proportion 0.2
            proportion 0.25
            proportion 0.5
            proportion 0.75
            proportion 0.8
        }
    }
}
```

它接受与[顶层`layout {}`块](./Configuration:-Layout)相同的所有选项。

为了取消设置标志，请用`false`写入，例如：

```kdl
layout {
    // 全局启用。
    always-center-single-column
}

output "eDP-1" {
    layout {
        // 在此输出上取消设置。
        always-center-single-column false
    }
}
```