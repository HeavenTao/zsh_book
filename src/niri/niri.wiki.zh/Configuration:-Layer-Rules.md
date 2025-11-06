### 概述

<sup>自：25.01</sup>

层规则允许您为单个layer-shell表面调整行为。
它们有`match`和`exclude`指令，控制规则应适用于哪些layer-shell表面，以及您可以设置的多个属性。

层规则的处理和工作方式与窗口规则非常相似，只是匹配器和属性不同。
请阅读[窗口规则wiki页面](./Configuration:-Window-Rules)以了解匹配是如何工作的。

以下是层规则可能具有的所有匹配器和属性：

```kdl
layer-rule {
    match namespace="waybar"
    match at-startup=true

    // 持续应用的属性。
    opacity 0.5
    block-out-from "screencast"
    // block-out-from "screen-capture"

    shadow {
        on
        // off
        softness 40
        spread 5
        offset x=0 y=5
        draw-behind-window true
        color "#00000064"
        // inactive-color "#00000064"
    }

    geometry-corner-radius 12
    place-within-backdrop true
    baba-is-float true
}
```

### 层表面匹配

让我们更详细地看看匹配器。

#### `namespace`

这是一个应该在表面命名空间中任何地方匹配的正则表达式。
您可以在这里阅读支持的正则表达式语法：[here](https://docs.rs/regex/latest/regex/#syntax)。

```kdl
// 匹配命名空间包含"waybar"的表面，
layer-rule {
    match namespace="waybar"
}
```

您可以通过运行`niri msg layers`找到所有打开的layer-shell表面的命名空间。

#### `at-startup`

可以是`true`或`false`。
在启动niri后的前60秒内匹配。

```kdl
// 在niri启动时以0.5不透明度显示层shell表面，但之后不显示。
layer-rule {
    match at-startup=true

    opacity 0.5
}
```

### 动态属性

这些属性持续应用于打开的layer-shell表面。

#### `block-out-from`

您可以从xdg-desktop-portal屏幕录制或所有屏幕捕获中屏蔽表面。
它们将被纯黑色矩形替换。

这对于通知很有用。

相同的注意事项和说明适用于[`block-out-from`窗口规则](./Configuration:-Window-Rules#block-out-from)，所以请查看那里的文档。

![显示通知正常可见，但在OBS中被屏蔽的截图。](./img/layer-block-out-from-screencast.png)

```kdl
// 从屏幕录制中屏蔽mako通知。
layer-rule {
    match namespace="^notifications$"

    block-out-from "screencast"
}
```

#### `opacity`

设置表面的不透明度。
`0.0`是完全透明，`1.0`是完全不透明。
这是在表面自身不透明度之上应用的，所以半透明表面会变得更加透明。

不透明度单独应用于layer-shell表面的每个子元素，所以子表面和弹出菜单将在其后显示窗口内容。

```kdl
// 使fuzzel半透明。
layer-rule {
    match namespace="^launcher$"

    opacity 0.95
}
```

#### `shadow`

<sup>自：25.02</sup>

覆盖表面的阴影选项。

这些规则具有与布局部分中的普通[`shadow`配置](./Configuration:-Layout#shadow)相同的选项，所以请查看那里的文档。

与窗口阴影不同，层表面阴影始终需要通过层规则启用。
也就是说，在布局配置部分中启用阴影不会自动为层表面启用它们。

> \[!NOTE]
> 层表面无法告诉niri它们的*视觉几何*。
> 例如，如果层表面包含一些不可见的边距（如mako），niri无法知道这一点，并将在整个表面（包括不可见边距）后面绘制阴影。
>
> 所以要使用niri阴影，您需要配置layer-shell客户端以移除它们自己的边距或阴影。

```kdl
// 为fuzzel添加阴影。
layer-rule {
    match namespace="^launcher$"

    shadow {
        on
    }

    // Fuzzel默认为10 px圆角。
    geometry-corner-radius 10
}
```

#### `geometry-corner-radius`

<sup>自：25.02</sup>

设置表面的圆角半径。

此设置仅影响阴影——它将将其圆角调整为匹配几何圆角半径。

```kdl
layer-rule {
    match namespace="^launcher$"

    geometry-corner-radius 12
}
```

#### `place-within-backdrop`

<sup>自：25.05</sup>

设置为`true`以将表面放置到在[概览](./Overview)和工作区之间可见的背景中。

这只适用于*背景*层表面，忽略独占区域（壁纸工具典型）。
背景中的层将忽略所有输入。

```kdl
// 将swaybg放入概览背景中。
layer-rule {
    match namespace="^wallpaper$"

    place-within-backdrop true
}
```

#### `baba-is-float`

<sup>自：25.05</sup>

使您的层表面上下FLOAT。

这是[2025年愚人节功能](./Configuration:-Window-Rules#baba-is-float)的自然扩展。

```kdl
// 使fuzzel FLOAT。
layer-rule {
    match namespace="^launcher$"

    baba-is-float true
}
```