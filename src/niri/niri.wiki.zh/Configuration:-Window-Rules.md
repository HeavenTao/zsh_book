### 概述

窗口规则允许您调整单个窗口的行为。
它们具有`match`和`exclude`指令，控制规则应适用于哪些窗口，以及您可以设置的多个属性。

窗口规则按配置文件中出现的顺序处理。
这意味着您可以先放置更通用的规则，然后在后面为特定窗口覆盖它们。
例如：

```kdl
// 为所有窗口设置open-maximized为true。
window-rule {
    open-maximized true
}

// 然后，对于Alacritty，将open-maximized设置回false。
window-rule {
    match app-id="Alacritty"
    open-maximized false
}
```

> \[!TIP]
> 一般来说，您无法在后续规则中"取消设置"属性，只能将其设置为不同值。
> 使用`exclude`指令以避免对特定窗口应用规则。

以下是窗口规则可能具有的所有匹配器和属性：

```kdl
window-rule {
    match title="Firefox"
    match app-id="Alacritty"
    match is-active=true
    match is-focused=false
    match is-active-in-column=true
    match is-floating=true
    match is-window-cast-target=true
    match is-urgent=true
    match at-startup=true

    // 窗口打开时一次性应用的属性。
    default-column-width { proportion 0.75; }
    default-window-height { fixed 500; }
    open-on-output "Some Company CoolMonitor 1234"
    open-on-workspace "chat"
    open-maximized true
    open-maximized-to-edges true
    open-fullscreen true
    open-floating true
    open-focused false

    // 持续应用的属性。
    draw-border-with-background false
    opacity 0.5
    block-out-from "screencast"
    // block-out-from "screen-capture"
    variable-refresh-rate true
    default-column-display "tabbed"
    default-floating-position x=100 y=200 relative-to="bottom-left"
    scroll-factor 0.75

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
        // 与focus-ring相同。
    }

    shadow {
        // on
        off
        softness 40
        spread 5
        offset x=0 y=5
        draw-behind-window true
        color "#00000064"
        // inactive-color "#00000064"
    }

    tab-indicator {
        active-color "red"
        inactive-color "gray"
        urgent-color "blue"
        // active-gradient from="#80c8ff" to="#bbddff" angle=45
        // inactive-gradient from="#505050" to="#808080" angle=45 relative-to="workspace-view"
        // urgent-gradient from="#800" to="#a33" angle=45
    }

    geometry-corner-radius 12
    clip-to-geometry true
    tiled-state true
    baba-is-float true

    min-width 100
    max-width 200
    min-height 300
    max-height 300
}
```

### 窗口匹配

每个窗口规则可以有多个`match`和`exclude`指令。
为了使规则应用，窗口需要匹配*任何*`match`指令，并且*不匹配*任何`exclude`指令。

```kdl
window-rule {
    // 匹配所有Telegram窗口...
    match app-id=r#"^org\.telegram\.desktop$"#

    // ...但媒体查看器窗口除外。
    exclude title="^Media viewer$"

    // 要应用的属性。
    open-on-output "HDMI-A-1"
}
```

匹配和排除指令具有相同的语法。
一个指令中可以有多个*匹配器*，窗口需要匹配所有匹配器才能使指令应用。

```kdl
window-rule {
    // 匹配标题中包含Gmail的Firefox窗口。
    match app-id="firefox" title="Gmail"
}

window-rule {
    // 匹配Firefox，但仅当它处于活动状态时...
    match app-id="firefox" is-active=true

    // ...或匹配Telegram...
    match app-id=r#"^org\.telegram\.desktop$"#

    // ...但不匹配Telegram媒体查看器。
    // 如果您在Firefox中打开一个名为"Media viewer"的标签页，
    // 它不会被排除，因为它不匹配此排除指令的app-id。
    exclude app-id=r#"^org\.telegram\.desktop$"# title="Media viewer"
}
```

让我们更详细地看看匹配器。

#### `title`和`app-id`

这些是正则表达式，应分别在窗口标题和应用ID中的任何地方匹配。
您可以在这里阅读支持的正则表达式语法：[here](https://docs.rs/regex/latest/regex/#syntax)。

```kdl
// 匹配标题中包含"Mozilla Firefox"的窗口，
// 或ID中包含"Alacritty"的应用窗口。
window-rule {
    match title="Mozilla Firefox"
    match app-id="Alacritty"
}
```

原始KDL字符串有助于编写正则表达式：

```kdl
window-rule {
    exclude app-id=r#"^org\.keepassxc\.KeePassXC$"#
}
```

您可以通过运行`niri msg pick-window`并点击相关窗口来找到窗口的标题和应用ID。

> \[!TIP]
> 查找窗口标题和应用ID的另一种方法是配置[Waybar](https://github.com/Alexays/Waybar)中的`wlr/taskbar`模块以在工具提示中包含它们：
>
> ```json
> "wlr/taskbar": {
>     "tooltip-format": "{title} | {app_id}",
> }
> ```

#### `is-active`

可以是`true`或`false`。
匹配活动窗口（具有活动边框/焦点环颜色的相同窗口）。

聚焦显示器上的每个工作区将有一个活动窗口。
这意味着您通常会有多个活动窗口（每个工作区一个），当您在工作区之间切换时，您可以看到两个活动窗口。

```kdl
window-rule {
    match is-active=true
}
```

#### `is-focused`

可以是`true`或`false`。
匹配具有键盘焦点的窗口。

与`is-active`相反，只能有一个焦点窗口。
此外，当打开layer-shell应用程序启动器或弹出菜单时，键盘焦点转到layer-shell。
当layer-shell具有键盘焦点时，窗口不匹配此规则。

```kdl
window-rule {
    match is-focused=true
}
```

#### `is-active-in-column`

<sup>自：0.1.6</sup>

可以是`true`或`false`。
匹配其列中"活动"的窗口。

与`is-active`相反，每个列中始终有一个`is-active-in-column`窗口。
它是列中最后聚焦的窗口，即如果聚焦此列，将获得焦点的窗口。

<sup>自：25.01</sup> 此规则在初始窗口打开期间将匹配`true`。

```kdl
window-rule {
    match is-active-in-column=true
}
```

#### `is-floating`

<sup>自：25.01</sup>

可以是`true`或`false`。
匹配浮动窗口。

> \[!NOTE]
> 此匹配器仅在窗口已打开后应用。
> 这意味着您无法使用它来更改窗口打开属性，如`default-window-height`或`open-on-workspace`。

```kdl
window-rule {
    match is-floating=true
}
```

#### `is-window-cast-target`

<sup>自：25.02</sup>

可以是`true`或`false`。
对于正在进行的窗口屏幕录制目标窗口，匹配`true`。

> \[!NOTE]
> 这仅匹配单个窗口屏幕录制。
> 例如，它不会匹配在显示器屏幕录制中可见的窗口。

```kdl
// 用红色指示屏幕录制窗口。
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

![显示只有is-window-cast-target=true窗口接收特殊边框颜色的截图](https://github.com/user-attachments/assets/375b381e-3a87-4e94-8676-44404971d893)

#### `is-urgent`

<sup>自：25.05</sup>

可以是`true`或`false`。
匹配请求用户注意的窗口。

```kdl
window-rule {
    match is-urgent=true
}
```

#### `at-startup`

<sup>自：0.1.6</sup>

可以是`true`或`false`。
匹配启动niri后的前60秒。

这对于您可能只想在启动niri后立即应用的属性（如`open-on-output`）很有用。

```kdl
// 在niri启动时在HDMI-A-1显示器上打开窗口，但之后不打开。
window-rule {
    match at-startup=true
    open-on-output "HDMI-A-1"
}
```

### 窗口打开属性

这些属性在窗口首次打开时一次性应用。

准确地说，它们在niri向窗口发送初始配置请求时应用。

#### `default-column-width`

设置新窗口的默认宽度。

尽管名称中有"column"，这也适用于浮动窗口。

```kdl
// 为Blender和GIMP在打开时提供一些保证的宽度。
window-rule {
    match app-id="^blender$"

    // GIMP应用ID包含版本号，如"gimp-2.99"，
    // 所以我们只匹配开头（用^）而不匹配结尾。
    match app-id="^gimp"

    default-column-width { fixed 1200; }
}
```

#### `default-window-height`

<sup>自：25.01</sup>

设置新窗口的默认高度。

```kdl
// 以480×270大小浮动打开Firefox画中画窗口。
window-rule {
    match app-id="firefox$" title="^Picture-in-Picture$"

    open-floating true
    default-column-width { fixed 480; }
    default-window-height { fixed 270; }
}
```

#### `open-on-output`

使窗口在特定输出上打开。

如果不存在这样的输出，窗口将像往常一样在当前聚焦的输出上打开。

如果窗口在当前未聚焦的输出上打开，窗口将不会自动聚焦。

```kdl
// 在特定显示器上打开Firefox和Telegram（但不包括其媒体查看器）
window-rule {
    match app-id="firefox$"
    match app-id=r#"^org\.telegram\.desktop$"#
    exclude app-id=r#"^org\.telegram\.desktop$"# title="^Media viewer$"

    open-on-output "HDMI-A-1"
    // 或：
    // open-on-output "Some Company CoolMonitor 1234"
}
```

<sup>自：0.1.9</sup> `open-on-output`现在可以使用显示器制造商、型号和序列号。
之前，它只能使用连接器名称。

#### `open-on-workspace`

<sup>自：0.1.6</sup>

使窗口在特定[命名工作区](./Configuration:-Named-Workspaces)上打开。

如果不存在这样的工作区，窗口将像往常一样在当前聚焦的工作区上打开。

如果窗口在当前未聚焦的输出上打开，窗口将不会自动聚焦。

```kdl
// 在"chat"工作区上打开Fractal。
window-rule {
    match app-id=r#"^org\.gnome\.Fractal$"#

    open-on-workspace "chat"
}
```

#### `open-maximized`

使窗口作为最大化列打开。

```kdl
// 默认最大化Firefox。
window-rule {
    match app-id="firefox$"

    open-maximized true
}
```

#### `open-maximized-to-edges`

<sup>自：下一个版本</sup>

使窗口[最大化到边缘](./Fullscreen-and-Maximize)打开。

```kdl
window-rule {
    open-maximized-to-edges true
}
```

您也可以将其设置为`false`以*防止*窗口最大化到边缘打开。

```kdl
window-rule {
    open-maximized-to-edges false
}
```

#### `open-fullscreen`

使窗口[全屏](./Fullscreen-and-Maximize)打开。

```kdl
window-rule {
    open-fullscreen true
}
```

您也可以将其设置为`false`以*防止*窗口全屏打开。

```kdl
// 使Telegram媒体查看器以窗口模式打开。
window-rule {
    match app-id=r#"^org\.telegram\.desktop$"# title="^Media viewer$"

    open-fullscreen false
}
```

#### `open-floating`

<sup>自：25.01</sup>

使窗口在浮动布局中打开。

```kdl
// 将Firefox画中画窗口作为浮动窗口打开。
window-rule {
    match app-id="firefox$" title="^Picture-in-Picture$"

    open-floating true
}
```

您也可以将其设置为`false`以*防止*窗口在浮动布局中打开。

```kdl
// 在平铺布局中打开所有窗口，覆盖任何自动浮动逻辑。
window-rule {
    open-floating false
}
```

#### `open-focused`

<sup>自：25.01</sup>

将其设置为`false`以防止此窗口在打开时自动聚焦。

```kdl
// 不给GIMP启动屏幕焦点。
window-rule {
    match app-id="^gimp" title="^GIMP Startup$"

    open-focused false
}
```

您也可以将其设置为`true`以聚焦窗口，即使通常它不会自动聚焦。

```kdl
// 始终聚焦KeePassXC-Browser解锁对话框。
//
// 此对话框作为KeePassXC窗口的子窗口打开，而不是浏览器的子窗口，
// 所以默认情况下它不会自动聚焦。
window-rule {
    match app-id=r#"^org\.keepassxc\.KeePassXC$"# title="^Unlock Database - KeePassXC$"

    open-focused true
}
```

### 动态属性

这些属性持续应用于打开的窗口。

#### `block-out-from`

您可以将窗口从xdg-desktop-portal屏幕录制中屏蔽。
它们将被纯黑色矩形替换。

这对于密码管理器或消息窗口等很有用。
对于layer-shell通知弹出窗口等，您可以使用[`block-out-from`层规则](./Configuration:-Layer-Rules#block-out-from)。

![显示窗口正常可见但在OBS中被屏蔽的截图](./img/block-out-from-screencast.png)

要预览和设置此规则，请检查配置的调试部分中的`preview-render`选项。

> \[!CAUTION]
> 窗口**不会**从第三方屏幕截图工具中屏蔽。
> 如果您在屏幕录制时打开一些带有预览的屏幕截图工具，被屏蔽的窗口**将在屏幕录制中可见**。

内置屏幕截图UI不受此问题影响。
如果您在屏幕录制时打开屏幕截图UI，您将能够在看到所有窗口正常显示的同时选择要截图的区域，但在屏幕录制中选择UI将显示被屏蔽的窗口。

```kdl
// 从屏幕录制中屏蔽密码管理器。
window-rule {
    match app-id=r#"^org\.keepassxc\.KeePassXC$"#
    match app-id=r#"^org\.gnome\.World\.Secrets$"#

    block-out-from "screencast"
}
```

或者，您可以将窗口从*所有*屏幕捕获中屏蔽，包括第三方屏幕截图工具。
这样可以避免在打开第三方屏幕截图预览时意外显示窗口。

此设置仍允许您使用交互式内置屏幕截图UI，但它将阻止窗口进行完全自动的屏幕截图操作，如`screenshot-screen`和`screenshot-window`。
推理是，在交互式选择中，您可以确保避免截取敏感内容。

```kdl
window-rule {
    block-out-from "screen-capture"
}
```

> \[!WARNING]
> 小心基于动态变化的窗口标题屏蔽窗口。
>
> 例如，您可能尝试屏蔽特定的Firefox标签页，如下所示：
>
> ```kdl
> window-rule {
>     // 不完全工作！尝试屏蔽Gmail标签页。
>     match app-id="firefox$" title="- Gmail "
>
>     block-out-from "screencast"
> }
> ```
>
> 它会工作，但当从敏感标签页切换到常规标签页时，敏感标签页的内容**将在屏幕录制中出现**一瞬间。
> 这是因为窗口标题（和应用ID）在Wayland协议中没有双重缓冲，所以它们不与特定窗口内容相关联。
> Firefox没有可靠的方法来同步可见地显示不同标签页和更改窗口标题。

#### `opacity`

设置窗口的不透明度。
`0.0`是完全透明，`1.0`是完全不透明。
这是在窗口自身不透明度之上应用的，所以半透明窗口会变得更加透明。

不透明度单独应用于窗口的每个表面，所以子表面和弹出菜单将在其后显示窗口内容。

![显示Adwaita Demo具有半透明弹出菜单的截图](./img/opacity-popup.png)

此外，带有背景的焦点环和边框将透过半透明窗口显示（参见`prefer-no-csd`和下面的`draw-border-with-background`窗口规则）。

可以使用[`toggle-window-rule-opacity`](./Configuration:-Key-Bindings#toggle-window-rule-opacity)操作切换窗口的不透明度。

```kdl
// 使非活动窗口半透明。
window-rule {
    match is-active=false

    opacity 0.95
}
```

#### `variable-refresh-rate`

<sup>自：0.1.9</sup>

如果设置为true，每当此窗口显示在具有按需VRR的输出上时，它将在该输出上启用VRR。

```kdl
// 配置具有按需VRR的输出。
output "HDMI-A-1" {
    variable-refresh-rate on-demand=true
}

// 当mpv显示在输出上时启用按需VRR。
window-rule {
    match app-id="^mpv$"

    variable-refresh-rate true
}
```

#### `default-column-display`

<sup>自：25.02</sup>

设置从此窗口创建的列的默认显示模式。

这在窗口进入其自己的列时使用。
例如：

*   打开新窗口。
*   将窗口排出到自己的列中。
*   将窗口从浮动布局移动到平铺布局。

```kdl
// 使Evince窗口作为标签列打开。
window-rule {
    match app-id="^evince$"

    default-column-display "tabbed"
}
```

#### `default-floating-position`

<sup>自：25.01</sup>

设置此窗口在其上打开或移动到浮动布局时的初始位置。

之后，窗口将记住其最后的浮动位置。

默认情况下，新的浮动窗口在屏幕中心打开，平铺布局中的窗口在接近其视觉屏幕位置处打开。

该位置使用相对于工作区域的逻辑坐标。
默认情况下，它们相对于工作区域的左上角，但您可以通过将`relative-to`设置为以下值之一来更改此设置：`top-left`、`top-right`、`bottom-left`、`bottom-right`、`top`、`bottom`、`left`或`right`。

例如，如果您的顶部有栏，则`x=0 y=0`将使窗口的左上角直接位于栏下方。
如果改为写入`x=0 y=0 relative-to="top-right"`，则窗口的右上角将与工作区的右上角对齐，也直接位于栏下方。
当仅指定一个侧面（例如top）时，窗口将与该侧面的中心对齐。

坐标根据`relative-to`改变方向。
例如，默认情况下（左上角），`x=100 y=200`将使窗口在左上角右侧100像素和下方200像素处。
如果您使用`x=100 y=200 relative-to="bottom-left"`，它将使窗口在左下角右侧100像素和*上方*200像素处。

```kdl
// 以小间隙在屏幕左下角打开Firefox画中画窗口。
window-rule {
    match app-id="firefox$" title="^Picture-in-Picture$"

    default-floating-position x=32 y=32 relative-to="bottom-left"
}
```

您可以使用单侧面`relative-to`获得下拉效果。

```kdl
// 示例：下拉终端。
window-rule {
    // 通过"dropdown"应用ID匹配。
    // 您需要在运行终端时设置此应用ID，例如：
    // spawn "alacritty" "--class" "dropdown"
    match app-id="^dropdown$"

    // 作为浮动窗口打开。
    open-floating true
    // 锚定到屏幕顶部边缘。
    default-floating-position x=0 y=0 relative-to="top"
    // 半个屏幕高。
    default-window-height { proportion 0.5; }
    // 80%屏幕宽。
    default-column-width { proportion 0.8; }
}
```

#### `scroll-factor`

<sup>自：25.02</sup>

为发送到窗口的所有滚动事件设置滚动系数。

这将与您在[输入部分](./Configuration:-Input#pointing-devices)中为输入设备设置的滚动系数相乘。

```kdl
// 使Firefox中的滚动稍微慢一些。
window-rule {
    match app-id="firefox$"

    scroll-factor 0.75
}
```

#### `draw-border-with-background`

覆盖边框和焦点环是否使用背景绘制。

将此设置为`true`以绘制它们为纯色矩形，即使对于同意省略其客户端装饰的窗口。
将此设置为`false`以绘制它们为窗口周围的边框，即使对于使用客户端装饰的窗口。

此属性对于不支持xdg-decoration协议的矩形窗口很有用。

| 带背景 | 不带背景 |
| ------------------------------------------------ | --------------------------------------------------- |
| [显示draw-border-with-background设置为true的窗口的截图](./img/simple-egl-border-with-background.png) | [显示draw-border-with-background设置为false的窗口的截图](./img/simple-egl-border-without-background.png) |

```kdl
window-rule {
    draw-border-with-background false
}
```

#### `focus-ring`和`border`

<sup>自：0.1.6</sup>

覆盖窗口的焦点环和边框选项。

这些规则具有与普通[布局部分中的`focus-ring`和`border`配置](./Configuration:-Layout#focus-ring-and-border)相同的选项，所以请查看那里的文档。

但是，除了`off`禁用边框/焦点环外，此窗口规则还有一个`on`标志，即使否则被禁用，也为窗口启用边框/焦点环。
如果同时设置了`on`和`off`标志，`on`标志优先。

```kdl
window-rule {
    focus-ring {
        off
        width 2
    }
}

window-rule {
    border {
        on
        width 8
    }
}
```

#### `shadow`

<sup>自：25.02</sup>

覆盖窗口的阴影选项。

此规则具有与普通[布局部分中的`shadow`配置](./Configuration:-Layout#shadow)相同的选项，所以请查看那里的文档。

但是，除了`on`启用阴影外，此窗口规则还有一个`off`标志，即使否则启用，也为窗口禁用阴影。
如果同时设置了`on`和`off`标志，`on`标志优先。

```kdl
// 为浮动窗口开启阴影。
window-rule {
    match is-floating=true

    shadow {
        on
    }
}
```

#### `tab-indicator`

<sup>自：25.02</sup>

覆盖窗口的标签指示器选项。

此规则中的选项匹配普通[布局部分中的`tab-indicator`配置](./Configuration:-Layout#tab-indicator)的相同选项，所以请查看那里的文档。

```kdl
// 使KeePassXC标签具有深红色非活动颜色。
window-rule {
    match app-id=r#"^org\.keepassxc\.KeePassXC$"#

    tab-indicator {
        inactive-color "darkred"
    }
}
```

#### `geometry-corner-radius`

<sup>自：0.1.6</sup>

设置窗口的圆角半径。

仅此设置将仅影响边框和焦点环——它们将将其圆角调整为匹配几何圆角半径。
如果您想强制将窗口本身的圆角变圆，请除了此设置外还设置[`clip-to-geometry true`](#clip-to-geometry)。

```kdl
window-rule {
    geometry-corner-radius 12
}
```

半径以逻辑像素设置，并控制窗口本身的半径，即边框的内半径：

![显示具有每个角落圆角的窗口的截图](./img/geometry-corner-radius.png)

您可以设置四个半径，每个角落一个。
顺序与CSS相同：左上角、右上角、右下角、左下角。

```kdl
window-rule {
    geometry-corner-radius 8 8 0 0
}
```

这样，您可以匹配具有方形底部角落的GTK 3应用程序：

![显示只有顶部圆角的窗口的截图](./img/different-corner-radius.png)

#### `clip-to-geometry`

<sup>自：0.1.6</sup>

将窗口裁剪到其视觉几何。

这将切掉任何客户端窗口阴影，还根据`geometry-corner-radius`对窗口圆角进行处理。

![显示具有圆角并裁剪到视觉几何的窗口的截图](./img/clip-to-geometry.png)

```kdl
window-rule {
    clip-to-geometry true
}
```

启用边框，设置[`geometry-corner-radius`](#geometry-corner-radius)和`clip-to-geometry`，您就有了经典设置：

![显示具有圆角和边框的窗口的截图](./img/border-radius-clip.png)

```kdl
prefer-no-csd

layout {
    focus-ring {
        off
    }

    border {
        width 2
    }
}

window-rule {
    geometry-corner-radius 12
    clip-to-geometry true
}
```

#### `tiled-state`

<sup>自：25.05</sup>

通知窗口它是平铺的。
通常，窗口会通过变为矩形并隐藏其客户端阴影来响应。
将大小调整为网格的窗口（例如[foot](https://codeberg.org/dnkl/foot)等终端）在平铺时通常会禁用此调整。

默认情况下，niri将与[`prefer-no-csd`](./Configuration:-Miscellaneous#prefer-no-csd)一起将平铺状态设置为`true`，以改善不支持服务器端装饰的应用程序的行为。
您可以使用此窗口规则覆盖此设置，例如获得使用CSD的矩形窗口。

```kdl
// 使平铺窗口矩形化同时使用CSD。
window-rule {
    match is-floating=false

    tiled-state true
}
```

#### `baba-is-float`

<sup>自：25.02</sup>

使您的窗口上下FLOAT。

这是2025年愚人节功能。

```kdl
window-rule {
    match is-floating=true

    baba-is-float true
}
```

<video controls src="https://github.com/user-attachments/assets/3f4cb1a4-40b2-4766-98b7-eec014c19509">

https://github.com/user-attachments/assets/3f4cb1a4-40b2-4766-98b7-eec014c19509

</video>

#### 大小覆盖

您可以在逻辑像素中修改窗口的最小和最大大小。

请记住，窗口本身始终对其大小有最终决定权。
这些值指示niri永远不要要求窗口小于您设置的最小值，或大于您设置的最大值。

> \[!NOTE]
> `max-height`仅在自动调整大小的窗口中应用，如果它等于`min-height`。
> 要么将其设置为等于`min-height`，要么在使用`set-window-height`打开后手动更改窗口高度。
>
> 这是niri窗口高度分布算法的限制。

```kdl
window-rule {
    min-width 100
    max-width 200
    min-height 300
    max-height 300
}
```

```kdl
// 修复服务器端装饰的OBS缺少最小宽度。
window-rule {
    match app-id=r#"^com\.obsproject\.Studio$"#

    min-width 876
}
```