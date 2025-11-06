<sup>自：下一个版本</sup>

您可以在配置的顶层包含其他文件。

```kdl,must-fail
// 一些设置...

include "colors.kdl"

// 更多设置...
```

包含的文件具有与主配置文件相同的结构。
来自包含文件的设置将与主配置文件的设置合并。

包含的配置文件可以反过来包含更多文件。
所有包含的文件都会被监视更改，并且当其中任何一个更改时配置会实时重新加载。

包含只能在配置的顶层工作：

```kdl,must-fail
// 好的：顶层包含。
include "something.kdl"

layout {
    // 不允许：在其他部分内部包含。
    include "other.kdl"
}
```

### 位置性

包含是*位置性的*。
它们将覆盖在它们*之前*设置的选项。
来自包含文件的窗口规则将在`include`行的位置插入。例如：

```kdl
// colors.kdl
layout {
    border {
        active-color "green"
    }
}

overview {
    backdrop-color "green"
}
```

```kdl,must-fail
// config.kdl
layout {
    border {
        active-color "red"
    }
}

// 这会将边框颜色和背景颜色覆盖为绿色。
include "colors.kdl"

// 这会再次将概览背景颜色设置为红色。
overview {
    backdrop-color "red"
}
```

最终结果：

*   边框颜色是绿色（来自`colors.kdl`），
*   概览背景颜色是红色（它是在`colors.kdl`*之后*设置的）。

另一个例子：

```kdl
// rules.kdl
window-rule {
    match app-id="Alacritty"
    open-maximized false
}
```

```kdl,must-fail
// config.kdl
window-rule {
    open-maximized true
}

// 窗口规则在此位置插入。
include "rules.kdl"

window-rule {
    match app-id="firefox$"
    open-maximized true
}
```

这等同于以下配置文件：

```kdl
window-rule {
    open-maximized true
}

// 来自rules.kdl。
window-rule {
    match app-id="Alacritty"
    open-maximized false
}

window-rule {
    match app-id="firefox$"
    open-maximized true
}
```

### 合并

大多数配置部分在包含之间是合并的，这意味着您可以只设置一些属性，只有这些属性会改变。

```kdl
// colors.kdl
layout {
    // 不影响间距、边框宽度等。
    // 只改变写入的颜色。
    focus-ring {
        active-color "blue"
    }

    border {
        active-color "green"
    }
}
```

```kdl,must-fail
// config.kdl
include "colors.kdl"

layout {
    // 不设置边框和焦点环颜色，
    // 所以使用colors.kdl中的颜色。
    gaps 8

    border {
        width 8
    }
}
```

#### 多部分部分

像`window-rule`、`output`或`workspace`这样的多部分部分将按原样插入而不合并：

```kdl
// laptop.kdl
output "eDP-1" {
    // ...
}
```

```kdl,must-fail
// config.kdl
output "DP-2" {
    // ...
}

include "laptop.kdl"

// 最终结果：DP-2和eDP-1设置都有。
```

#### 绑定

`binds`将覆盖先前定义的冲突键：

```kdl
// binds.kdl
binds {
    Mod+T { spawn "alacritty"; }
}
```

```kdl,must-fail
// config.kdl
include "binds.kdl"

binds {
    // 覆盖binds.kdl中的Mod+T。
    Mod+T { spawn "foot"; }
}
```

#### 标志

大多数标志可以用`false`禁用：

```kdl
// csd.kdl

// 写"false"以明确禁用。
prefer-no-csd false
```

```kdl,must-fail
// config.kdl

// 在主配置中启用prefer-no-csd。
prefer-no-csd

// 包含csd.kdl将再次禁用它。
include "csd.kdl"
```

#### 非合并部分

一些内容代表组合结构的部分不合并。
示例是`struts`、`preset-column-widths`、`animations`中的单个子部分、`input`中的指向设备部分。

```kdl
// struts.kdl
layout {
    struts {
        left 64
        right 64
    }
}
```

```kdl,must-fail
// config.kdl
layout {
    struts {
        top 64
        bottom 64
    }
}

include "struts.kdl"

// Struts不合并。
// 最终结果只有左侧和右侧struts。
```

### 边框特殊情况

主配置和包含配置之间有一个特殊情况不同。

在包含配置中写入`layout { border {} }`什么都不做（因为没有属性被更改）。
然而，在主配置中写入相同内容将*启用*边框，即它等同于`layout { border { on; } }`。

所以，如果您想将布局配置从主配置移动到单独文件，请记住在边框部分添加`on`，例如：

```kdl
// separate.kdl
layout {
    border {
        // 添加此行：
        on

        width 4
        active-color "#ffc87f"
        inactive-color "#505050"
    }
}
```

此特殊情况的原因是历史上它是这样工作的：当我添加边框时，我们没有任何`on`标志，所以我让写入`border {}`部分启用边框，用显式的`off`来禁用它。
改变它不会太有问题，但默认配置总是有一个预填充的`layout { border { off; } }`部分，并带有注释说明注释掉`off`就足以启用边框。
现在许多人可能在他们的配置中嵌入了这部分默认配置，所以改变它的工作方式只会引起很多困惑。