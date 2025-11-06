在niri中有几种方法可以使窗口变大：最大化列、将窗口最大化到边缘和全屏窗口。
让我们看看它们的区别。

## 最大化（全宽）列

通过`maximize-column`（默认绑定到<kbd>Mod</kbd><kbd>F</kbd>）最大化列将其宽度扩展以覆盖整个屏幕。
最大化的列仍然为[struts]和[gaps]留出空间，并且可以包含多个窗口。
窗口保留其边框。
这是最简单的大小调整模式，等同于`proportion 1.0`列宽，或`set-column-width "100%"`。

![具有两个窗口的最大化列截图。](./img/maximized-column.png)

您可以通过[`open-maximized true`](./Configuration:-Window-Rules#open-maximized)窗口规则使窗口在最大化列中打开。

## 窗口最大化到边缘

<sup>自：下一个版本</sup>

您可以通过`maximize-window-to-edges`最大化单个窗口。
这与您可以在其他桌面环境和操作系统上找到的最大化相同：它将窗口扩展到可用屏幕区域的边缘。
您仍然会看到您的栏，但看不到struts、gaps或边框。

窗口知道它们最大化到边缘的状态，通常通过将它们的角落变方来响应。
窗口也可以控制最大化到边缘：当您单击窗口标题栏中的方块图标，或双击标题栏时，窗口将请求niri最大化或取消最大化。

您可以将多个最大化窗口放入[标签列](./Tabs)中，但不能放入常规列中。

![最大化到边缘的窗口截图。](./img/window-maximized-to-edges.png)

您可以通过[`open-maximized-to-edges`](./Configuration:-Window-Rules#open-maximized-to-edges)窗口规则使窗口以最大化到边缘的方式打开，或防止窗口在打开时最大化。

## 全屏窗口

窗口可以全屏，通常在视频播放器、演示文稿或游戏中看到。
您还可以通过`fullscreen-window`（默认绑定到<kbd>Mod</kbd><kbd>Shift</kbd><kbd>F</kbd>）强制窗口全屏。
全屏窗口覆盖整个屏幕。
与最大化到边缘类似，窗口知道它们的全屏状态，并可以通过隐藏标题栏或其他UI部分来响应。

Niri在全屏窗口后面渲染纯黑色背景。
当窗口本身仍然太小时，此背景有助于匹配屏幕大小（例如，如果您尝试全屏固定大小的对话窗口），这是[Wayland协议定义的行为](https://wayland.app/protocols/xdg-shell#xdg_toplevel:request:set_fullscreen)。

当全屏窗口聚焦且不动画时，它将覆盖浮动窗口和顶层shell层。
如果您希望例如将layer-shell通知或启动器显示在全屏窗口上方，请配置相应工具将它们放在覆盖层shell层上。

![全屏窗口截图。](./img/fullscreen-window.png)

您可以通过[`open-fullscreen`](./Configuration:-Window-Rules#open-fullscreen)窗口规则使窗口全屏打开，或防止窗口在打开时全屏。

## 全屏和最大化的常见行为

全屏或最大化到边缘的窗口只能在滚动布局中。
因此，如果您尝试全屏或最大化[浮动窗口](./Floating-Windows)，它将移动到滚动布局中。
然后，取消全屏/取消最大化将自动将其带回浮动布局。

得益于可滚动平铺，全屏和最大化窗口仍是布局的正常参与者：您可以从中向左和向右滚动并看到其他窗口。

![显示全屏窗口与其他窗口并排的概览截图。](./img/fullscreen-window-in-overview.png)

全屏和最大化到边缘都是窗口知道并可以控制的特殊状态。
窗口有时希望在打开时恢复它们的全屏或更频繁地恢复最大化状态。
最佳机会是在*初始配置*序列期间，窗口在告诉niri打开窗口之前应该知道的一切。
如果窗口这样做，那么`open-maximized-to-edges`和`open-fullscreen`窗口规则有机会阻止或调整请求。

然而，一些客户端倾向于在初始配置序列*之后*不久请求最大化，此时niri已经向它们发送了初始大小请求（有时甚至在显示在屏幕上之后，导致打开后立即调整大小）。
从niri的角度来看，窗口此时已经打开，因此如果窗口这样做，那么`open-maximized-to-edges`和`open-fullscreen`窗口规则不会做任何事情。

## 窗口化全屏

<sup>自：25.05</sup>

Niri还可以通过`toggle-windowed-fullscreen`操作告诉窗口它处于全屏状态，而实际上不使其全屏。
这通常对屏幕录制基于浏览器的演示文稿很有用，当您想隐藏浏览器UI，但仍希望窗口大小为正常窗口大小。

在窗口化全屏时，您可以使用niri操作来最大化或取消最大化窗口。
窗口端的标题栏最大化按钮和手势可能无法工作，因为窗口将始终认为它处于全屏状态。

另请参见[屏幕录制功能wiki页面](./Screencasting#windowed-fakedetached-fullscreen)上的窗口化全屏。

[struts]: ./Configuration:-Layout.md#struts

[gaps]: ./Configuration:-Layout.md#gaps