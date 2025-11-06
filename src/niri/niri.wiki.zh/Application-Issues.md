### Electron应用程序

基于Electron的应用程序可以直接在Wayland上运行，但这不是默认设置。

对于Electron > 28，您可以设置环境变量：

```kdl
environment {
    ELECTRON_OZONE_PLATFORM_HINT "auto"
}
```

对于早期版本，您需要向目标应用程序传递命令行标志：

    --enable-features=UseOzonePlatform --ozone-platform-hint=auto

如果应用程序有[桌面条目](https://specifications.freedesktop.org/menu-spec/latest/menu-add-example.html)，您可以将命令行参数放入`Exec`部分。

### VSCode

如果您遇到某些VSCode热键的问题，请尝试启动`Xwayland`并为VSCode设置`DISPLAY=:0`环境变量。
也就是说，仍然使用Wayland后端运行VSCode，但将`DISPLAY`设置为正在运行的Xwayland实例。
显然，VSCode目前无条件地向X服务器查询键映射。

### WezTerm

> \[!NOTE]
> 这两个问题似乎在WezTerm的夜间构建中已修复。

WezTerm中有一个[错误](https://github.com/wezterm/wezterm/issues/4708)，它等待零大小的Wayland配置事件，所以它的窗口在niri中从未显示。要解决这个问题，请在niri配置中放入此窗口规则（包含在默认配置中）：

```kdl
window-rule {
    match app-id=r#"^org\.wezfurlong\.wezterm$"#
    default-column-width {}
}
```

这个空的默认列宽度让WezTerm选择自己的初始宽度，使其正确显示。

WezTerm中还有[另一个错误](https://github.com/wezterm/wezterm/issues/6472)，导致它在平铺状态下选择错误的大小，并阻止调整大小。
Niri通过[`prefer-no-csd`](./Configuration:-Miscellaneous#prefer-no-csd)将窗口置于平铺状态。
因此，如果您遇到此问题，请在niri配置中注释掉`prefer-no-csd`并重新启动WezTerm。

### Ghidra

像Ghidra这样的某些Java应用程序在xwayland-satellite下可能显示为空白。
要修复此问题，请使用`_JAVA_AWT_WM_NONREPARENTING=1`环境变量运行它们。

### Zen浏览器

由于某些原因，Zen浏览器中禁用了DMABUF屏幕录制，因此在niri上屏幕录制无法开箱即用。
要修复此问题，请打开`about:config`并将`widget.dmabuf.force-enabled`设置为`true`。

### 全屏游戏

一些视频游戏，无论是Linux原生还是在Wine上，在使用非堆叠桌面环境时都有各种问题。
大多数这些问题可以通过Valve的[gamescope](https://github.com/ValveSoftware/gamescope)避免，例如：

```sh
gamescope -f -w 1920 -h 1080 -W 1920 -H 1080 --force-grab-cursor --backend sdl -- <game>
```

此命令将在1080p全屏下运行*<game>*——请确保将宽度和高度值替换为您想要的分辨率。
`--force-grab-cursor`强制gamescope使用相对鼠标移动，防止光标在多显示器设置中逃离游戏窗口。
请注意，目前还需要`--backend sdl`，因为gamescope的默认Wayland后端不能正确锁定光标（可能与https://github.com/ValveSoftware/gamescope/issues/1711有关）。

Steam用户应该通过游戏的[启动选项](https://help.steampowered.com/en/faqs/view/7D01-D2DD-D75E-2955)使用gamescope，将游戏可执行文件替换为`%command%`。
其他游戏启动器如[Lutris](https://lutris.net/)有自己设置gamescope选项的方式。

使用此方法运行基于X11的游戏不需要Xwayland，因为gamescope会创建自己的Xwayland服务器。
您也可以通过向gamescope传递`--expose-wayland`来运行Wayland原生游戏，从而消除X11。

### Steam

在某些系统上，Steam会显示完全黑色的窗口。
要修复此问题，请通过Steam的托盘图标或盲目找到窗口左上角的Steam菜单导航到设置->界面，然后**禁用**网页视图中的GPU加速渲染。
重新启动Steam，现在应该正常工作了。

如果您不想禁用GPU加速渲染，可以尝试传递启动参数`-system-composer`。

Steam通知不通过标准通知守护程序运行，而是以浮动窗口的形式显示在屏幕中央。
您可以通过在niri配置中添加窗口规则将其移动到更方便的位置：

```kdl
window-rule {
    match app-id="steam" title=r#"^notificationtoasts_\d+_desktop$"#
    default-floating-position x=10 y=10 relative-to="bottom-right"
}
```

### Waybar和其他GTK 3组件

如果您的Waybar有圆角并且角落显示黑色像素，请将Waybar不透明度设置为0.99，这应该能修复它。

GTK 3似乎有一个错误，即使有圆角它也会将表面报告为完全不透明。
这导致niri用黑色填充角落内的透明像素。

将表面不透明度设置为低于1的值可以解决问题，因为这样GTK不再将表面报告为不透明。