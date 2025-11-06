### 概述

在本节中，您可以配置键盘和鼠标等输入设备，以及一些与输入相关的选项。

每种设备类型都有一个部分：`keyboard`、`touchpad`、`mouse`、`trackpoint`、`tablet`、`touch`。
这些部分中的设置将应用于该类型的所有设备。
目前，无法单独配置特定设备（但这是计划中的）。

所有设置一览：

```kdl
input {
    keyboard {
        xkb {
            // layout "us"
            // variant "colemak_dh_ortho"
            // options "compose:ralt,ctrl:nocaps"
            // model ""
            // rules ""
            // file "~/.config/keymap.xkb"
        }

        // repeat-delay 600
        // repeat-rate 25
        // track-layout "global"
        numlock
    }

    touchpad {
        // off
        tap
        // dwt
        // dwtp
        // drag false
        // drag-lock
        natural-scroll
        // accel-speed 0.2
        // accel-profile "flat"
        // scroll-factor 1.0
        // scroll-factor vertical=1.0 horizontal=-2.0
        // scroll-method "two-finger"
        // scroll-button 273
        // scroll-button-lock
        // tap-button-map "left-middle-right"
        // click-method "clickfinger"
        // left-handed
        // disabled-on-external-mouse
        // middle-emulation
    }

    mouse {
        // off
        // natural-scroll
        // accel-speed 0.2
        // accel-profile "flat"
        // scroll-factor 1.0
        // scroll-factor vertical=1.0 horizontal=-2.0
        // scroll-method "no-scroll"
        // scroll-button 273
        // scroll-button-lock
        // left-handed
        // middle-emulation
    }

    trackpoint {
        // off
        // natural-scroll
        // accel-speed 0.2
        // accel-profile "flat"
        // scroll-method "on-button-down"
        // scroll-button 273
        // scroll-button-lock
        // left-handed
        // middle-emulation
    }

    trackball {
        // off
        // natural-scroll
        // accel-speed 0.2
        // accel-profile "flat"
        // scroll-method "on-button-down"
        // scroll-button 273
        // scroll-button-lock
        // left-handed
        // middle-emulation
    }

    tablet {
        // off
        map-to-output "eDP-1"
        // left-handed
        // calibration-matrix 1.0 0.0 0.0 0.0 1.0 0.0
    }

    touch {
        // off
        map-to-output "eDP-1"
        // calibration-matrix 1.0 0.0 0.0 0.0 1.0 0.0
    }

    // disable-power-key-handling
    // warp-mouse-to-focus
    // focus-follows-mouse max-scroll-amount="0%"
    // workspace-auto-back-and-forth

    // mod-key "Super"
    // mod-key-nested "Alt"
}
```

### 键盘

#### 布局

在`xkb`部分中，您可以设置布局、变体、选项、模型和规则。
这些直接传递给libxkbcommon，其他大多数Wayland合成器也使用它。
有关更多信息，请参见`xkeyboard-config(7)`手册。

```kdl
input {
    keyboard {
        xkb {
            layout "us"
            variant "colemak_dh_ortho"
            options "compose:ralt,ctrl:nocaps"
        }
    }
}
```

> \[!TIP]
>
> <sup>自：25.02</sup>
>
> 或者，您可以直接设置包含xkb键映射的.xkb文件的路径。
> 这会覆盖所有其他xkb设置。
>
> ```kdl
> input {
>     keyboard {
>         xkb {
>             file "~/.config/keymap.xkb"
>         }
>     }
> }
> ```

> \[!NOTE]
>
> <sup>自：25.08</sup>
>
> 如果`xkb`部分为空（默认情况下），niri将通过D-Bus从systemd-localed获取xkb设置。
> 这样，例如，系统安装程序可以动态设置niri键盘布局。
> 您可以在`localectl`中查看此布局，并使用`localectl set-x11-keymap`更改它，例如：
>
> ```sh
> $ localectl set-x11-keymap "us" "" "colemak_dh_ortho" "compose:ralt,ctrl:nocaps"
> $ localectl
> System Locale: LANG=en_US.UTF-8
>                LC_NUMERIC=ru_RU.UTF-8
>                LC_TIME=ru_RU.UTF-8
>                LC_MONETARY=ru_RU.UTF-8
>                LC_PAPER=ru_RU.UTF-8
>                LC_MEASUREMENT=ru_RU.UTF-8
>     VC Keymap: us-colemak_dh_ortho
>    X11 Layout: us
>   X11 Variant: colemak_dh_ortho
>   X11 Options: compose:ralt,ctrl:nocaps
> ```
>
> 默认情况下，`localectl`会将TTY键映射设置为XKB键映射的最接近匹配。
> 您可以使用`--no-convert`标志防止这种情况，例如：`localectl set-x11-keymap --no-convert "us,ru"`。
>
> 这些设置也会被其他一些程序使用，如GDM。

使用多个布局时，niri可以全局记住当前布局（默认）或按窗口记住。
您可以使用`track-layout`选项控制这一点。

*   `global`：布局更改对所有窗口都是全局的。
*   `window`：为每个窗口单独跟踪布局。

```kdl
input {
    keyboard {
        track-layout "global"
    }
}
```

#### 重复

延迟是键盘重复开始前的毫秒数。
速率是每秒字符数。

```kdl
input {
    keyboard {
        repeat-delay 600
        repeat-rate 25
    }
}
```

#### 数字锁定

<sup>自：25.05</sup>

设置`numlock`标志以在启动时自动打开数字锁定。

如果您使用的是在常规键上叠加数字锁定键的笔记本电脑键盘，可能需要禁用（注释掉）`numlock`。

```kdl
input {
    keyboard {
        numlock
    }
}
```

### 指向设备

指向设备的大多数设置直接传递给libinput。
其他Wayland合成器也使用libinput，因此您可能会在那里找到相同的设置。
对于像`tap`这样的标志，省略它们或注释掉它们以禁用设置。

一些设置在输入设备之间是通用的：

*   `off`：如果设置，则不会从此设备发送事件。

一些设置在`touchpad`、`mouse`、`trackpoint`和`trackball`之间是通用的：

*   `natural-scroll`：如果设置，则反转滚动方向。
*   `accel-speed`：指针加速速度，有效值从`-1.0`到`1.0`，默认为`0.0`。
*   `accel-profile`：可以是`adaptive`（默认）或`flat`（禁用指针加速）。
*   `scroll-method`：何时生成滚动事件而不是指针移动事件，可以是`no-scroll`、`two-finger`、`edge`或`on-button-down`。
    默认和受支持的方法因设备类型而异。
*   `scroll-button`：<sup>自：0.1.10</sup>用于`on-button-down`滚动方法的按钮代码。您可以在`libinput debug-events`中找到它。
*   `scroll-button-lock`：<sup>自：25.08</sup>启用时，按钮不需要一直按住。按一次启动滚动，按第二次停止滚动，双击作为基础按钮的单击。
*   `left-handed`：如果设置，则将设备更改为左手模式。
*   `middle-emulation`：通过同时按下左右鼠标按钮来模拟中键单击。

`touchpad`特有的设置：

*   `tap`：轻触点击。
*   `dwt`：打字时禁用。
*   `dwtp`：使用轨迹球时禁用。
*   `drag`：<sup>自：25.05</sup>可以是`true`或`false`，控制是否启用轻触拖拽。
*   `drag-lock`：<sup>自：25.02</sup>如果设置，拖拽时短暂抬起手指不会释放拖拽的项目。请参见[libinput文档](https://wayland.freedesktop.org/libinput/doc/latest/tapping.html#tap-and-drag)。
*   `tap-button-map`：可以是`left-right-middle`或`left-middle-right`，控制哪两个手指轻触和三个手指轻触对应哪个按钮。
*   `click-method`：可以是`button-areas`或`clickfinger`，更改[点击方法](https://wayland.freedesktop.org/libinput/doc/latest/clickpad-softbuttons.html)。
*   `disabled-on-external-mouse`：插入外部指针设备时不要发送事件。

`touchpad`和`mouse`特有的设置：

*   `scroll-factor`：<sup>自：0.1.10</sup>按此值缩放滚动速度。

    <sup>自：25.08</sup> 您还可以分别覆盖水平和垂直滚动因子，如下所示：`scroll-factor horizontal=2.0 vertical=-1.0`

`tablet`和`touch`特有的设置：

*   `calibration-matrix`：设置六个浮点数以更改校准矩阵。请参见[`LIBINPUT_CALIBRATION_MATRIX`文档](https://wayland.freedesktop.org/libinput/doc/latest/device-configuration-via-udev.html)中的示例。
    *   <sup>自：25.02</sup> 用于`tablet`
    *   <sup>自：下一个版本</sup> 用于`touch`

平板电脑和触摸屏是绝对指向设备，可以映射到特定输出，如下所示：

```kdl
input {
    tablet {
        map-to-output "eDP-1"
    }

    touch {
        map-to-output "eDP-1"
    }
}
```

有效的输出名称与用于输出配置的名称相同。

<sup>自：0.1.7</sup> 当平板电脑未映射到任何输出时，它将映射到所有连接输出的并集，不进行宽高比校正。

### 通用设置

这些设置不特定于特定的输入设备。

#### `disable-power-key-handling`

默认情况下，niri将接管电源按钮以使其进入睡眠而不是关机。
如果您想在其他地方配置电源按钮（即`logind.conf`），请设置此选项。

```kdl
input {
    disable-power-key-handling
}
```

#### `warp-mouse-to-focus`

使鼠标扭曲到新聚焦的窗口。

如果光标已被隐藏，则不会使其可见。

```kdl
input {
    warp-mouse-to-focus
}
```

默认情况下，光标在水平和垂直方向上*分别*扭曲。
也就是说，如果仅水平移动鼠标就足以将其放在新聚焦的窗口内，则鼠标将仅水平移动，而不是垂直移动。

<sup>自：25.05</sup> 您可以使用`mode`属性自定义此行为。

*   `mode="center-xy"`：按X和Y坐标一起扭曲。
    因此，如果鼠标在新聚焦窗口的任何地方，它将扭曲到窗口中心。
*   `mode="center-xy-always"`：按X和Y坐标一起扭曲，即使鼠标已经在新聚焦窗口的某个地方。

```kdl
input {
    warp-mouse-to-focus mode="center-xy"
}
```

#### `focus-follows-mouse`

在鼠标移动到窗口和输出上时自动聚焦它们。

```kdl
input {
    focus-follows-mouse
}
```

<sup>自：0.1.8</sup> 您可以选择性地设置`max-scroll-amount`。
然后，如果聚焦窗口会导致视图滚动超过设定的数量，focus-follows-mouse将不会聚焦窗口。
该值是工作区域宽度的百分比。

```kdl
input {
    // 当滚动最多为屏幕的10%时允许focus-follows-mouse。
    focus-follows-mouse max-scroll-amount="10%"
}
```

```kdl
input {
    // 仅当不会滚动视图时才允许focus-follows-mouse。
    focus-follows-mouse max-scroll-amount="0%"
}
```

#### `workspace-auto-back-and-forth`

通常，通过索引两次切换到相同的工作区不会执行任何操作（因为您已经在该工作区上）。
如果启用此标志，通过索引两次切换到相同的工作区将切换回前一个工作区。

即使工作区在此期间被重新排序，Niri也会正确切换到您来自的工作区。

```kdl
input {
    workspace-auto-back-and-forth
}
```

#### `mod-key`, `mod-key-nested`

<sup>自：25.05</sup>

自定义[key bindings](./Configuration:-Key-Bindings)的`Mod`键。
只允许有效的修饰符，例如`Super`、`Alt`、`Mod3`、`Mod5`、`Ctrl`、`Shift`。

默认情况下，当在TTY上运行niri时，`Mod`等于`Super`，当在嵌套的winit窗口中运行niri时，`Mod`等于`Alt`。

> \[!NOTE]
> 有很多默认绑定使用Mod，它们都不会"传递"到底层窗口。
> 您可能不想将`mod-key`设置为Ctrl或Shift，因为Ctrl通常用于应用程序热键，而Shift用于常规输入。

```kdl
// 交换mod键：正常使用Alt，在嵌套窗口中使用Super。
input {
    mod-key "Alt"
    mod-key-nested "Super"
}
```