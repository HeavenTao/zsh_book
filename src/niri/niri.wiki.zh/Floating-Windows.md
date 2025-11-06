### 概述

<sup>自：25.01</sup>

niri中的浮动窗口始终显示在平铺窗口的顶部。
浮动布局不会滚动。
每个工作区/显示器都有自己的浮动布局，就像每个工作区/显示器都有自己的平铺布局一样。

新窗口如果具有父窗口（例如对话框）或固定大小（例如启动屏幕），将自动浮动。
要在浮动和平铺之间切换窗口，您可以使用`toggle-window-floating`绑定，或在拖拽/移动窗口时右键单击。
您还可以使用`open-floating true/false`窗口规则来强制窗口以浮动方式打开，或禁用自动浮动逻辑。

使用`switch-focus-between-floating-and-tiling`在两种布局之间切换焦点。
当聚焦在浮动布局上时，绑定（如`focus-column-right`）将在浮动窗口上操作。

您可以使用类似`niri msg action move-floating-window -x 100 -y 200`的命令精确放置浮动窗口。