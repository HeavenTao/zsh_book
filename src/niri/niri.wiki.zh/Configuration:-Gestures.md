### 概述

<sup>自：25.02</sup>

`gestures`配置部分包含手势设置。
有关所有niri手势的概述，请参见[Gestures](./Gestures) wiki页面。

这是可用设置及其默认值的快速浏览。

```kdl
gestures {
    dnd-edge-view-scroll {
        trigger-width 30
        delay-ms 100
        max-speed 1500
    }

    dnd-edge-workspace-switch {
        trigger-height 50
        delay-ms 100
        max-speed 1500
    }

    hot-corners {
        // off
        top-left
        // top-right
        // bottom-left
        // bottom-right
    }
}
```

### `dnd-edge-view-scroll`

在拖放（DnD）期间将鼠标光标移动到显示器边缘时滚动平铺视图。
也适用于触摸屏。

这适用于常规拖放（例如从文件管理器拖拽文件），以及在针对平铺布局时窗口交互式移动。

选项包括：

*   `trigger-width`：触发滚动的显示器边缘附近的区域大小，以逻辑像素为单位。
*   `delay-ms`：滚动开始前的延迟，以毫秒为单位。
    避免在跨显示器拖拽物品时意外滚动。
*   `max-speed`：最大滚动速度，以每秒逻辑像素为单位。
    当您将鼠标光标从`trigger-width`移动到显示器边缘时，滚动速度线性增加。

```kdl
gestures {
    // 增加触发区域和最大速度。
    dnd-edge-view-scroll {
        trigger-width 100
        max-speed 3000
    }
}
```

### `dnd-edge-workspace-switch`

<sup>自：25.05</sup>

在概览中拖放（DnD）期间将鼠标光标移动到显示器边缘时上下滚动工作区。
也适用于触摸屏。

选项包括：

*   `trigger-height`：触发滚动的显示器边缘附近的区域大小，以逻辑像素为单位。
*   `delay-ms`：滚动开始前的延迟，以毫秒为单位。
    避免在跨显示器拖拽物品时意外滚动。
*   `max-speed`：最大滚动速度；1500对应每秒一个屏幕高度。
    当您将鼠标光标从`trigger-width`移动到显示器边缘时，滚动速度线性增加。

```kdl
gestures {
    // 增加触发区域和最大速度。
    dnd-edge-workspace-switch {
        trigger-height 100
        max-speed 3000
    }
}
```

### `hot-corners`

<sup>自：25.05</sup>

将鼠标放在显示器的左上角以切换概览。
在拖放物品时也有效。

`off`禁用热角。

```kdl
// 禁用热角。
gestures {
    hot-corners {
        off
    }
}
```

<sup>自：下一个版本</sup> 您可以通过名称选择特定的热角：`top-left`、`top-right`、`bottom-left`、`bottom-right`。
如果没有明确设置角落，则默认激活左上角。

```kdl
// 启用右上角和右下角热角。
gestures {
    hot-corners {
        top-right
        bottom-right
    }
}
```

您还可以在[输出配置中](./Configuration:-Outputs#hot-corners)自定义每个输出的热角。