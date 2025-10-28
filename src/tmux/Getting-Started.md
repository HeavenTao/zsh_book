## 入门指南

### 关于 tmux

tmux 是一个在终端中运行的程序，它允许在其中运行多个其他终端程序。tmux 内部的每个程序都有自己的终端，由 tmux 管理，可以从运行 tmux 的单个终端访问 - 这称为复用，tmux 是一个终端复用器。

tmux 及其内部运行的任何程序都可以从运行它的终端（外部终端）分离，稍后重新附加到相同或另一个终端。

在 tmux 内部运行的程序可以是全屏交互式程序，如 *vi(1)* 或 *top(1)*，shell 如 *bash(1)* 或 *ksh(1)*，或任何可以在 Unix 终端中运行的其他程序。

tmux 拥有一套强大的功能集，可以交互式地或通过脚本访问、管理和组织内部的程序。

tmux 的主要用途是：

* 通过在 tmux 内部运行程序来保护远程服务器上运行的程序免受连接中断的影响。

* 允许从多个不同的本地计算机访问在远程服务器上运行的程序。

* 在一个终端中同时处理多个程序和 shell，有点像窗口管理器。

例如：

* 用户从工作计算机上的 *xterm(1)* 使用 *ssh(1)* 连接到远程服务器并运行几个程序，也许是一个编辑器、一个编译器和几个 shell。

* 他们以交互方式处理这些程序，也许开始编译，然后关闭带有 tmux 的 *xterm(1)* 并回家。

* 然后他们能够从家里连接到同一个远程服务器，附加到 tmux，并从他们之前的地方继续。

这是 tmux 在 *xterm(1)* 中显示 shell 的屏幕截图：

<p align="center"><img src="../images/tmux_default.png" width=368 height=235></p>

### 关于本文档

本文档概述了 tmux 的一些关键概念，描述了如何交互式地使用主要功能，并提供了一些关于基本自定义和配置的信息。

请注意，本文档可能会提到仅在最新 tmux 版本中可用的功能。仅支持最新的 tmux 版本。大约每六个月发布一次新版本。

tmux 可以从大多数主要平台的包管理系统安装。有关如何安装 tmux 或如何从源代码构建它的说明，请参见 [此文档](Installing)。

### 其他文档和帮助

以下是查找有关 tmux 的文档和帮助的几个地方：

- <img src="../images/man_tmux.png" align="right" width=376 height=243>[手册页](https://man.openbsd.org/tmux) 提供了关于 tmux 的详细参考文档，以及每个命令、标志和选项的描述。一旦安装了 tmux，它在第 1 节中也可用：

  ~~~~
  $ man 1 tmux
  ~~~~

- [常见问题解答 (FAQ)](FAQ) 提供了常见问题的解决方案，主要涉及特定的配置问题。

- [tmux-users@googlegroups.com 邮件列表](mailto:tmux-users@googlegroups.com)。

### 基本概念

tmux 有一套基本概念和术语，熟悉这些很重要。本节描述了 tmux 内部的终端是如何分组的以及 tmux 使用的各种术语。

#### tmux 服务器和客户端

tmux 将其所有状态保存在一个名为 tmux 服务器的主进程中。该进程在后台运行，管理所有在 tmux 内部运行的程序并跟踪它们的输出。当用户运行 tmux 命令时，tmux 服务器会自动启动，默认情况下，当没有正在运行的程序时，它会退出。

用户通过启动客户端来连接到 tmux 服务器。这会接管运行它的终端，并使用 `/tmp` 中的套接字文件与服务器通信。每个客户端在一个终端中运行，该终端可以是 *X(7)* 终端，如 *xterm(1)*、系统控制台，或另一个程序（如 tmux 本身）中的终端。每个客户端由其启动的外部终端的名称标识，例如 `/dev/ttypf`。

#### 会话、窗口和窗格

<p><img src="../images/tmux_with_panes.png" align="right" width=376 height=243>
tmux 内部的每个终端都属于一个窗格，这是一个显示 tmux 内部终端内容的矩形区域。因为 tmux 内部的每个终端只显示在一个窗格中，所以窗格一词可以指整个窗格、终端以及其中运行的程序。右侧的屏幕截图显示了带有窗格的 tmux。</p>

每个窗格出现在一个窗口中。窗口由一个或多个窗格组成，这些窗格共同覆盖其整个区域 - 因此可以同时看到多个窗格。窗口通常占据 tmux 附加的整个终端，但它可以更大或更小。窗口中所有窗格的大小和位置称为窗口布局。

每个窗口都有一个名称 - 默认情况下，tmux 会选择一个名称，但用户可以更改它。窗口名称不必唯一，窗口通常由会话和窗口索引而不是名称来标识。

<p><img src="../images/tmux_pane_diagram.png" align="right" width=418 height=285>
每个窗格通过一条线与周围的窗格分隔开，这称为窗格边框。每个窗口中有一个窗格称为活动窗格，这是任何输入文本发送到的地方，也是用于针对窗口的命令的默认窗格。活动窗格的边框标记为绿色，或者如果只有两个窗格，则边框的顶部、底部、左侧或右侧的一半是绿色。</p>

多个窗口被分组成会话。如果一个窗口是会话的一部分，则称其链接到该会话。窗口可以同时链接到多个会话，尽管大多数情况下它们只在一个会话中。会话中的每个窗口都有一个数字，称为窗口索引 - 同一个窗口在不同会话中可能链接到不同的索引。会话的窗口列表是按索引顺序链接到该会话的所有窗口。

每个会话都有一个当前窗口，这是会话附加时显示的窗口，也是针对会话的任何命令的默认窗口。如果当前窗口发生更改，则以前的当前窗口将成为上一个窗口。

一个会话可以附加到一个或多个客户端，这意味着它显示在运行该客户端的外部终端上。在该外部终端中键入的任何文本都会发送到附加会话的当前窗口中的活动窗格。会话没有索引，但它们有一个名称，该名称必须是唯一的。

总结：

* 程序在窗格中的终端中运行，每个窗格属于一个窗口。
* 每个窗口都有一个名称和一个活动窗格。
* 窗口链接到一个或多个会话。
* 每个会话都有一个窗口列表，每个窗口都有一个索引。
* 会话中的一个窗口是当前窗口。
* 会话附加到一个或多个客户端，或分离（不附加到任何客户端）。
* 每个客户端附加到一个会话。

#### 术语摘要

术语|描述
---|---
客户端|从外部终端（如 *xterm(1)*）附加 tmux 会话
会话|将一个或多个窗口分组在一起
窗口|将一个或多个窗格分组在一起，链接到一个或多个会话
窗格|包含终端和正在运行的程序，出现在一个窗口中
活动窗格|当前窗口中发送键入内容的窗格；每个窗口一个
当前窗口|附加会话中发送键入内容的窗口；每个会话一个
上一个窗口|之前的当前窗口
会话名称|会话的名称，默认为从零开始的数字
窗口列表|按数字顺序排列的会话中的窗口列表
窗口名称|窗口的名称，默认为活动窗格中运行的程序的名称
窗口索引|会话窗口列表中的窗口编号
窗口布局|窗口中窗格的大小和位置

### 交互式使用 tmux

#### 创建会话

要创建第一个 tmux 会话，从 shell 运行 tmux。使用 `new-session` 命令创建新会话 - 简写为 `new`：

~~~~
$ tmux new
~~~~

不带参数时，`new-session` 会创建一个新会话并附加它。因为这是第一个会话，所以 tmux 服务器会启动，从 shell 运行的 tmux 成为第一个客户端并附加到它。

新会话将有一个窗口（索引为 0），其中包含一个运行 shell 的窗格。shell 提示符应出现在终端顶部，绿色状态行在底部（有关状态行的更多信息见下文）。

默认情况下，第一个会话将被称为 `0`，第二个 `1`，依此类推。`new-session` 允许使用 `-s` 标志为会话指定名称：

~~~~
$ tmux new -smysession
~~~~

这将创建一个名为 `mysession` 的新会话。可以通过传递额外参数来运行命令而不是 shell。如果给出一个参数，tmux 会将其传递给 shell，如果给出多个参数，则直接运行命令。例如，这些命令运行 *emacs(1)*：

~~~~
$ tmux new 'emacs ~/.tmux.conf'
~~~~

或：

~~~~
$ tmux new -- emacs ~/.tmux.conf
~~~~

默认情况下，tmux 会根据窗口中运行的内容来命名会话中的第一个窗口。`-n` 标志可以指定替代名称，在这种情况下，是一个运行 *top(1)* 的窗口 `mytopwindow`：

~~~~
$ tmux new -nmytopwindow top
~~~~

`new-session` 还有其他标志 - 下面会介绍一些。完整列表见 [tmux 手册](https://man.openbsd.org/tmux#new-session)。

#### 状态行

当 tmux 客户端附加时，它会在屏幕的最底行显示状态行。默认情况下，这是绿色的，显示：

* 在左侧，附加会话的名称：`[0]`。

* 在中间，会话中的窗口列表及其索引，例如索引为 0 的窗口 `ksh`：`0:ksh`。

* 在右侧，窗格标题（用引号括起来）（默认为运行 tmux 的主机名）以及时间和日期。

<p align="center"><img src="../images/tmux_status_line_diagram.png" width=613 height=204></p>

随着新窗口的打开，窗口列表会增长 - 如果窗口太多而无法适应终端的宽度，则会在左侧或右侧或两者都添加 `<` 或 `>` 以显示有隐藏的窗口。

在窗口列表中，当前窗口在名称后标记为 `*`，上一个窗口标记为 `-`。

#### 前缀键

一旦 tmux 客户端附加，输入的任何键都会转发到当前窗口活动窗格中运行的程序。对于控制 tmux 本身的键，必须先按下一个特殊键 - 这称为前缀键。

默认的前缀键是 `C-b`，表示 `Ctrl` 键和 `b`。在 tmux 中，修饰键通过在键前加上 `C-` 表示控制键，`M-` 表示元键（在现代计算机上通常是 `Alt`），`S-` 表示移位键。这些可以组合在一起，因此 `C-M-x` 表示同时按下控制键、元键和 `x`。

按下前缀键时，tmux 会等待另一个按键按下，这决定了执行哪个 tmux 命令。这样的键在这里显示为用空格分隔：`C-b c` 表示先按下前缀键 `C-b`，然后释放，再按下 `c` 键。如果需要，在按下 `C-b` 后必须释放 `Ctrl` 键 - `C-b c` 与 `C-b C-c` 不同。

按下 `C-b` 两次会将 `C-b` 键发送到活动窗格中运行的程序。

#### 帮助键

每个默认的 tmux 键绑定都有一个简短的描述，帮助记住该键的作用。通过按下 `C-b ?` 可以看到所有键及其相应描述文本的列表。

<img src="../images/tmux_list_keys.png" align="right" width=376 height=243>

`C-b ?` 进入视图模式以显示文本。处于视图模式的窗格有自己的键绑定，不需要前缀键。这些大致遵循 *emacs(1)*。最重要的是 `Up`、`Down`、`C-Up`、`C-Down` 用于上下滚动，以及 `q` 用于退出模式。可见区域顶部行号和总行数显示在右上角。

或者，可以通过运行以下命令从 shell 中看到相同的列表：

~~~~
$ tmux lsk -N|more
~~~~

`C-b /` 显示单个键的描述 - 终端底部会出现提示。按下键将在同一位置显示其描述。例如，按下 `C-b /` 然后 `?` 显示：

~~~~
C-b ? List key bindings
~~~~

#### 命令和标志

tmux 有一大套命令。这些命令都有一个名称，如 `new-window`、`new-session` 或 `list-keys`，许多命令还有较短的别名，如 `neww`、`new` 或 `lsk`。

每次使用键绑定时，它都会运行一个或多个 tmux 命令。例如，`C-b c` 运行 `new-window` 命令。

命令也可以从 shell 中使用，如上面的 `new-session` 和 `list-keys`。

每个命令有零个或多个标志，与标准 Unix 命令相同。标志可能需要或不需要单个参数。此外，命令在标志之后可以接受其他参数。标志在命令之后传递，例如运行 `new-session` 命令（别名 `new`）并带有 `-d` 和 `-n` 标志：

~~~~
$ tmux new-session -d -nmysession
~~~~

所有命令及其标志都在 tmux 手册页中有文档。

本文档重点介绍可用的键绑定，但也会提及命令或有用的标志。它们可以从 shell 或命令提示符中输入，如下一节所述。

#### 命令提示符

<img src="../images/tmux_command_prompt.png" align="right" width=376 height=243>

tmux 有一个交互式命令提示符。可以通过按下 `C-b :` 打开，它会替代状态行显示，如屏幕截图所示。

在提示符处，可以像在 shell 中一样输入命令。输出将在状态行中显示一小段时间，或将活动窗格切换到视图模式。

默认情况下，命令提示符使用类似于 *emacs(1)* 的键；但是，如果 `VISUAL` 或 `EDITOR` 环境变量设置为包含 `vi` 的内容（如 `vi`、`vim` 或 `nvi`），则使用 *vi(1)* 风格的键。

可以使用分号（`;`）在命令提示符处一起输入多个命令。这称为命令序列。

#### 附加和分离

从 tmux 分离意味着客户端退出并从外部终端分离，返回到 shell，并让 tmux 会话及其内部的任何程序在后台继续运行。要分离 tmux，请使用 `C-b d` 键绑定。当 tmux 分离时，它会打印一条包含会话名称的消息：

~~~~
[detached (from session mysession)]
~~~~

`attach-session` 命令附加到现有会话。不带参数时，它将附加到最近使用且尚未附加的会话：

~~~~
$ tmux attach
~~~~

或者 `-t` 给出要附加到的会话名称：

~~~~
$ tmux attach -tmysession
~~~~

默认情况下，附加到会话不会分离附加到同一会话的其他客户端。`-d` 标志会这样做：

~~~~
$ tmux attach -dtmysession
~~~~

`new-session` 命令有一个 `-A` 标志，如果会话存在则附加到现有会话，如果不存在则创建一个新会话。对于名为 `mysession` 的会话：

~~~~
$ tmux new -Asmysession
~~~~

可以添加 `-D` 标志，使 `new-session` 也像使用 `-d` 的 `attach-session` 一样工作，并分离附加到会话的其他客户端。

#### 列出会话

`list-sessions` 命令（别名 `ls`）显示可附加的会话列表。这显示了四个会话，分别名为 `1`、`2`、`myothersession` 和 `mysession`：

~~~~
$ tmux ls
1: 3 windows (created Sat Feb 22 11:44:51 2020)
2: 1 windows (created Sat Feb 22 11:44:51 2020)
myothersession: 2 windows (created Sat Feb 22 11:44:51 2020)
mysession: 1 windows (created Sat Feb 22 11:44:51 2020)
~~~~

#### 完全杀死 tmux

如果 tmux 内部没有会话、窗口或窗格，服务器将退出。
也可以使用 `kill-server` 命令完全杀死它。例如，在命令提示符下：

~~~~
:kill-server
~~~~

#### 创建新窗口

<img src="../images/tmux_new_windows.png" align="right" width=376 height=243>

在附加的会话中，可以通过 `C-b c` 键绑定创建一个新窗口，该键绑定运行 `new-window` 命令。新窗口在第一个可用索引处创建 - 因此第二个窗口将具有索引 1。新窗口成为会话的当前窗口。

如果窗口列表中有任何间隙，它们将被新窗口填充。因此，如果索引为 0 和 2 的窗口存在，则下一个新窗口将在索引 1 处创建。

`new-window` 命令有一些有用的标志，可以在命令提示符下使用：

* `-d` 标志创建窗口，但不使其成为当前窗口。

* `-n` 允许为新窗口指定名称。例如，使用命令提示符创建一个名为 `mynewwindow` 的窗口而不使其成为当前窗口：

  ~~~~
  :neww -dnmynewwindow
  ~~~~

* `-t` 标志指定窗口的目标。命令目标有特殊的语法，但对于 `new-window` 的简单使用，只需给出窗口索引即可。这将在索引 999 处创建一个窗口：

  ~~~~
  :neww -t999
  ~~~~

可以像 `new-session` 一样向 `new-window` 提供要在新窗口中运行的命令。例如，创建一个运行 *top(1)* 的新窗口：

~~~~
:neww top
~~~~

#### 分割窗口

<img src="../images/tmux_split_h.png" align="right" width=376 height=243>

通过分割窗口创建窗格。这是通过 `split-window` 命令完成的，默认情况下绑定到两个键：

* `C-b %` 将当前窗格水平分割成两个，产生两个相邻的窗格，一个在左，一个在右。

* `C-b "` 将当前窗格垂直分割成两个，产生两个上下排列的窗格。

每次将窗格分割成两个时，可以使用相同的键绑定再次分割这些窗格，直到窗格变得太小。

<img src="../images/tmux_split_v.png" align="right" width=376 height=243>

`split-window` 有几个有用的标志：

* `-h` 进行水平分割，`-v` 进行垂直分割。

* `-d` 不将活动窗格更改为新创建的窗格。

* `-f` 使新窗格跨越窗口的整个宽度或高度，而不是受被分割窗格大小的限制。

* `-b` 将新窗格放置在被分割窗格的左侧或上方，而不是右侧或下方。

可以像 `new-session` 和 `new-window` 一样向 `split-window` 提供要在新窗格中运行的命令。

#### 更改当前窗口

有几种键绑定可以更改会话的当前窗口：

* `C-b 0` 更改为窗口 0，`C-b 1` 更改为窗口 1，依此类推，直到窗口 `C-b 9`。

* `C-b '` 提示输入窗口索引并更改为该窗口。

* `C-b n` 按数字更改为窗口列表中的下一个窗口。因此，在窗口 1 中按下 `C-b n` 将更改为窗口 2（如果存在）。

* `C-b p` 按数字更改为窗口列表中的上一个窗口。

* `C-b l` 更改为上一个窗口，即当前窗口之前的窗口。

这些都是 `select-window` 命令的变体。

#### 更改活动窗格

可以使用以下键绑定在窗口中的窗格之间更改活动窗格：

* `C-b Up`、`C-b Down`、`C-b Left` 和 `C-b Right` 分别更改为活动窗格上方、下方、左侧或右侧的窗格。这些键在窗口中循环，因此在底部的窗格上按下 `C-b Down` 将更改为顶部的窗格。

<img src="../images/tmux_display_panes.png" align="right" width=368 height=235>

* `C-b q` 在窗格上短暂显示窗格编号和大小。在它们消失之前按下其中一个数字键会将活动窗格更改为所选窗格，因此 `C-b q 1` 将更改为窗格编号 1。

* `C-b o` 按窗格编号移动到下一个窗格，`C-b C-o` 将该窗格与活动窗格交换，因此它们在窗口中的位置和大小会交换。

这些使用 `select-pane` 和 `display-panes` 命令。

窗格编号不是固定的，而是按窗口中的位置编号，因此如果编号为 0 的窗格与编号为 1 的窗格交换，则编号也会交换。

#### 选择会话、窗口和窗格

<p><img src="../images/tmux_choose_tree1.png" align="right" width=376 height=243>
tmux 包含一种模式，可以从树中选择会话、窗口或窗格，这称为树模式。它可以用于浏览会话、窗口和窗格；更改附加的会话、当前窗口或活动窗格；杀死会话、窗口和窗格；或通过标记它们一次应用命令到多个。</p>

有两种键绑定可以进入树模式：`C-b s` 仅显示会话并选择附加的会话；`C-b w` 以会话展开的方式开始，显示窗口并选择附加会话中的当前窗口。

树模式将窗口分为两个部分：上半部分是会话、窗口和窗格的树，下半部分是每个窗格中光标周围区域的预览。对于会话，预览显示尽可能多的窗口中的活动窗格；对于窗口，显示尽可能多的窗格；对于窗格，仅显示所选窗格。

<img src="../images/tmux_choose_tree2.png" align="right" width=376 height=243>

控制树模式的键不需要前缀。可以使用 `Up` 和 `Down` 键导航列表。`Enter` 更改选定的项目（它成为附加的会话、当前窗口或活动窗格）并退出模式。`Right` 展开项目（如果可能）- 会话展开以显示其窗口，窗口展开以显示其窗格。`Left` 折叠项目以隐藏任何窗口或窗格。`O` 更改项目的顺序，`q` 退出树模式。

通过按 `t` 标记项目，再次按 `t` 取消标记。标记的项目以粗体显示，并在名称后带有 `*`。所有标记的项目可以通过按 `T` 取消标记。标记的项目可以通过按 `X` 一起杀死，或通过按 `:` 提示将命令应用到所有标记的项目。

树中的每个项目在行首的括号中有快捷键。按下此键将立即选择该项目（就像已选择并按下 `Enter` 一样）。前十个项目的键是 `0` 到 `9`，之后使用键 `M-a` 到 `M-z`。

这是在树模式下可用的键列表，无需按下前缀键：

键|功能
---|---
`Enter`|更改附加的会话、当前窗口或活动窗格
`Up`|选择上一个项目
`Down`|选择下一个项目
`Right`|展开项目
`Left`|折叠项目
`x`|杀死选定的项目
`X`|杀死标记的项目
`<`|向左滚动预览
`>`|向右滚动预览
`C-s`|按名称搜索
`n`|重复上次搜索
`t`|切换项目是否被标记
`T`|取消标记所有项目
`C-t`|标记所有项目
`:`|提示输入要为选定项目或每个标记项目运行的命令
`O`|更改排序字段
`r`|反转排序顺序
`v`|切换预览
`q`|退出树模式

树模式通过 `choose-tree` 命令激活。

#### 分离其他客户端

<img src="../images/tmux_choose_client.png" align="right" width=376 height=243>

通过按下 `C-b D`（即 `C-b S-d`）可以获得客户端列表。这类似于树模式，称为客户端模式。

每个客户端在列表的上半部分显示其名称、附加的会话、大小以及上次使用的时间和日期；下半部分显示所选客户端的状态行预览。

移动和标记键与树模式相同，但其他键不同，例如 `Enter` 键分离选定的客户端。

这是客户端模式中不包括移动和标记键的键列表：

键|功能
---|---
`Enter`|分离选定的客户端
`d`|分离选定的客户端，与 `Enter` 相同
`D`|分离标记的客户端
`x`|分离选定的客户端并尝试杀死启动它的 shell
`X`|分离标记的客户端并尝试杀死启动它们的 shell

除了使用客户端模式，`detach-client` 命令有一个 `-a` 标志，用于分离除附加客户端之外的所有客户端。

#### 杀死会话、窗口或窗格

按下 `C-b &` 会提示确认，然后杀死（关闭）当前窗口。窗口中的所有窗格同时被杀死。`C-b x` 只杀死活动窗格。这些绑定到 `kill-window` 和 `kill-pane` 命令。

`kill-session` 命令杀死附加的会话及其所有窗口并分离客户端。`kill-session` 没有键绑定，但可以从命令提示符或树模式中的 `:` 提示符使用。

#### 重命名会话和窗口

<img src="../images/tmux_rename_session.png" align="right" width=368 height=235>

`C-b $` 会提示输入附加会话的新名称。这使用 `rename-session` 命令。同样，`C-b ,` 会提示输入当前窗口的新名称，使用 `rename-window` 命令。

#### 交换和移动

tmux 允许使用 `swap-pane` 和 `swap-window` 命令交换窗格和窗口。

为了方便交换，可以标记单个窗格。所有会话中有一个标记的窗格。`C-b m` 键绑定切换当前会话中当前窗口的活动窗格是否为标记的窗格。`C-b M` 完全清除标记的窗格，这样就没有窗格被标记。标记的窗格通过其边框的绿色背景显示，包含标记窗格的窗口在状态行中有一个 `M` 标志。

<img src="../images/tmux_marked_pane.png" align="right" width=368 height=235>

一旦窗格被标记，可以使用 `swap-pane` 命令将其与当前窗口的活动窗格交换，或者使用 `swap-window` 命令将包含标记窗格的窗口与当前窗口交换。例如，使用命令提示符：

~~~~
:swap-pane
~~~~

窗格还可以使用 `C-b {` 和 `C-b }` 键绑定与上方或下方的窗格交换。

移动窗口使用 `move-window` 命令或 `C-b .` 键绑定。
按下 `C-b .` 将提示输入当前窗口的新索引。如果给定索引处已存在窗口，将显示错误。可以通过使用 `-k` 标志替换现有窗口 - 将窗口移动到索引 999：

~~~~
:move-window -kt999
~~~~

如果窗口列表中有间隙，可以通过 `move-window` 的 `-r` 标志重新编号。例如，这将把窗口列表 0, 1, 3, 999 更改为 0, 1, 2, 3：

~~~~
:movew -r
~~~~

#### 调整窗格大小和缩放

窗格可以通过 `C-b C-Left`、`C-b C-Right`、`C-b C-Up` 和 `C-b C-Down` 以小步调整大小，通过 `C-b M-Left`、`C-b M-Right`、`C-b M-Up` 和 `C-b M-Down` 以大步调整大小。这些使用 `resize-pane` 命令。

单个窗格可以通过 `C-b z` 暂时占据整个窗口，隐藏任何其他窗格。再次按下 `C-b z` 会使窗格和窗口布局恢复到原来的状态。这称为缩放和取消缩放。窗格被缩放的窗口在状态行中标记为 `Z`。更改窗口中窗格大小或位置的命令会自动取消窗口的缩放。

#### 窗口布局

<img src="../images/tmux_tiled.png" align="right" width=368 height=235>

窗口中的窗格可以自动排列成几种命名布局之一，这些布局可以通过 `C-b Space` 键绑定在之间轮换，或直接通过 `C-b M-1`、`C-b M-2` 等选择。

可用的布局有：

名称|键|描述
---|---|---
even-horizontal|`C-b M-1`|均匀分布
even-vertical|`C-b M-2`|均匀上下分布
main-horizontal|`C-b M-3`|顶部一个大窗格，其余均匀分布
main-vertical|`C-b M-4`|左侧一个大窗格，其余均匀上下分布
tiled|`C-b M-5`|按相同行数和列数平铺

#### 复制和粘贴

tmux 有自己的复制和粘贴系统。复制的文本片段称为粘贴缓冲区。使用复制模式（通过 `C-b [` 进入）复制文本，最近复制的文本通过 `C-b ]` 粘贴到活动窗格中。

<img src="../images/tmux_copy_mode.png" align="right" width=368 height=235>

粘贴缓冲区可以命名，但默认情况下它们由 tmux 分配名称，如 `buffer0` 或 `buffer1`。这样的缓冲区称为自动缓冲区，最多保留 50 个 - 一旦有 50 个缓冲区，添加新缓冲区时会删除最旧的。如果缓冲区被命名，则称为命名缓冲区；命名缓冲区无论有多少都不会被删除。

可以配置 tmux 将任何复制的文本发送到系统剪贴板：[此文档](Clipboard) 解释了配置的不同方式。

复制模式冻结窗格中的任何输出并允许复制文本。视图模式（前面描述过）是复制模式的只读形式。

与命令提示符类似，复制模式使用类似于 *emacs(1)* 的键；但是，如果 `VISUAL` 或 `EDITOR` 环境变量设置为包含 `vi` 的内容（如 `vi`、`vim` 或 `nvi`），则使用 *vi(1)* 风格的键。以下是一些在 *emacs(1)* 键的复制模式下可用的键：

键|操作
---|---
`Up`, `Down`, `Left`, `Right`|移动光标
`C-Space`|开始选择
`C-w`|复制选择并退出复制模式
`q`|退出复制模式
`C-g`|停止选择而不复制，或停止搜索
`C-a`|将光标移动到行首
`C-e`|将光标移动到行尾
`C-r`|向后交互式搜索
`M-f`|将光标移动到下一个单词
`M-b`|将光标移动到上一个单词

*vi(1)* 和 *emacs(1)* 的完整键列表在 [手册页](https://man.openbsd.org/tmux#WINDOWS_AND_PANES) 中可用。以下是一些精选内容：

<img src="../images/tmux_buffer_mode.png" align="right" width=368 height=235>

复制一些文本后，可以使用 `C-b ]` 粘贴最近的文本，或通过使用缓冲区模式（通过 `C-b =` 进入）粘贴较旧的缓冲区。缓冲区模式类似于客户端模式和树模式，提供缓冲区列表及其内容的预览。除了树模式和客户端模式中使用的导航和标记键外，缓冲区模式还支持以下键：

键|功能
---|---
`Enter`|粘贴选定的缓冲区
`p`|粘贴选定的缓冲区，与 `Enter` 相同
`P`|粘贴标记的缓冲区
`d`|删除选定的缓冲区
`D`|删除标记的缓冲区

可以使用 `set-buffer` 命令重命名缓冲区。`-b` 标志给出现有的缓冲区名称，`-n` 给出新名称。这会将其转换为命名缓冲区。例如，从命令提示符重命名 `buffer0` 为 `mybuffer`：

~~~~
:setb -bbuffer0 -nmybuffer
~~~~

`set-buffer` 也可以用于创建缓冲区。要创建一个名为 `foo` 且文本为 `bar` 的缓冲区：

~~~~
:setb -bfoo bar
~~~~

`load-buffer` 将从文件加载缓冲区：

~~~~
:loadb -bbuffername ~/a/file
~~~~

不带 `-b` 的 `set-buffer` 或 `load-buffer` 会创建一个自动缓冲区。

可以使用 `save-buffer` 将现有缓冲区保存到文件：

~~~~
:saveb -bbuffer0 ~/saved_buffer
~~~~

#### 查找窗口和窗格

<img src="../images/tmux_find_window.png" align="right" width=368 height=235>

`C-b f` 会提示输入一些文本，然后进入树模式，并使用过滤器显示仅包含该文本在可见内容或窗格标题或窗口名称中出现的窗格。如果找到窗格，则树中仅显示这些窗格，并在预览上方显示文本 `filter: active`。如果未找到窗格，则树中显示所有窗格，并在预览上方显示文本 `filter: no matches`。

#### 使用鼠标

tmux 对鼠标有丰富的支持。它可以用于更改活动窗格或窗口、调整窗格大小、复制文本或从菜单中选择项目。

鼠标支持通过 `mouse` 选项启用；选项和配置文件在下一节中有详细描述。要从命令提示符启用鼠标，请使用 `set-option` 命令：

~~~~
:set -g mouse on
~~~~

启用鼠标后：

<img src="../images/tmux_pane_menu.png" align="right" width=376 height=243>

* 在窗格上按下左键将使该窗格成为活动窗格。

* 在状态行上的窗口名称上按下左键将使其成为当前窗口。

* 在窗格边框上拖动左键可以调整窗格大小。

* 在窗格内拖动左键可以选择文本；释放鼠标时复制选定的文本。

* 在窗格上按下右键会打开一个包含各种命令的菜单。释放鼠标按钮时，选定的命令会以窗格为目标运行。每个菜单项在括号中显示一个键快捷方式。

* 在窗口或状态行上的会话名称上按下右键会为窗口或会话打开一个类似的菜单。

### 配置 tmux

#### 配置文件

当 tmux 服务器启动时，tmux 会运行用户主目录中的 `.tmux.conf` 文件。此文件包含按顺序执行的 tmux 命令列表。需要注意的是，`.tmux.conf` *仅*在服务器启动时运行，而不是在创建新会话时运行。

可以使用 `source-file` 命令从 `.tmux.conf` 或正在运行的 tmux 服务器运行不同的配置文件，例如，要从正在运行的服务器使用命令提示符再次运行 `.tmux.conf`：

~~~~
:source ~/.tmux.conf
~~~~

配置文件中的命令每行一个。以 `#` 开头的行是注释，会被忽略：

~~~~
# 这是一个注释 - 下面的命令关闭状态行
set -g status off
~~~~

配置文件中的行以类似于 shell 的方式处理，例如：

- 参数可以用 `'` 或 `"` 括起来以包含空格，或者可以转义空格。这四行做同样的事情：
  ~~~~
  set -g status-left "hello word"
  set -g status-left "hello\ word"
  set -g status-left 'hello word'
  set -g status-left hello\ word
  ~~~~

- 但在 `'` 内部不会发生转义。这里的字符串是 `hello\ world` 而不是 `hello world`：
  ~~~~
  set -g status-left 'hello\ word'
  ~~~~

- `~` 展开为主目录（除了在 `'` 内部）：
  ~~~~
  source ~/myfile
  ~~~~

- 可以设置环境变量并且也会展开（但在 `'` 内部不会）：
  ~~~~
  MYFILE=myfile
  source "~/$MYFILE"
  ~~~~
  在配置文件中设置的任何变量都将传递给在 tmux 内部创建的新窗格。

- 一些特殊字符如 `\n`（换行符）和 `\t`（制表符）会被替换。字面量 `\` 必须写成 `\\`。

尽管 tmux 配置文件具有一些与 shell 类似的功能，但它们不是 shell 脚本，不能使用 shell 构造如 `$()`。

#### 键绑定

使用 `bind-key` 和 `unbind-key` 命令更改 tmux 键绑定。tmux 中的每个键绑定都属于一个命名的键表。有四个默认键表：

* `root` 表包含不带前缀键按下的键绑定。

* `prefix` 表包含在前缀键之后按下的键绑定，如本文档中提到的那些。

* `copy-mode` 表包含在复制模式中使用 *emacs(1)* 风格键的键绑定。

* `copy-mode-vi` 表包含在复制模式中使用 *vi(1)* 风格键的键绑定。

可以使用 `list-keys` 命令列出所有键绑定或单个表的键绑定。默认情况下，这会将键显示为一系列 `bind-key` 命令。`-T` 标志给出要显示的键表，`-N` 标志显示键帮助，如 `C-b ?` 键绑定。

例如，仅列出 `prefix` 表中的键：

~~~~
$ tmux lsk -Tprefix
bind-key    -T prefix C-b     send-prefix
bind-key    -T prefix C-o     rotate-window
...
~~~~

或：

~~~~
$ tmux lsk -Tprefix -N
C-b     Send the prefix key
C-o     Rotate through the panes
...
~~~~

`bind-key` 命令可以用于设置键绑定，无论是交互式还是最常见的是从配置文件中。与 `list-keys` 一样，`bind-key` 有一个 `-T` 标志用于指定要使用的键表。如果未给出 `-T`，则键被放入 `prefix` 表；`-n` 标志是使用 `root` 表的 `-Troot` 的简写。

例如，`list-keys` 命令显示 `C-b 9` 使用 `select-window` 命令更改为窗口 9：

~~~~
$ tmux lsk -Tprefix 9
bind-key -T prefix 9 select-window -t :=9
~~~~

可以添加类似的键绑定，使 `C-b M-0` 更改为窗口 10：

~~~~
bind M-0 selectw -t:=10
~~~~

`select-window` 的 `-t` 标志指定目标窗口。在此示例中，`:` 表示目标是窗口，`=` 表示名称必须完全匹配 `10`。目标在 [手册页的 COMMANDS 部分](https://man.openbsd.org/tmux#COMMANDS) 中有进一步说明。

`unbind-key` 命令移除键绑定。与 `bind-key` 一样，它有 `-T` 和 `-n` 标志用于键表。在重新绑定键之前不必移除键绑定，`bind-key` 会替换任何现有的键绑定。`unbind-key` 仅在需要完全移除键绑定时才需要：

~~~~
unbind M-0
~~~~

#### 复制模式键绑定

复制模式键绑定在 `copy-mode` 和 `copy-mode-vi` 键表中设置。复制模式有一组单独的命令，这些命令通过 `-X` 标志传递给 `send-keys` 命令，例如复制模式的 `start-of-line` 命令将光标移动到行首，并在 `copy-mode` 键表中绑定到 `C-a`：

~~~~
$ tmux lsk -Tcopy-mode C-a
bind-key -T copy-mode C-a send-keys -X start-of-line
~~~~

复制模式命令的完整列表在 [手册页](https://man.openbsd.org/tmux#WINDOWS_AND_PANES) 中可用。以下是一些精选内容：

命令|*emacs(1)*|*vi(1)*|描述
---|---|---|---
begin-selection|C-Space|Space|开始选择
cancel|q|q|退出复制模式
clear-selection|C-g|Escape|清除选择
copy-pipe|||复制并传送到第一个参数中的命令
copy-selection-and-cancel|M-w|Enter|复制选择并退出复制模式
cursor-down|Down|j|向下移动光标
cursor-left|Left|h|向左移动光标
cursor-right|Right|l|向右移动光标
cursor-up|Up|k|向上移动光标
end-of-line|C-e|$|将光标移动到行尾
history-bottom|M->|G|移动到历史记录底部
history-top|M-<|g|移动到历史记录顶部
middle-line|M-r|M|移动到中间行
next-word-end|M-f|e|移动到下一个单词的末尾
page-down|PageDown|C-f|向下翻页
page-up|PageUp|C-b|向上翻页
previous-word|M-b|b|移动到上一个单词
rectangle-toggle|R|v|切换矩形选择
search-again|n|n|重复上次搜索
search-backward||?|向后搜索，第一个参数是搜索词
search-backward-incremental|C-r||增量向后搜索，通常与 `command-prompt` 的 `-i` 标志一起使用
search-forward||/|向前搜索，第一个参数是搜索词
search-forward-incremental|C-s||增量向前搜索
search-reverse|N|N|重复上次搜索但方向相反
start-of-line|C-a|0|移动到行首

#### 选项类型

通过设置选项来配置 tmux。有几种类型的选项：

* 服务器选项，影响整个服务器。

* 会话选项，影响一个或所有会话。

* 窗口选项，影响一个或所有窗口。

* 窗格选项，影响一个或所有窗格。

* 用户选项，tmux 不使用但保留给用户。

会话和窗口选项都有全局选项集和每个会话或窗口的选项集。如果会话或窗口集中不存在该选项，则使用全局选项。窗格选项类似，但也会检查窗口选项。

配置 tmux 时，最常见的是设置服务器选项和全局会话或窗口选项。本文档仅涵盖这些。

#### 显示选项

使用 `show-options` 命令显示选项。`-g` 标志显示全局选项。它可以显示服务器、会话或窗口选项：

* `-s` 显示服务器选项：

~~~~
$ tmux show -s
backspace C-?
buffer-limit 50
...
~~~~

* 不带其他标志的 `-g` 显示全局会话选项：

~~~~
$ tmux show -g
activity-action other
assume-paste-time 1
...
~~~~

* `-g` 和 `-w` 一起显示全局窗口选项：

~~~~
$ tmux show -wg
aggressive-resize off
allow-rename off
...
~~~~

可以通过将选项名称提供给 `show-option` 来显示单个选项值。
当给出选项名称时，没有必要给出 `-s` 或 `-w`，因为 tmux 可以从选项名称中推断出来。例如，要显示 `status` 选项：

~~~~
$ tmux show -g status
status on
~~~~

#### 更改选项

使用 `set-option` 命令设置或取消设置选项。与 `show-option` 一样，
没有必要给出 `-s` 或 `-w`，因为 tmux 可以从选项名称中推断出来。`-g` 对于设置全局会话或窗口选项是必需的；对于服务器选项则无效。

要设置 `status` 选项：

~~~~
set -g status off
~~~~

或 `default-terminal` 选项：

~~~~
set -s default-terminal 'tmux-256color'
~~~~

`-u` 标志取消设置选项。取消设置全局选项会将其恢复为默认值，例如：

~~~~
set -gu status
~~~~

#### 格式

许多选项使用格式。格式提供了一种强大的语法来配置文本的外观，基于 tmux 服务器、会话、窗口或窗格的各种属性。格式在字符串选项中用 `#{}` 括起来，或作为单个大写字母如 `#F`。这是默认的 `status-right`，带有几个格式：

~~~~
$ tmux show -s status-right
status-right "#{?window_bigger,[#{window_offset_x}#,#{window_offset_y}] ,}\"#{=21:pane_title}\" %H:%M %d-%b-%y"
~~~~

格式在 [此文档](https://github.com/tmux/tmux/wiki/Formats) 和 [手册页](https://man.openbsd.org/tmux#FORMATS) 中有描述。

#### 嵌入式命令

一些选项可能包含嵌入式 shell 命令。这仅限于状态行选项，如 `status-left`。嵌入式 shell 命令用 `#()` 括起来。它们可以：

1) 打印一行并退出，在这种情况下，该行将显示在状态行中，并且命令会定期运行以更新它。例如：

   ~~~~
   set -g status-left '#(uptime)'
   ~~~~

   最大间隔由 `status-interval` 选项设置，但如果 tmux 需要，命令可能会更早运行。命令不会每秒运行超过一次。

2) 保持运行并在需要时打印一行，例如：

   ~~~~
   set -g status-left '#(while :; do uptime; sleep 1; done)'
   ~~~~

需要注意的是，通常没有必要为日期和时间使用嵌入式命令，因为 tmux 会在状态行选项中自行展开日期格式如 `%H` 和 `%S`。如果使用像 *date(1)* 这样的命令，任何 `%` 都必须加倍为 `%%`。

#### 颜色和样式

tmux 允许使用简单的语法配置文本的颜色和属性，这被称为样式。样式出现在两个地方：

* 在选项中，如 `status-style`。

* 在选项值中用 `#[]` 括起来，这称为嵌入式样式（见下一节）。

样式有多个由空格或逗号分隔的术语，最有用的是：

* `default` 使用默认颜色；这必须单独出现。默认颜色通常由另一个选项设置，例如在 `status-left` 选项中的嵌入式样式，它是 `status-style`。

* `bg` 设置背景颜色。颜色也给出，例如 `bg=red`。

* `fg` 设置前景颜色。像 `bg` 一样，颜色也给出：`fg=green`。

* `bright` 或 `bold`、`underscore`、`reverse`、`italics` 设置属性。
  这些单独出现，例如：`bright,reverse`。

颜色可以是标准终端颜色的 `black`、`red`、`green`、`yellow`、`blue`、`magenta`、`cyan`、`white`；`brightred`、`brightyellow` 等明亮变体；`colour0` 到 `colour255` 的 256 色调色板颜色；`default` 表示默认颜色；或十六进制 RGB 颜色如 `#882244`。

其余的样式术语在 [手册页](https://man.openbsd.org/tmux#STYLES) 中有描述。

例如，使用 `status-style` 选项将状态行背景设置为蓝色：

~~~~
set -g status-style 'bg=blue'
~~~~

#### 嵌入式样式

嵌入式样式包含在另一个选项中的 `#[` 和 `]` 之间。
每个都会更改后续文本的样式，直到下一个嵌入式样式或文本结束。

例如，在 `status-left` 中放置一些红色和蓝色文本：

~~~~
set -g status-left 'default #[fg=red] red #[fg=blue] blue'
~~~~

因为这很长，所以还需要增加 `status-left-length` 选项：

~~~~
set -g status-left-length 100
~~~~

或者可以有条件地使用嵌入式样式，例如在按下前缀键时显示红色的 `P`，否则显示默认样式：

~~~~
set -g status-left '#{?client_prefix,#[bg=red],}P#[default] [#{session_name}] '
~~~~

#### 有用的选项列表

这是除样式选项外最常用的 tmux 选项的简短列表：

选项|类型|描述
---|---|---
`base-index`|会话|如果设置，则窗口索引从该值开始，而不是从 0 开始
`buffer-limit`|服务器|要保留的自动缓冲区的最大数量，默认为 50
`default-terminal`|服务器|tmux 内部 `TERM` 环境变量的默认值
`display-panes-time`|窗口|`C-b q` 显示窗格编号的时间（毫秒）
`display-time`|会话|状态行上消息显示的时间（毫秒）
`escape-time`|服务器|tmux 在接收到 `Escape` 键后等待的时间，以查看它是否是更长键序列的一部分
`focus-events`|服务器|当活动窗格更改时，tmux 是否发送焦点键序列，以及当从支持它们的外部终端接收到时是否发送
`history-limit`|会话|每个窗格中保留的历史行数的最大值
`mode-keys`|窗口|在复制模式中是否使用 *emacs(1)* 或 *vi(1)* 键绑定
`mouse`|会话|是否启用鼠标
`pane-border-status`|窗口|是否在每个窗格边框中显示状态行：`top` 或 `bottom`
`prefix`|会话|前缀键，默认为 `C-b`
`remain-on-exit`|窗口|当运行在其中的程序退出时，窗格是否自动被杀死
`renumber-windows`|会话|如果为 `on`，则窗口会自动重新编号以关闭窗口列表中的间隙
`set-clipboard`|服务器|当复制文本时，tmux 是否应尝试设置外部 *X(7)* 剪贴板，如果外部终端支持它
`set-titles`|会话|如果为 `on`，tmux 将设置外部终端的标题
`status`|会话|状态行是否可见
`status-keys`|会话|在命令提示符中是否使用 *emacs(1)* 或 *vi(1)* 键绑定
`status-interval`|会话|状态行重绘前的最大时间（秒）
`status-position`|会话|状态行的位置：`top` 或 `bottom`
`synchronize-panes`|窗口|如果为 `on`，则在窗口中的任何窗格中键入的内容会发送到窗口中的所有窗格 - 使用此选项时应小心！
`terminal-overrides`|服务器|tmux 应该从为外部终端指定的 `TERM` 中覆盖的任何功能

#### 样式和格式选项列表

这是最常用的 tmux 样式和格式选项的列表：

选项|类型|描述
---|---|---
`display-panes-active-colour`|会话|`C-b q` 的活动窗格编号样式
`display-panes-colour`|会话|`C-b q` 的窗格编号样式，除了活动窗格
`message-style`|会话|状态行上显示的消息和命令提示符的样式
`mode-style`|窗口|复制模式中选择的样式
`pane-active-border-style`|窗口|活动窗格边框的样式
`pane-border-format`|窗口|如果设置了 `pane-border-status`，窗格边框状态行中显示的文本格式
`pane-border-style`|窗口|窗格边框的样式，除了活动窗格
`status-left-length`|会话|状态行左侧的最大长度
`status-left-style`|会话|状态行左侧的样式
`status-left`|会话|状态行左侧的文本格式
`status-right-length`|会话|状态行右侧的最大长度
`status-right-style`|会话|状态行右侧的样式
`status-right`|会话|状态行右侧的文本格式
`status-style`|会话|整个状态行的样式，部分可能被更具体的选项如 `status-left-style` 覆盖
`window-active-style`|窗口|窗口中活动窗格的默认颜色样式
`window-status-current-format`|窗口|窗口列表中当前窗口的格式
`window-status-current-style`|窗口|窗口列表中当前窗口的样式
`window-status-format`|窗口|窗口列表中除当前窗口外的窗口格式
`window-status-separator`|窗口|窗口列表中窗口之间的分隔符
`window-status-style`|窗口|窗口列表中除当前窗口外的窗口样式
`window-style`|窗口|窗口中窗格的默认颜色样式，除了活动窗格

### 常见配置更改

本节展示了 `.tmux.conf` 的一些常见配置更改示例。

#### 更改前缀键

前缀键由 `prefix` 选项设置。`C-b` 键也绑定到前缀键表中的 `send-prefix` 命令，因此按两次 `C-b` 会将其发送到活动窗格。要更改为 `C-a`：

~~~~
set -g prefix C-a
unbind C-b
bind C-a send-prefix
~~~~

#### 自定义状态行

有许多选项可以自定义状态行。最简单的选项是：

* 关闭状态行：`set -g status off`

* 将其移动到顶部：`set -g status-position top`

* 将背景颜色设置为红色：`set -g status-style bg=red`

* 将右侧文本更改为仅显示时间：`set -g status-right '%H:%M'`

* 下划线当前窗口：`set -g window-status-current-style 'underscore'`

#### 配置窗格边框

可以设置窗格边框颜色：

~~~~
set -g pane-border-style fg=red
set -g pane-active-border-style 'fg=red,bg=yellow'
~~~~

每个窗格可以通过 `pane-border-status` 选项获得状态行，例如以粗体显示窗格标题：

~~~~
set -g pane-border-status top
set -g pane-border-format '#[bold]#{pane_title}#[default]'
~~~~

#### *vi(1)* 键绑定

tmux 支持基于 *vi(1)* 的键绑定用于复制模式和命令提示符。有两个选项设置键绑定：

1) `mode-keys` 设置复制模式的键绑定。如果设置为 `vi`，
   则在复制模式中使用 `copy-mode-vi` 键表；否则使用 `copy-mode` 键表。

2) `status-keys` 设置命令提示符的键绑定。

如果在 tmux 服务器首次启动时，`VISUAL` 或 `EDITOR` 环境变量设置为包含 `vi` 的内容（如 `vi`、`vim`、`nvi`），则这两个选项都设置为 `vi`。

要设置两者都使用 *vi(1)* 键：

~~~~
set -g mode-keys vi
set -g status-keys vi
~~~~

#### 鼠标复制行为

当拖动鼠标复制文本时，tmux 在释放鼠标按钮时复制并退出复制模式。通过更改 `MouseDragEnd1Pane` 键绑定可以配置替代行为。最有用的三种是：

1) 在释放鼠标时不要复制或清除选择，也不要退出复制模式。必须使用键盘复制选择：

~~~~
unbind -Tcopy-mode MouseDragEnd1Pane
~~~~

2) 复制并清除选择但不退出复制模式：

~~~~
bind -Tcopy-mode MouseDragEnd1Pane send -X copy-selection
~~~~

3) 复制但不清除选择：

~~~~
bind -Tcopy-mode MouseDragEnd1Pane send -X copy-selection-no-clear
~~~~

### 其他功能

tmux 有许多本文档未提及的功能和命令，许多功能允许强大的脚本编写。以下是一些可能值得进一步阅读的列表：

* 警报：`monitor-activity`、`monitor-bell`、`monitor-silence`、
  `activity-action`、`bell-action` 和其他选项。

* 单个会话、窗口和窗格的选项。

* 使用 `join-pane` 和 `break-pane` 移动窗格。

* 使用 `send-keys` 向窗格发送按键。

* 命令提示符 `history-file` 选项。

* 使用 `select-layout` 保存的布局字符串。

* 命令序列（用 `;` 分隔）：`select-window; kill-window`。

* 配置文件语法：`{}`、`%if` 等。

* 鼠标键绑定：`MouseDown1Pane` 等。

* 锁定：`lock-command`、`lock-after-time` 和其他选项。

* 使用 `capture-pane` 捕获窗格内容和使用 `pipe-pane` 管道传输。

* 链接窗口：`link-window` 命令。

* 会话组：`new-session` 的 `-t` 标志。

* 使用 `respawn-window` 和 `respawn-pane` 重新生成窗口和窗格。

* 使用 `display-menu` 命令的自定义菜单和使用 `command-prompt` 和 `confirm-before` 的自定义提示。

* 不同的键表：`bind-key` 和 `switch-client` 的 `-T` 标志。

* 空窗格：带有空命令的 `split-window` 和 `display-message` 的 `-I`。

* 钩子：`set-hook` 和 `show-hooks`。

* 脚本同步：`wait-for`。