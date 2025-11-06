### 概述

键绑定在配置的`binds {}`部分中声明。

> \[!NOTE]
> 这是少数几个如果省略它*不会*自动填充默认值的部分之一，因此请确保从默认配置中复制它。

每个绑定是热键后跟一个用花括号括起来的操作。
例如：

```kdl
binds {
    Mod+Left { focus-column-left; }
    Super+Alt+L { spawn "swaylock"; }
}
```

热键由`+`号分隔的修饰符组成，最后跟一个XKB键名。

有效修饰符为：

*   `Ctrl`或`Control`；
*   `Shift`；
*   `Alt`；
*   `Super`或`Win`；
*   `ISO_Level3_Shift`或`Mod5`——这是某些布局上的AltGr键；
*   `ISO_Level5_Shift`：可以与xkb lv5选项一起使用，如`lv5:caps_switch`；
*   `Mod`。

`Mod`是一个特殊修饰符，在TTY上运行niri时等于`Super`，在作为嵌套winit窗口运行niri时等于`Alt`。
这样，您可以在窗口中测试niri而不与宿主合成器的键绑定造成太多冲突。
出于这个原因，大多数默认键使用`Mod`修饰符。

<sup>自：25.05</sup> 您可以在[配置的`input`部分](./Configuration:-Input#mod-key-mod-key-nested)自定义`Mod`键。

> \[!TIP]
> 要查找特定键的XKB名称，您可以使用像[`wev`](https://git.sr.ht/~sircmpwn/wev)这样的程序。
>
> 从终端打开它并按您想检测的键。
> 在终端中，您将看到如下输出：
>
>     [14:     wl_keyboard] key: serial: 757775; time: 44940343; key: 113; state: 1 (pressed)
>                           sym: Left         (65361), utf8: ''
>     [14:     wl_keyboard] key: serial: 757776; time: 44940432; key: 113; state: 0 (released)
>                           sym: Left         (65361), utf8: ''
>     [14:     wl_keyboard] key: serial: 757777; time: 44940753; key: 114; state: 1 (pressed)
>                           sym: Right        (65363), utf8: ''
>     [14:     wl_keyboard] key: serial: 757778; time: 44940846; key: 114; state: 0 (released)
>                           sym: Right        (65363), utf8: ''
>
> 这里，看`sym: Left`和`sym: Right`：这些是键名。
> 在这个例子中我按了左右箭头。
>
> 请记住，绑定带Shift的键需要拼写Shift和键的未Shift版本，根据您的XKB布局。
> 例如，在美国QWERTY布局上，<kbd><</kbd>在<kbd>Shift</kbd> + <kbd>,</kbd>上，所以要绑定它，您需要拼写类似`Mod+Shift+Comma`。
>
> 再举一个例子，如果您配置了法语[BÉPO](https://en.wikipedia.org/wiki/B%C3%89PO) XKB布局，您的<kbd><</kbd>在<kbd>AltGr</kbd> + <kbd>«</kbd>上。<kbd>AltGr</kbd>是`ISO_Level3_Shift`，或等同于`Mod5`，所以要绑定它，您需要拼写类似`Mod+Mod5+guillemotleft`。
>
> 当解析拉丁键时，niri将搜索具有拉丁键的第一个已配置XKB布局。
> 因此，例如配置了美国QWERTY和RU布局，美国QWERTY将用于拉丁绑定。

<sup>自：0.1.8</sup> 绑定默认会重复（即按住绑定会使其反复触发）。
您可以使用`repeat=false`为特定绑定禁用此功能：

```kdl
binds {
    Mod+T repeat=false { spawn "alacritty"; }
}
```

绑定也可以有冷却时间，这将限制绑定频率并防止其快速重复触发。

```kdl
binds {
    Mod+T cooldown-ms=500 { spawn "alacritty"; }
}
```

这主要用于滚动绑定。

### 滚动绑定

您可以使用以下语法绑定鼠标滚轮滚动。
这些绑定将根据`natural-scroll`设置改变方向。

```kdl
binds {
    Mod+WheelScrollDown cooldown-ms=150 { focus-workspace-down; }
    Mod+WheelScrollUp   cooldown-ms=150 { focus-workspace-up; }
    Mod+WheelScrollRight                { focus-column-right; }
    Mod+WheelScrollLeft                 { focus-column-left; }
}
```

同样，您可以绑定触摸板滚动"点击"。
触摸板滚动是连续的，所以对于这些绑定，它根据移动距离分为离散间隔。

这些绑定也受触摸板`natural-scroll`的影响，所以这些示例绑定是"反转的"，因为niri默认为触摸板启用`natural-scroll`。

```kdl
binds {
    Mod+TouchpadScrollDown { spawn "wpctl" "set-volume" "@DEFAULT_AUDIO_SINK@" "0.02+"; }
    Mod+TouchpadScrollUp   { spawn "wpctl" "set-volume" "@DEFAULT_AUDIO_SINK@" "0.02-"; }
}
```

鼠标滚轮和触摸板滚动绑定都会在按住修饰键时阻止应用程序接收任何滚动事件。
例如，如果您有`Mod+WheelScrollDown`绑定，那么在按住`Mod`时，所有鼠标滚轮滚动将被niri消耗。

### 鼠标点击绑定

<sup>自：25.01</sup>

您可以使用以下语法绑定鼠标点击。

```kdl
binds {
    Mod+MouseLeft    { close-window; }
    Mod+MouseRight   { close-window; }
    Mod+MouseMiddle  { close-window; }
    Mod+MouseForward { close-window; }
    Mod+MouseBack    { close-window; }
}
```

鼠标点击作用于点击时聚焦的窗口，而不是您点击的窗口。

请注意，绑定`Mod+MouseLeft`或`Mod+MouseRight`将覆盖相应的手势（移动或调整窗口大小）。

### 自定义热键覆盖标题

<sup>自：25.02</sup>

热键覆盖（重要热键对话框）显示一个硬编码的绑定列表。
您可以使用`hotkey-overlay-title`属性自定义此列表。

要将绑定添加到热键覆盖，请将属性设置为要显示的标题：

```kdl
binds {
    Mod+Shift+S hotkey-overlay-title="切换深色/浅色样式" { spawn "some-script.sh"; }
}
```

带有自定义标题的绑定列在硬编码绑定之后和非自定义Spawn绑定之前。

要从热键覆盖中删除硬编码绑定，请将属性设置为null：

```kdl
binds {
    Mod+Q hotkey-overlay-title=null { close-window; }
}
```

> \[!TIP]
> 当多个键组合绑定到相同操作时：
>
> *   如果任何绑定具有自定义热键覆盖标题，niri将显示该绑定。
> *   否则，如果任何绑定具有null标题，niri将隐藏该绑定。
> *   否则，niri将显示第一个键组合。

自定义标题支持[Pango标记](https://docs.gtk.org/Pango/pango_markup.html)：

```kdl
binds {
    Mod+Shift+S hotkey-overlay-title="<b>切换</b> <span foreground='red'>深色</span>/浅色样式" { spawn "some-script.sh"; }
}
```

![自定义标记示例。](https://github.com/user-attachments/assets/2a2ba914-bfa7-4dfa-bb5e-49839034765d)

### 操作

您可以绑定的每个操作也可以通过`niri msg action`进行程序化调用。
运行`niri msg action`以获取操作的完整列表及其简短描述。

以下是几个需要更多解释的操作。

#### `spawn`

运行程序。

`spawn`接受程序二进制文件的路径作为第一个参数，后跟程序的参数。
例如：

```kdl
binds {
    // 运行alacritty。
    Mod+T { spawn "alacritty"; }

    // 运行`wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.1+`。
    XF86AudioRaiseVolume { spawn "wpctl" "set-volume" "@DEFAULT_AUDIO_SINK@" "0.1+"; }
}
```

> \[!TIP]
>
> <sup>自：0.1.5</sup>
> Spawn绑定有一个特殊的`allow-when-locked=true`属性，使其即使在会话锁定时也能工作：
>
> ```kdl
> binds {
>     // 此静音绑定即使在会话锁定时也能工作。
>     XF86AudioMute allow-when-locked=true { spawn "wpctl" "set-mute" "@DEFAULT_AUDIO_SINK@" "toggle"; }
> }
> ```

对于`spawn`，niri*不会*使用shell运行命令，这意味着您需要手动分离参数。
请参见下面使用shell的操作[`spawn-sh`](#spawn-sh)。

```kdl
binds {
    // 正确：每个参数都在自己的引号中。
    Mod+T { spawn "alacritty" "-e" "/usr/bin/fish"; }

    // 错误：将整个`alacritty -e /usr/bin/fish`字符串解释为二进制文件路径。
    Mod+D { spawn "alacritty -e /usr/bin/fish"; }

    // 错误：将`-e /usr/bin/fish`作为单个参数传递，alacritty将无法理解。
    Mod+Q { spawn "alacritty" "-e /usr/bin/fish"; }
}
```

这也意味着您无法扩展环境变量或`~`。
如果需要，您可以手动通过shell运行命令。

```kdl
binds {
    // 错误：这里没有shell扩展。这些字符串将按字面意思传递给程序。
    Mod+T { spawn "grim" "-o" "$MAIN_OUTPUT" "~/screenshot.png"; }

    // 正确：手动通过shell运行，以便它可以扩展参数。
    // 注意整个命令作为单个参数传递，
    // 因为shell将按空格进行自己的参数分割。
    Mod+D { spawn "sh" "-c" "grim -o $MAIN_OUTPUT ~/screenshot.png"; }

    // 您也可以使用shell运行多个命令，
    // 使用管道、进程替换等。
    Mod+Q { spawn "sh" "-c" "notify-send clipboard \"$(wl-paste)\""; }
}
```

作为特殊情况，niri将扩展`~`到主目录*仅*在程序名的开头。

```kdl
binds {
    // 这将工作：开头只有一个~。
    Mod+T { spawn "~/scripts/do-something.sh"; }
}
```

#### `spawn-sh`

<sup>自：25.08</sup>

通过shell运行命令。

参数是单个字符串，原样传递给`sh`。
您可以像预期的那样使用shell变量、管道、`~`扩展等。

```kdl
binds {
    // 与spawn-sh配合：所有参数在同一个字符串中。
    Mod+D { spawn-sh "alacritty -e /usr/bin/fish"; }

    // 与spawn-sh配合：shell变量($MAIN_OUTPUT)，~扩展。
    Mod+T { spawn-sh "grim -o $MAIN_OUTPUT ~/screenshot.png"; }

    // 与spawn-sh配合：进程替换。
    Mod+Q { spawn-sh "notify-send clipboard \"$(wl-paste)\""; }

    // 与spawn-sh配合：多个命令。
    Super+Alt+S { spawn-sh "pkill orca || exec orca"; }
}
```

`spawn-sh "some command"`等同于`spawn "sh" "-c" "some command"`——只是一个不那么令人困惑的简写。
请记住，通过shell会产生微小的性能损失，相比直接`spawn`某些二进制文件。

使用`sh`是硬编码的，与其他合成器一致。
如果想要不同的shell，请使用`spawn`写出，例如`spawn "fish" "-c" "some fish command"`。

#### `quit`

显示确认对话框后退出niri以避免意外触发。

```kdl
binds {
    Mod+Shift+E { quit; }
}
```

如果您想跳过确认对话框，请设置标志，如下所示：

```kdl
binds {
    Mod+Shift+E { quit skip-confirmation=true; }
}
```

#### `do-screen-transition`

<sup>自：0.1.6</sup>

冻结屏幕短暂时间然后交叉淡入新内容。

```kdl
binds {
    Mod+Return { do-screen-transition; }
}
```

此操作主要用于从更改系统主题或样式的脚本中触发（例如在浅色和深色之间）。
它使这样的过渡，其中窗口逐个改变其样式，看起来平滑和同步。

例如，使用GNOME颜色方案设置：

```shell
niri msg action do-screen-transition
dconf write /org/gnome/desktop/interface/color-scheme "\"prefer-dark\""
```

默认情况下，屏幕冻结250毫秒给窗口时间重绘，然后交叉淡入。
您可以这样设置此延迟：

```kdl
binds {
    Mod+Return { do-screen-transition delay-ms=100; }
}
```

或者，在脚本中：

```shell
niri msg action do-screen-transition --delay-ms 100
```

#### `toggle-window-rule-opacity`

<sup>自：25.02</sup>

切换聚焦窗口的不透明度窗口规则。
这仅在窗口的不透明度窗口规则已设置为半透明时有效。

```kdl
binds {
    Mod+O { toggle-window-rule-opacity; }
}
```

#### `screenshot`, `screenshot-screen`, `screenshot-window`

用于截图的操作。

*   `screenshot`：打开内置的交互式截图UI。
*   `screenshot-screen`, `screenshot-window`：分别截图聚焦的屏幕或窗口。

截图既存储到剪贴板又保存到磁盘，根据[`screenshot-path`选项](./Configuration:-Miscellaneous#screenshot-path)。

<sup>自：25.02</sup> 您可以使用`write-to-disk=false`属性为特定绑定禁用保存到磁盘：

```kdl
binds {
    Ctrl+Print { screenshot-screen write-to-disk=false; }
    Alt+Print { screenshot-window write-to-disk=false; }
}
```

在交互式截图UI中，按<kbd>Ctrl</kbd><kbd>C</kbd>将复制截图到剪贴板而不写入磁盘。

<sup>自：25.05</sup> 您可以使用`show-pointer=false`属性在截图中隐藏鼠标指针：

```kdl
binds {
    // 指针将默认隐藏
    // （您仍可以通过按P显示它）。
    Print { screenshot show-pointer=false; }

    // 指针将在截图中隐藏。
    Ctrl+Print { screenshot-screen show-pointer=false; }
}
```

#### `toggle-keyboard-shortcuts-inhibit`

<sup>自：25.02</sup>

像远程桌面客户端和软件KVM开关这样的应用程序可能请求niri停止处理其键盘快捷键，以便它们可以例如将按键原样转发到远程机器。
`toggle-keyboard-shortcuts-inhibit`是一个切换抑制器的逃生门。
绑定它是个好主意，这样有缺陷的应用程序就不能劫持您的会话。

```kdl
binds {
    Mod+Escape { toggle-keyboard-shortcuts-inhibit; }
}
```

您还可以使用`allow-inhibiting=false`属性使某些绑定忽略抑制。
它们将始终由niri处理，永远不会传递给窗口。

```kdl
binds {
    // 此绑定将始终工作，即使在使用虚拟机时。
    Super+Alt+L allow-inhibiting=false { spawn "swaylock"; }
}
```