### 概述

在`layout {}`部分中，您可以更改影响窗口定位和大小的各种设置。

以下是此部分的内容概览：

```kdl
layout {
    gaps 16
    center-focused-column "never"
    always-center-single-column
    empty-workspace-above-first
    default-column-display "tabbed"
    background-color "#003300"

    preset-column-widths {
        proportion 0.33333
        proportion 0.5
        proportion 0.66667
    }

    default-column-width { proportion 0.5; }

    preset-window-heights {
        proportion 0.33333
        proportion 0.5
        proportion 0.66667
    }

    focus-ring {
        // off
        on
        width 4
        active-color "#7fc8ff"
        inactive-color "#505050"
        urgent-color "#9b0000"
        // active-gradient from="#80c8ff" to="#bbddff" angle=45
        // inactive-gradient from="#505050" to="#808080" angle=45 relative-to="workspace-view"
        // urgent-gradient from="#800" to="#a33" angle=45
    }

    border {
        off
        // on
        width 4
        active-color "#ffc87f"
        inactive-color "#505050"
        urgent-color "#9b0000"
        // active-gradient from="#ffbb66" to="#ffc880" angle=45 relative-to="workspace-view"
        // inactive-gradient from="#505050" to="#808080" angle=45 relative-to="workspace-view" in="srgb-linear"
        // urgent-gradient from="#800" to="#a33" angle=45
    }

    shadow {
        off
        // on
        softness 30
        spread 5
        offset x=0 y=5
        draw-behind-window true
        color "#00000070"
        // inactive-color "#00000054"
    }

    tab-indicator {
        // off
        on
        hide-when-single-tab
        place-within-column
        gap 5
        width 4
        length total-proportion=1.0
        position "right"
        gaps-between-tabs 2
        corner-radius 8
        active-color "red"
        inactive-color "gray"
        urgent-color "blue"
        // active-gradient from="#80c8ff" to="#bbddff" angle=45
        // inactive-gradient from="#505050" to="#808080" angle=45 relative-to="workspace-view"
        // urgent-gradient from="#800" to="#a33" angle=45
    }

    insert-hint {
        // off
        on
        color "#ffc87f80"
        // gradient from="#ffbb6680" to="#ffc88080" angle=45 relative-to="workspace-view"
    }

    struts {
        // left 64
        // right 64
        // top 64
        // bottom 64
    }
}
```

<sup>自：下一个版本</sup> 您可以为特定[输出](./Configuration:-Outputs#layout-config-overrides)和[命名工作区](./Configuration:-Named-Workspaces#layout-config-overrides)覆盖这些设置。

### `gaps`

在窗口周围（内部和外部）设置逻辑像素的间距。

<sup>自：0.1.7</sup> 您可以使用小数值。
该值将根据每个输出的缩放因子四舍五入为物理像素。
例如，在`scale 2`的输出上`gaps 0.5`将导致一个物理像素宽的间距。

<sup>自：0.1.8</sup> 您可以使用负`struts`值模拟"内部"与"外部"间距（参见下面的struts部分）。

```kdl
layout {
    gaps 16
}
```

### `center-focused-column`

更改焦点时何时居中列。
这可以设置为：

*   `"never"`：无特殊居中，聚焦屏幕外的列将滚动到屏幕的左侧或右侧边缘。这是默认值。
*   `"always"`，聚焦的列将始终居中。
*   `"on-overflow"`，如果列与先前聚焦的列不能同时适合屏幕，则聚焦列将居中。

```kdl
layout {
    center-focused-column "always"
}
```

### `always-center-single-column`

<sup>自：0.1.9</sup>

如果设置，niri将始终居中工作区上的单列，无论`center-focused-column`选项如何。

```kdl
layout {
    always-center-single-column
}
```

### `empty-workspace-above-first`

<sup>自：25.01</sup>

如果设置，niri将在最开始始终添加一个空工作区，除了最末尾的空工作区。

```kdl
layout {
    empty-workspace-above-first
}
```

### `default-column-display`

<sup>自：25.02</sup>

设置新列的默认显示模式。
可以是`normal`或`tabbed`。

```kdl
// 使所有新列默认为标签页。
layout {
    default-column-display "tabbed"

    // 您可能还希望在列中只有一个窗口时隐藏标签指示器。
    tab-indicator {
        hide-when-single-tab
    }
}
```

### `preset-column-widths`

设置`switch-preset-column-width`操作（Mod+R）在之间切换的宽度。

`proportion`将宽度设置为输出宽度的分数，考虑间距。
例如，无论间距设置如何，您都可以在输出上完美地容纳四个大小为`proportion 0.25`的窗口。
默认预设宽度是输出的<sup>1</sup>⁄<sub>3</sub>、<sup>1</sup>⁄<sub>2</sub>和<sup>2</sup>⁄<sub>3</sub>。

`fixed`精确地将窗口宽度设置为逻辑像素。

```kdl
layout {
    // 在输出的1/3、1/2、2/3和固定1280逻辑像素之间循环。
    preset-column-widths {
        proportion 0.33333
        proportion 0.5
        proportion 0.66667
        fixed 1280
    }
}
```

### `default-column-width`

设置新窗口的默认宽度。

语法与上面的`preset-column-widths`相同。

```kdl
layout {
    // 打开大小为输出1/3的新窗口。
    default-column-width { proportion 0.33333; }
}
```

您也可以将括号留空，然后窗口本身将决定其初始宽度。

```kdl
layout {
    // 新窗口自行决定其初始宽度。
    default-column-width {}
}
```

> \[!NOTE]
> `default-column-width {}`导致niri在初始配置请求中发送(0, H)大小。
>
> 这在Wayland协议中有点[定义不清](https://gitlab.freedesktop.org/wayland/wayland-protocols/-/issues/155)，所以一些客户端可能误解它。
> 无论如何，`default-column-width {}`对于特定窗口最有用，以[窗口规则](./Configuration:-Window-Rules#default-column-width)的形式具有相同语法。

### `preset-window-heights`

<sup>自：0.1.9</sup>

设置`switch-preset-window-height`操作（Mod+Shift+R）在之间切换的高度。

`proportion`将高度设置为输出高度的分数，考虑间距。
默认预设高度是输出的<sup>1</sup>⁄<sub>3</sub>、<sup>1</sup>⁄<sub>2</sub>和<sup>2</sup>⁄<sub>3</sub>。

`fixed`精确地将高度设置为逻辑像素。

```kdl
layout {
    // 在输出的1/3、1/2、2/3和固定720逻辑像素之间循环。
    preset-window-heights {
        proportion 0.33333
        proportion 0.5
        proportion 0.66667
        fixed 720
    }
}
```

### `focus-ring`和`border`

焦点环和边框绘制在窗口周围并指示活动窗口。
它们非常相似并且具有相同的选项。

区别在于焦点环仅绘制在活动窗口周围，而边框绘制在所有窗口周围并影响它们的大小（窗口缩小以为边框腾出空间）。

| 焦点环 | 边框 |
| ------------------------- | --------------------- |
| ![显示中心行聚焦图像使用焦点环的截图](./img/focus-ring.png) | ![显示中心行聚焦图像使用边框的截图，而顶部和底部窗口具有非活动颜色](./img/border.png) |

> \[!TIP]
> 默认情况下，焦点环和边框渲染为窗口后面的纯色背景矩形。
> 即，它们将透过半透明窗口显示。
> 这是因为使用客户端装饰的窗口可以具有任意形状。
>
> 如果您不喜欢这样，您应该取消注释配置顶层的[`prefer-no-csd`设置](./Configuration:-Miscellaneous#prefer-no-csd)。
> Niri将绘制同意省略其客户端装饰的窗口的*周围*焦点环和边框。
>
> 或者，您可以使用[`draw-border-with-background`窗口规则](./Configuration:-Window-Rules#draw-border-with-background)覆盖此行为。

焦点环和边框具有以下选项。

```kdl
layout {
    // focus-ring具有相同的选项。
    border {
        // 取消注释此行以禁用边框。
        // off

        // 边框的宽度（逻辑像素）。
        width 4

        active-color "#ffc87f"
        inactive-color "#505050"

        // 请求您注意的窗口周围的边框颜色。
        urgent-color "#9b0000"

        // active-gradient from="#ffbb66" to="#ffc880" angle=45 relative-to="workspace-view"
        // inactive-gradient from="#505050" to="#808080" angle=45 relative-to="workspace-view" in="srgb-linear"
    }
}
```

#### 宽度

以逻辑像素设置边框的厚度。

<sup>自：0.1.7</sup> 您可以使用小数值。
该值将根据每个输出的缩放因子四舍五入为物理像素。
例如，在`scale 2`的输出上`width 0.5`将导致一个物理像素厚的边框。

```kdl
layout {
    border {
        width 2
    }
}
```

#### 颜色

颜色可以通过多种方式设置：

*   CSS命名颜色："red"
*   RGB十六进制："#rgb"、"#rgba"、"#rrggbb"、"#rrggbbaa"
*   CSS类表示法："rgb(255, 127, 0)"、"rgba()"、"hsl()"和一些其他。

`active-color`是活动窗口周围的焦点环/边框的颜色，`inactive-color`是所有其他窗口周围的焦点环/边框的颜色。

*焦点环*仅在每个显示器上的活动窗口周围绘制，所以使用单个显示器时您永远不会看到其`inactive-color`。
不过，如果您有多个显示器，您将看到它。

还有一个*已弃用*的语法，使用四个数字设置颜色，分别代表R、G、B和A：`active-color 127 200 255 255`。

#### 渐变

类似于颜色，您可以设置`active-gradient`和`inactive-gradient`，它们将优先。

渐变渲染与CSS [`linear-gradient(angle, from, to)`](https://developer.mozilla.org/en-US/docs/Web/CSS/gradient/linear-gradient)相同。
角度与`linear-gradient`相同，是可选的，默认为`180`（从上到下的渐变）。
您可以在网上使用任何CSS线性渐变工具来设置它们，如[css-gradient.com](https://www.css-gradient.com/)。

```kdl
layout {
    focus-ring {
        active-gradient from="#80c8ff" to="#bbddff" angle=45
    }
}
```

渐变可以相对于窗口单独着色（默认），或者相对于整个工作区视图着色。
要这样做，请设置`relative-to="workspace-view"`。
这是一个视觉示例：

| 默认 | `relative-to="workspace-view"` |
| -------------------------------- | --------------------------------------------------- |
| ![显示4个窗口的截图，每个窗口都有单独的渐变边框](./img/gradients-default.png) | ![显示4个窗口的截图，边框上有共享渐变](./img/gradients-relative-to-workspace-view.png) |

```kdl
layout {
    border {
        active-gradient from="#ffbb66" to="#ffc880" angle=45 relative-to="workspace-view"
        inactive-gradient from="#505050" to="#808080" angle=45 relative-to="workspace-view"
    }
}
```

<sup>自：0.1.8</sup> 您可以使用语法如`in="srgb-linear"`或`in="oklch longer hue"`设置渐变插值颜色空间。
支持的颜色空间有：

*   `srgb`（默认），
*   `srgb-linear`，
*   `oklab`，
*   `oklch`，具有`shorter hue`、`longer hue`、`increasing hue`或`decreasing hue`。

它们与CSS渲染相同。
例如，`active-gradient from="#f00f" to="#0f05" angle=45 in="oklch longer hue"`将看起来与CSS `linear-gradient(45deg in oklch longer hue, #f00f, #0f05)`相同。

![显示使用oklch颜色空间渐变边框的窗口截图](./img/gradients-oklch.png)

```kdl
layout {
    border {
        active-gradient from="#f00f" to="#0f05" angle=45 in="oklch longer hue"
    }
}
```

### `shadow`

<sup>自：25.02</sup>

窗口后面的阴影。

设置`on`以启用阴影。

`softness`控制阴影的柔和度/大小（逻辑像素），与[CSS box-shadow] *模糊半径*相同。
设置`softness 0`将给您硬阴影。

`spread`是以逻辑像素扩展窗口矩形的距离，与CSS box-shadow展开相同。<sup>自：25.05</sup> 展开可以为负。

`offset`以逻辑像素移动相对于窗口的阴影，与CSS box-shadow偏移相同。
例如，`offset x=2 y=2`将阴影向右下方移动2个逻辑像素。

设置`draw-behind-window`为`true`以使阴影绘制在窗口后面而不仅仅是围绕它。
请注意，niri无法了解CSD窗口圆角半径。
它必须假设窗口有方形角，导致CSD圆角内部出现阴影伪影。
此设置修复这些伪影。

然而，您可能想设置`prefer-no-csd`和/或`geometry-corner-radius`。
然后，niri将知道圆角半径并正确绘制阴影，无需绘制在窗口后面。
这些也会移除窗口绘制的任何客户端阴影。

`color`是阴影颜色和不透明度。

`inactive-color`允许您覆盖非活动窗口的阴影颜色；默认情况下，使用更透明的`color`。

阴影绘制将跟随使用[`geometry-corner-radius`窗口规则](./Configuration:-Window-Rules#geometry-corner-radius)设置的窗口圆角半径。

> \[!NOTE]
> 目前，阴影绘制仅支持所有角匹配半径。如果您设置`geometry-corner-radius`为四个值而不是一个，第一个（左上角）圆角半径将用于阴影。

```kdl
// 启用阴影。
layout {
    shadow {
        on
    }
}

// 还要求窗口省略客户端装饰，
// 这样它们就不会绘制自己的窗口阴影。
prefer-no-csd
```

[CSS box-shadow]: https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow

### `tab-indicator`

<sup>自：25.02</sup>

控制标签页显示模式下出现在列旁边的标签指示器的外观。

设置`off`以隐藏标签指示器。

设置`hide-when-single-tab`以隐藏仅有一个窗口的标签页列的指示器。

设置`place-within-column`以将标签指示器放在列"内"，而不是外部。
这将包括列大小调整并避免覆盖相邻列。

`gap`设置标签指示器和窗口之间的逻辑像素间距。
间距可以为负，这将把标签指示器放在窗口顶部。

`width`设置指示器的厚度（逻辑像素）。

`length`控制指示器的长度。
设置`total-proportion`属性使标签相对于窗口大小占用这么多长度。
默认情况下，标签指示器的长度等于窗口大小的一半，或`length total-proportion=0.5`。

`position`设置标签指示器相对于窗口的位置。
可以是`left`、`right`、`top`或`bottom`。

`gaps-between-tabs`控制单个标签之间的逻辑像素间距。

`corner-radius`设置指示器中标签的圆角半径（逻辑像素）。
当`gaps-between-tabs`为零时，只有第一个和最后一个标签有圆角，否则所有标签都有。

`active-color`、`inactive-color`、`urgent-color`、`active-gradient`、`inactive-gradient`、`urgent-gradient`允许您覆盖标签的颜色。
它们具有与边框和焦点环颜色和渐变相同的语义。

标签颜色按此顺序选择：

1.  如果设置了`tab-indicator`窗口规则的颜色。
2.  如果设置了`tab-indicator`布局选项的颜色（您在这里）。
3.  如果两者都没有设置，niri选择匹配窗口边框或焦点环的颜色，无论哪个处于活动状态。

```kdl
// 使标签指示器更宽并匹配窗口高度，
// 还将它放在顶部并在列内。
layout {
    tab-indicator {
        width 8
        gap 8
        length total-proportion=1.0
        position "top"
        place-within-column
    }
}
```

### `insert-hint`

<sup>自：0.1.10</sup>

交互式窗口移动期间窗口插入位置提示的设置。

`off`完全禁用插入提示。

`color`和`gradient`允许您更改提示颜色，具有与边框和焦点环中颜色和渐变相同的语法。

```kdl
layout {
    insert-hint {
        // off
        color "#ffc87f80"
        gradient from="#ffbb6680" to="#ffc88080" angle=45 relative-to="workspace-view"
    }
}
```

### `struts`

Struts缩小窗口占用的区域，类似于layer-shell面板。
您可以将它们视为一种外部间距。
它们以逻辑像素设置。

左右struts将导致侧边的下一个窗口总是稍微露出。
上下struts将简单地在layer-shell面板和常规间距占用的区域之外添加外部间距。

<sup>自：0.1.7</sup> 您可以使用小数值。
该值将根据每个输出的缩放因子四舍五入为物理像素。
例如，在`scale 2`的输出上`top 0.5`将导致一个物理像素宽的顶部strut。

```kdl
layout {
    struts {
        left 64
        right 64
        top 64
        bottom 64
    }
}
```

![说明struts效果的截图，如本节第二段所述](./img/struts.png)

<sup>自：0.1.8</sup> 您可以使用负值。
它们将窗口向外推，甚至推到屏幕边缘之外。

您可以使用与间距值匹配的负struts来模拟"内部"与"外部"间距。
例如，使用此设置内部间距而无外部间距：

```kdl
layout {
    gaps 16

    struts {
        left -16
        right -16
        top -16
        bottom -16
    }
}
```

### `background-color`

<sup>自：25.05</sup>

设置niri为工作区绘制的默认背景颜色。
当您不使用任何背景工具（如swaybg）时可见。

```kdl
layout {
    background-color "#003300"
}
```

您还可以在[输出配置中](./Configuration:-Outputs#layout-config-overrides)按输出设置颜色。