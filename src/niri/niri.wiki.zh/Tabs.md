### 概述

<sup>自：25.02</sup>

您可以将列切换为以标签页形式显示窗口，而不是垂直平铺。
列中的所有标签页具有相同的窗口大小，因此这对于获得更多垂直空间很有用。

![左侧带有标签指示器的终端。](https://github.com/user-attachments/assets/0e94ac0d-796d-4f85-a264-c105ef41c13f)

使用此绑定在正常和标签页显示之间切换列：

```kdl
binds {
   Mod+W { toggle-column-tabbed-display; }
}
```

所有其他绑定保持不变：使用`focus-window-down/up`切换标签页，使用`consume-window-into-column`/`expel-window-from-column`添加或删除窗口，等等。

与常规列不同，标签页列可以包含多个窗口进行全屏显示。

### 标签指示器

标签页列在侧面显示标签指示器。
您可以单击指示器切换标签页。

请参见布局部分中的[`tab-indicator`部分](./Configuration:-Layout#tab-indicator)进行配置。

默认情况下，指示器在列"外部"绘制，因此它可以覆盖其他窗口或超出屏幕。
`place-within-column`标志将指示器放在列"内部"，调整窗口大小以为其腾出空间。
这对于较厚的标签指示器或当您有非常小的间隙时特别有用。

| 默认 | `place-within-column` |
| --- | --- |
| ![显示4个窗口的截图，中间列被聚焦。标签指示器溢出到左列](https://github.com/user-attachments/assets/c2f51f50-3d87-403a-8beb-cbbe5ec5c880) | ![显示4个窗口的截图，中间列被聚焦。标签指示器包含在其各自列内](https://github.com/user-attachments/assets/f1797cd0-d518-4be6-95b4-3540523c4370) |