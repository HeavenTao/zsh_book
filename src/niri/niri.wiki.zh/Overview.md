### 概述

<sup>自：25.05</sup>

概览是您的工作区和窗口的缩小视图。
它让您一目了然地查看正在发生的事情，进行导航，并拖拽窗口。

<video controls src="https://github.com/user-attachments/assets/379a5d1f-acdb-4c11-b36c-e85fd91f0995">

https://github.com/user-attachments/assets/379a5d1f-acdb-4c11-b36c-e85fd91f0995

</video>

通过`toggle-overview`绑定、左上角热角或使用触控板四指向上滑动打开它。
在概览中，所有键盘快捷键继续工作，而指向设备变得更简单：

*   鼠标：左键单击并拖拽窗口以移动它们，右键单击并拖拽以左右滚动工作区，滚动以切换工作区（无需按住Mod）。
*   触控板：与正常三指手势匹配的双指滚动。
*   触摸屏：单指滚动，或单指长按以移动窗口。

> \[!TIP]
> 概览需要在每个工作区下绘制背景。
> 因此，layer-shell表面的工作方式是：*背景*和*底部*层与工作区一起缩小，而*顶部*和*覆盖*层保持在概览之上。
>
> 将您的栏放在*顶部*层。

拖放将在概览中上下滚动工作区，并在按住一段时间时激活工作区。
结合热角，这使您能够仅使用鼠标在工作区之间进行拖放。

<video controls src="https://github.com/user-attachments/assets/5f09c5b7-ff40-462b-8b9c-f1b8073a2cbb">

https://github.com/user-attachments/assets/5f09c5b7-ff40-462b-8b9c-f1b8073a2cbb

</video>

您还可以将窗口拖放到现有工作区上方、下方或之间的新工作区。

<video controls src="https://github.com/user-attachments/assets/b76d5349-aa20-4889-ab90-0a51554c789d">

https://github.com/user-attachments/assets/b76d5349-aa20-4889-ab90-0a51554c789d

</video>

### 配置

查看`overview {}`部分的完整文档[这里](./Configuration:-Miscellaneous#overview)。

您可以像这样设置缩小级别：

```kdl
// 在概览中使工作区比正常小四倍。
overview {
    zoom 0.25
}
```

要更改工作区后面的颜色，请使用`backdrop-color`设置：

```kdl
// 使背景变亮。
overview {
    backdrop-color "#777777"
}
```

您还可以禁用热角：

```kdl
// 禁用热角。
gestures {
    hot-corners {
        off
    }
}
```

### 背景自定义

除了如上所述设置自定义背景颜色外，您还可以通过[layer rule](./Configuration:-Layer-Rules#place-within-backdrop)将layer-shell壁纸放入背景中，例如：

```kdl
// 将swaybg放入概览背景中。
layer-rule {
    match namespace="^wallpaper$"
    place-within-backdrop true
}
```

这仅适用于忽略独占区域的*背景*层表面（壁纸工具典型）。

您可以运行两个不同的壁纸工具（如swaybg和swww），一个用于背景，一个用于正常工作区背景。
这样您可以将背景工具设置为壁纸的模糊版本以获得不错的效果。

如果您不喜欢壁纸与工作区一起移动，还可以将其与透明背景颜色结合使用：

```kdl
// 使壁纸静止，而不是与工作区一起移动。
layer-rule {
    // 这是针对swaybg的；为其他壁纸工具更改。
    // 通过运行niri msg layers找到正确的命名空间。
    match namespace="^wallpaper$"
    place-within-backdrop true
}

// 设置透明工作区背景颜色。
layout {
    background-color "transparent"
}

// 可选地，在概览中禁用工作区阴影。
overview {
    workspace-shadow {
        off
    }
}
```