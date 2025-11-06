### 概述

<sup>自：0.1.10</sup>

开关事件绑定在配置的`switch-events {}`部分中声明。

以下是您可以绑定的所有事件概览：

```kdl
switch-events {
    lid-close { spawn "notify-send" "笔记本电脑盖子已关闭！"; }
    lid-open { spawn "notify-send" "笔记本电脑盖子已打开！"; }
    tablet-mode-on { spawn "bash" "-c" "gsettings set org.gnome.desktop.a11y.applications screen-keyboard-enabled true"; }
    tablet-mode-off { spawn "bash" "-c" "gsettings set org.gnome.desktop.a11y.applications screen-keyboard-enabled false"; }
}
```

语法类似于键绑定。
目前，仅支持[`spawn`操作](./Configuration:-Key-Bindings#spawn)。

> \[!NOTE]
> 与键绑定不同，开关事件绑定*总是*执行，即使会话被锁定。

### `lid-close`, `lid-open`

这些事件对应于笔记本电脑盖子的关闭和打开。

请注意，niri已经会自动根据笔记本电脑盖子的状态打开和关闭内置笔记本电脑显示器。

```kdl
switch-events {
    lid-close { spawn "notify-send" "笔记本电脑盖子已关闭！"; }
    lid-open { spawn "notify-send" "笔记本电脑盖子已打开！"; }
}
```

### `tablet-mode-on`, `tablet-mode-off`

当可转换笔记本电脑进入或退出平板模式时，这些事件触发。
在平板模式下，键盘和鼠标通常无法访问，因此您可以使用这些事件激活屏幕键盘。

```kdl
switch-events {
    tablet-mode-on { spawn "bash" "-c" "gsettings set org.gnome.desktop.a11y.applications screen-keyboard-enabled true"; }
    tablet-mode-off { spawn "bash" "-c" "gsettings set org.gnome.desktop.a11y.applications screen-keyboard-enabled false"; }
}
```