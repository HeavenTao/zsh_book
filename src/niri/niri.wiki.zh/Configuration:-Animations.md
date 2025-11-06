### 概述

Niri有几个动画，您可以以相同方式配置它们。
此外，您可以一次性禁用或减慢所有动画。

这是可用动画及其默认值的快速浏览。

```kdl
animations {
    // 取消注释以关闭所有动画。
    // 您也可以将"off"放入每个单独的动画中以禁用它。
    // off

    // 按此因子减慢所有动画。低于1的值会加速它们。
    // slowdown 3.0

    // 单独动画。

    workspace-switch {
        spring damping-ratio=1.0 stiffness=1000 epsilon=0.0001
    }

    window-open {
        duration-ms 150
        curve "ease-out-expo"
    }

    window-close {
        duration-ms 150
        curve "ease-out-quad"
    }

    horizontal-view-movement {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }

    window-movement {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }

    window-resize {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }

    config-notification-open-close {
        spring damping-ratio=0.6 stiffness=1000 epsilon=0.001
    }

    exit-confirmation-open-close {
        spring damping-ratio=0.6 stiffness=500 epsilon=0.01
    }

    screenshot-ui-open {
        duration-ms 200
        curve "ease-out-quad"
    }

    overview-open-close {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }
}
```

### 动画类型

有两种动画类型：缓动和弹簧。
每种动画可以是缓动或弹簧。

#### 缓动

这是一种相对常见的动画类型，使用插值曲线在设定的持续时间内改变值。

要使用此动画，请设置以下参数：

*   `duration-ms`：动画的持续时间（以毫秒为单位）。
*   `curve`：要使用的缓动曲线。

```kdl
animations {
    window-open {
        duration-ms 150
        curve "ease-out-expo"
    }
}
```

目前，niri仅支持五种曲线。
您可以在[easings.net](https://easings.net/)等页面上感受它们。

*   `ease-out-quad` <sup>自：0.1.5</sup>
*   `ease-out-cubic`
*   `ease-out-expo`
*   `linear` <sup>自：0.1.6</sup>
*   `cubic-bezier` <sup>自：25.08</sup>
    自定义[cubic Bézier曲线](https://www.w3.org/TR/css-easing-1/#cubic-bezier-easing-functions)。您需要设置4个定义曲线控制点的数字，例如：
    ```kdl
    animations {
        window-open {
            // 与CSS cubic-bezier(0.05, 0.7, 0.1, 1)相同
            curve "cubic-bezier" 0.05 0.7 0.1 1
        }
    }
    ```
    您可以在[easings.co](https://easings.co?curve=0.05,0.7,0.1,1)等页面上调整cubic-bezier参数。

#### 弹簧

弹簧动画使用物理弹簧模型来动画化值。
它们在触控板手势时感觉特别好，因为它们考虑了您释放滑动时手指的速度。
如果喜欢，弹簧也可以在结束时以正确的参数振荡/弹跳，但它们不必这样做（默认情况下它们大多不会）。

由于弹簧使用物理模型，动画参数不太明显，通常应通过试错进行调整。
特别是，您不能直接设置持续时间。
您可以使用[Elastic](https://flathub.org/apps/app.drey.Elastic)应用程序来帮助可视化弹簧参数如何改变动画。

弹簧动画配置如下，有三个必需参数：

```kdl
animations {
    workspace-switch {
        spring damping-ratio=1.0 stiffness=1000 epsilon=0.0001
    }
}
```

`damping-ratio`从0.1到10.0，具有以下特性：

*   低于1.0：欠阻尼弹簧，最终会振荡。
*   高于1.0：过阻尼弹簧，不会振荡。
*   1.0：临界阻尼弹簧，在最短时间内无振荡地停止。

然而，即使阻尼比=1.0，弹簧动画在从触控板滑动"发射"时也可能振荡。

> \[!WARNING]
> 过阻尼弹簧目前有一些数值稳定性问题，可能导致图形故障。
> 因此，不建议将`damping-ratio`设置为高于`1.0`。

较低的`stiffness`将导致更慢的动画，更容易振荡。

如果动画在结束时"跳跃"，请将`epsilon`设置为较低值。

> \[!TIP]
> 弹簧*质量*（您可以在Elastic中看到）硬编码为1.0，无法更改。
> 而是按比例改变`stiffness`。
> 例如，将质量增加2倍与将刚度减少2倍相同。

### 动画

现在让我们详细了解一下您可以配置的动画。

#### `workspace-switch`

上下切换工作区的动画，包括垂直触控板手势后（建议使用弹簧）。

```kdl
animations {
    workspace-switch {
        spring damping-ratio=1.0 stiffness=1000 epsilon=0.0001
    }
}
```

#### `window-open`

窗口打开动画。

这个默认使用缓动类型。

```kdl
animations {
    window-open {
        duration-ms 150
        curve "ease-out-expo"
    }
}
```

##### `custom-shader`

<sup>自：0.1.6</sup>

您可以为窗口在打开动画期间编写自定义着色器。

请参阅[此示例着色器](./examples/open_custom_shader.frag)以获得完整的文档和几种可试验的动画。

如果自定义着色器编译失败，niri将打印警告并回退到默认，或之前成功编译的着色器。
当作为systemd服务运行niri时，您可以在日志中看到警告：`journalctl -ef /usr/bin/niri`

> \[!WARNING]
>
> 自定义着色器没有向后兼容性保证。
> 我可能需要更改它们的接口，因为我在开发新功能。

示例：打开将用逐渐淡入的纯色渐变填充当前几何体。

```kdl
animations {
    window-open {
        duration-ms 250
        curve "linear"

        custom-shader r"
            vec4 open_color(vec3 coords_geo, vec3 size_geo) {
                vec4 color = vec4(0.0);

                if (0.0 <= coords_geo.x && coords_geo.x <= 1.0
                        && 0.0 <= coords_geo.y && coords_geo.y <= 1.0)
                {
                    vec4 from = vec4(1.0, 0.0, 0.0, 1.0);
                    vec4 to = vec4(0.0, 1.0, 0.0, 1.0);
                    color = mix(from, to, coords_geo.y);
                }

                return color * niri_clamped_progress;
            }
        "
    }
}
```

#### `window-close`

<sup>自：0.1.5</sup>

窗口关闭动画。

这个默认使用缓动类型。

```kdl
animations {
    window-close {
        duration-ms 150
        curve "ease-out-quad"
    }
}
```

##### `custom-shader`

<sup>自：0.1.6</sup>

您可以为窗口在关闭动画期间编写自定义着色器。

请参阅[此示例着色器](./examples/close_custom_shader.frag)以获得完整的文档和几种可试验的动画。

如果自定义着色器编译失败，niri将打印警告并回退到默认，或之前成功编译的着色器。
当作为systemd服务运行niri时，您可以在日志中看到警告：`journalctl -ef /usr/bin/niri`

> \[!WARNING]
>
> 自定义着色器没有向后兼容性保证。
> 我可能需要更改它们的接口，因为我在开发新功能。

示例：关闭将用逐渐淡出的纯色渐变填充当前几何体。

```kdl
animations {
    window-close {
        custom-shader r"
            vec4 close_color(vec3 coords_geo, vec3 size_geo) {
                vec4 color = vec4(0.0);

                if (0.0 <= coords_geo.x && coords_geo.x <= 1.0
                        && 0.0 <= coords_geo.y && coords_geo.y <= 1.0)
                {
                    vec4 from = vec4(1.0, 0.0, 0.0, 1.0);
                    vec4 to = vec4(0.0, 1.0, 0.0, 1.0);
                    color = mix(from, to, coords_geo.y);
                }

                return color * (1.0 - niri_clamped_progress);
            }
        "
    }
}
```

#### `horizontal-view-movement`

所有水平摄像机视图移动动画，例如：

*   当聚焦屏幕外的窗口且摄像机滚动到它时。
*   当屏幕外出现新窗口且摄像机滚动到它时。
*   水平触控板手势后（建议使用弹簧）。

```kdl
animations {
    horizontal-view-movement {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }
}
```

#### `window-movement`

<sup>自：0.1.5</sup>

工作区内单个窗口的移动。

包括：

*   用`move-column-left`和`move-column-right`移动窗口列。
*   用`move-window-up`和`move-window-down`在列内移动窗口。
*   窗口打开和关闭时移动窗口。
*   消耗/排出时窗口在列之间的移动。

此动画*不包括*摄像机视图移动，例如左右滚动工作区。

```kdl
animations {
    window-movement {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }
}
```

#### `window-resize`

<sup>自：0.1.5</sup>

窗口调整大小动画。

仅手动调整窗口大小有动画，即当您用`switch-preset-column-width`或`maximize-column`调整窗口大小时。
此外，非常小的调整（最多10像素）没有动画。

```kdl
animations {
    window-resize {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }
}
```

##### `custom-shader`

<sup>自：0.1.6</sup>

您可以为窗口在调整大小动画期间编写自定义着色器。

请参阅[此示例着色器](./examples/resize_custom_shader.frag)以获得完整的文档和几种可试验的动画。

如果自定义着色器编译失败，niri将打印警告并回退到默认，或之前成功编译的着色器。
当作为systemd服务运行niri时，您可以在日志中看到警告：`journalctl -ef /usr/bin/niri`

> \[!WARNING]
>
> 自定义着色器没有向后兼容性保证。
> 我可能需要更改它们的接口，因为我在开发新功能。

示例：调整大小将立即显示下一个（调整大小后）窗口纹理，拉伸到当前几何体。

```kdl
animations {
    window-resize {
        custom-shader r"
            vec4 resize_color(vec3 coords_curr_geo, vec3 size_curr_geo) {
                vec3 coords_tex_next = niri_geo_to_tex_next * coords_curr_geo;
                vec4 color = texture2D(niri_tex_next, coords_tex_next.st);
                return color;
            }
        "
    }
}
```

#### `config-notification-open-close`

配置解析错误和新默认配置通知的打开/关闭动画。

这个默认使用欠阻尼弹簧（`damping-ratio=0.6`），导致结束时轻微振荡。

```kdl
animations {
    config-notification-open-close {
        spring damping-ratio=0.6 stiffness=1000 epsilon=0.001
    }
}
```

#### `exit-confirmation-open-close`

<sup>自：25.08</sup>

退出确认对话框的打开/关闭动画。

这个默认使用欠阻尼弹簧（`damping-ratio=0.6`），导致结束时轻微振荡。

```kdl
animations {
    exit-confirmation-open-close {
        spring damping-ratio=0.6 stiffness=500 epsilon=0.01
    }
}
```

#### `screenshot-ui-open`

<sup>自：0.1.8</sup>

截图UI的打开（淡入）动画。

```kdl
animations {
    screenshot-ui-open {
        duration-ms 200
        curve "ease-out-quad"
    }
}
```

#### `overview-open-close`

<sup>自：25.05</sup>

[概览](./Overview)的打开/关闭缩放动画。

```kdl
animations {
    overview-open-close {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }
}
```

### 同步动画

<sup>自：0.1.5</sup>

有时，当两个动画旨在一起同步播放时，niri将使用相同配置驱动它们。

例如，如果窗口调整大小导致视图移动，则该视图移动动画也将使用`window-resize`配置（而不是`horizontal-view-movement`配置）。
当使用`center-focused-column "always"`时，这对于动画调整大小看起来很好特别重要。

作为另一个示例，在列中垂直调整窗口大小会导致其他窗口上下移动到新位置。
此移动将使用`window-resize`配置，而不是`window-movement`配置，以保持动画同步。

一些操作仍然缺少此同步逻辑，因为在某些情况下很难正确实现。
因此，为获得最佳结果，请考虑对相关动画使用相同参数（它们默认都相同）：

*   `horizontal-view-movement`
*   `window-movement`
*   `window-resize`