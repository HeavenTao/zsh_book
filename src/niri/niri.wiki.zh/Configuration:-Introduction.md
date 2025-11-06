### 按章节文档

您可以在这些wiki页面上找到配置文件各个部分的文档：

*   [`input {}`](./Configuration:-Input)
*   [`output "eDP-1" {}`](./Configuration:-Outputs)
*   [`binds {}`](./Configuration:-Key-Bindings)
*   [`switch-events {}`](./Configuration:-Switch-Events)
*   [`layout {}`](./Configuration:-Layout)
*   [顶级选项](./Configuration:-Miscellaneous)
*   [`window-rule {}`](./Configuration:-Window-Rules)
*   [`layer-rule {}`](./Configuration:-Layer-Rules)
*   [`animations {}`](./Configuration:-Animations)
*   [`gestures {}`](./Configuration:-Gestures)
*   [`debug {}`](./Configuration:-Debug-Options)
*   [`include "other.kdl"`](./Configuration:-Include)

### 加载

Niri将从`$XDG_CONFIG_HOME/niri/config.kdl`或`~/.config/niri/config.kdl`加载配置，回退到`/etc/niri/config.kdl`。
如果这些文件都缺失，niri将在`$XDG_CONFIG_HOME/niri/config.kdl`创建[默认配置文件](https://github.com/YaLTeR/niri/blob/main/resources/default-config.kdl)的内容，这些内容在构建时嵌入到niri二进制文件中。
请使用默认配置文件作为自定义配置的起点。

配置是实时重新加载的。
只需编辑并保存配置文件，您的更改就会应用。
这包括键盘绑定、输出设置（如模式）、窗口规则和所有其他内容。

您可以运行`niri validate`来解析配置并查看任何错误。

要使用不同的配置文件路径，请在`niri`的`--config`或`-c`参数中传入。

您还可以将`$NIRI_CONFIG`设置为配置文件的路径。
`--config`始终优先。
如果`--config`或`$NIRI_CONFIG`不指向实际文件，配置将不会被加载。
如果`$NIRI_CONFIG`被设置为空字符串，则会被忽略并使用默认配置位置。

### 语法

配置使用[KDL]编写。

#### 注释

以`//`开头的行是注释；它们被忽略。

此外，您可以在部分前面加上`/-`来注释掉整个部分：

```kdl
/-output "eDP-1" {
    // 这里面的所有内容都被忽略。
    // 显示器不会关闭
    // 因为整个部分都被注释掉了。
    off
}
```

#### 标志

niri中的切换选项通常表示为标志。
写出标志启用它，省略或注释掉它则禁用它。
例如：

```kdl
// "鼠标跟随焦点"已启用。
input {
    focus-follows-mouse

    // 其他设置...
}
```

```kdl
// "鼠标跟随焦点"已禁用。
input {
    // focus-follows-mouse

    // 其他设置...
}
```

#### 部分

大多数部分不能重复。例如：

```kdl
// 这是有效的：每个部分只出现一次。
input {
    keyboard {
        // ...
    }

    touchpad {
        // ...
    }
}
```

```kdl,must-fail
// 这是无效的：input部分出现了两次。
input {
    keyboard {
        // ...
    }
}

input {
    touchpad {
        // ...
    }
}
```

例外情况，例如，通过名称配置不同设备的部分：

<!-- NOTE: this may break in the future -->

```kdl
output "eDP-1" {
    // ...
}

// 这是有效的：这个部分配置了不同的输出。
output "HDMI-A-1" {
    // ...
}

// 这是无效的："eDP-1"已在上面出现。
// 它会抛出配置解析错误，或者以其他方式无法工作。
output "eDP-1" {
    // ...
}
```

### 默认值

省略配置文件的大部分部分将使该部分保留默认值。
一个显著的例外是[`binds {}`](./Configuration:-Key-Bindings)：它们不会填入默认值，因此请确保不要删除此部分。

### 破坏性变更政策

按照规则，niri更新不应破坏现有配置文件。
（例如，从niri v0.1.0的默认配置到我写这篇文章时的v25.02仍然可以正常解析。）

对于解析错误可以例外。
例如，niri过去接受相同键的多个绑定，但这是非预期的，也没有任何作用（总是使用第一个绑定）。
一个补丁版本将niri从静默接受更改为导致解析失败。
这不是一条通则，我将在决定继续进行此类破坏性更改之前考虑每种潜在影响。

请记住，破坏性变更政策仅适用于niri发布。
发布之间的提交偶尔会破坏配置，因为新功能正在被完善。
然而，我确实尝试限制这些，因为有几个人正在运行git构建。

[KDL]: https://kdl.dev/