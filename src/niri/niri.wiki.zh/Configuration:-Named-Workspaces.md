### 概述

<sup>自：0.1.6</sup>

您可以在配置的顶层声明命名工作区：

```kdl
workspace "browser"

workspace "chat" {
    open-on-output "Some Company CoolMonitor 1234"
}
```

与普通的动态工作区不同，命名工作区始终存在，即使它们没有窗口。
否则，它们的行为与任何其他工作区相同：您可以移动它们，将它们移动到不同的显示器上，等等。

像`focus-workspace`或`move-column-to-workspace`这样的操作可以通过名称引用工作区。
此外，您可以使用`open-on-workspace`窗口规则使窗口在特定的命名工作区上打开：

```kdl
// 声明一个名为"chat"的工作区，该工作区在"DP-2"输出上打开。
workspace "chat" {
    open-on-output "DP-2"
}

// 如果在niri启动时运行，将在"chat"工作区上打开Fractal。
window-rule {
    match at-startup=true app-id=r#"^org\.gnome\.Fractal$"#
    open-on-workspace "chat"
}
```

命名工作区最初按照配置文件中声明的顺序出现。
在niri运行时编辑配置时，新声明的命名工作区将出现在显示器的最顶部。

如果您从配置中删除某些命名工作区，该工作区将变为普通（未命名）工作区，如果上面没有窗口，它将被删除（像任何其他普通工作区一样）。
无法给已存在的工作区命名，但您可以简单地将想要的窗口移动到新的空命名工作区。

<sup>自：0.1.9</sup> `open-on-output`现在可以使用显示器制造商、型号和序列号。
之前，它只能使用连接器名称。

<sup>自：25.01</sup> 您可以使用`set-workspace-name`和`unset-workspace-name`操作动态更改工作区名称。

<sup>自：25.02</sup> 命名工作区在上面打开新窗口时不再更新/忘记其原始输出（未命名工作区将继续这样做）。
这意味着在更多情况下，命名工作区"粘"在其原始输出上，反映了它们更永久的性质。
明确将命名工作区移动到不同的显示器仍会更新其原始输出。

### 布局配置覆盖

<sup>自：下一个版本</sup>

您可以使用`layout {}`块为命名工作区自定义布局设置：

```kdl
workspace "aesthetic" {
    // 仅适用于此命名工作区的布局配置覆盖。
    layout {
        gaps 32

        struts {
            left 64
            right 64
            bottom 64
            top 64
        }

        border {
            on
            width 4
        }

        // ...任何其他设置。
    }
}
```

它接受与[顶层`layout {}`块](./Configuration:-Layout)相同的所有选项，除了：

*   `empty-workspace-above-first`：这是一个输出级别设置，在工作区上没有意义。
*   `insert-hint`：目前我们总是在输出级别绘制这些，所以不能按工作区自定义。

为了取消设置标志，请用`false`写入，例如：

```kdl
layout {
    // 全局启用。
    always-center-single-column
}

workspace "uncentered" {
    layout {
        // 在此工作区上取消设置。
        always-center-single-column false
    }
}
```