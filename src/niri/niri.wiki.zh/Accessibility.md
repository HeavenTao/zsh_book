## 屏幕阅读器

<sup>自：25.08</sup>

当作为完整的桌面会话运行时，niri对屏幕阅读器（特别是[Orca](https://orca.gnome.org)）有基本支持，即您需要通过显示管理器或通过`niri-session`启动niri。
为了避免与已运行的合成器冲突，当作为嵌套窗口启动，或作为TTY上的普通`/usr/bin/niri`启动时，niri不会暴露辅助功能接口。

我们实现了`org.freedesktop.a11y.KeyboardMonitor` D-Bus接口供Orca监听和捕获键盘按键，并通过[AccessKit](https://accesskit.dev)暴露主要的niri UI元素。
具体来说，niri会宣布：

*   工作区切换，例如当您切换到第二个工作区时会说"工作区2"；
*   退出确认对话框（默认按<kbd>Super</kbd><kbd>Shift</kbd><kbd>E</kbd>出现）；
*   进入截图UI和概览（niri会在这些被聚焦时说出来，目前没有其他内容）；
*   每当发生配置解析错误时；
*   重要热键列表（目前作为一个大公告，没有标签导航；默认按<kbd>Super</kbd><kbd>Shift</kbd><kbd>/</kbd>出现）。

这是一个演示视频，请打开声音观看。

<video controls src="https://github.com/user-attachments/assets/afceba6f-79f1-47ec-b859-a0fcb7f8eae3">

https://github.com/user-attachments/assets/afceba6f-79f1-47ec-b859-a0fcb7f8eae3

</video>

确保[Xwayland](./Xwayland)正常工作，然后运行`orca`。
默认配置将<kbd>Super</kbd><kbd>Alt</kbd><kbd>S</kbd>绑定为切换Orca，这是标准的按键绑定。

请注意有一些限制：

*   我们还没有Alt-Tab窗口切换器；正在开发中。
*   我们没有将焦点移动到layer-shell面板的绑定。这不难添加，但最好与LXQt/Xfce就具体如何工作达成一些共识或先例。
*   您需要连接并启用屏幕。没有屏幕，niri不会给任何窗口焦点。这对视力正常的用户来说是有意义的，我不完全确定对辅助功能来说什么最有意义（也许，用虚拟显示器会更好解决）。
*   您需要工作的EGL（硬件加速）。
*   我们还没有屏幕遮罩功能。

如果您正在发布niri并希望使其开箱即用地更好地支持屏幕阅读器，请考虑对默认niri配置进行以下更改：

*   将默认终端从Alacritty更改为支持屏幕阅读器的终端。例如，[GNOME控制台](https://gitlab.gnome.org/GNOME/console)或[GNOME终端](https://gitlab.gnome.org/GNOME/gnome-terminal)应该能很好地工作。
*   将默认应用程序启动器和屏幕锁更改为支持屏幕阅读器的程序。例如，[xfce4-appfinder](https://docs.xfce.org/xfce/xfce4-appfinder/start)是一个可访问的启动器。欢迎建议！可能基于GTK的程序会工作得很好。
*   添加一些[`spawn-at-startup`](./Configuration:-Miscellaneous#spawn-at-startup)命令来播放声音，这将向用户表明niri已完成加载。
*   添加`spawn-at-startup "orca"`以在niri启动时自动运行Orca。

## 桌面缩放

还没有内置的缩放功能，但您可以使用第三方工具如[wooz](https://github.com/negrel/wooz)。