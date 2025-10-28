## 高级使用

### 关于本文档

本文档简要描述了 tmux 的一些更高级的功能和一些示例。它分为三个部分，涵盖：

* 在交互式使用 tmux 时最有用的功能；

* 用于 tmux 脚本的功能；

* 高级配置。

然而，许多讨论的功能在交互式使用和脚本编写时都很有用。

### 使用 tmux

#### 套接字和多个服务器

tmux 在 `/tmp` 中为用户创建一个目录，然后服务器在该目录中创建一个套接字。默认套接字名为 `default`，例如：

~~~~
$ ls -l /tmp/tmux-1000/default
srw-rw----  1 nicholas  wheel     0B Mar  9 09:05 /tmp/tmux-1000/default=
~~~~

有时创建单独的 tmux 服务器很方便，可能是为了确保一个重要进程完全隔离或测试 tmux 配置。
这可以通过使用 `-L` 标志来完成，该标志在 `/tmp` 中创建一个套接字，但名称不是 `default`。要启动名为 `test` 的服务器：

~~~~
$ tmux -Ltest new
~~~~

或者，tmux 可以通过 `-S` 标志被告知使用 `/tmp` 之外的不同套接字文件：

~~~~
$ tmux -S/my/socket/file new
~~~~

正在运行的服务器使用的套接字可以通过 `socket_path` 格式查看。
这可以通过带有 `-p` 标志的 `display-message` 命令打印出来：

~~~~
$ tmux display -p '#{socket_path}'
/tmp/tmux-1000/default
~~~~

如果套接字意外删除，可以通过向 tmux 服务器发送 `USR1` 信号来重新创建：

~~~~
$ pkill -USR1 tmux
~~~~

#### 警报和监控

警报是在窗口中的窗格发生某些事情时通知用户的一种方式。tmux 支持三种警报：

* 铃声：当程序发送 ASCII `BEL` 字符时。这通过 `monitor-bell` 选项开启或关闭。

* 活动：当从程序接收到任何输出时。这通过 `monitor-activity` 选项开启或关闭。

* 静默：当没有从程序接收到输出时。通过 `monitor-silence` 选项设置必须没有输出的秒数时间段。零时间段禁用此警报。

<img src="../images/tmux_alert_flags.png" align="right" width=368 height=235>

窗格中的警报对包含该窗格窗口的每个会话执行两个操作。

首先，它在窗口列表中的窗口上设置一个标志，但仅当窗口不是当前窗口时。当此标志设置时：

* 窗口在窗口列表中使用 `window-status-bell-style`（铃声）或 `window-status-activity-style`（活动和静默）选项中的样式绘制。默认是使用反向属性。

* 窗口名称后跟 `!` 表示铃声，`#` 表示活动，`~` 表示静默。

当窗口成为当前窗口时，窗口上的警报标志会被清除。可以使用带有 `-C` 标志的 `kill-session` 清除会话中的所有标志：

~~~~
:kill-session -C
~~~~

<img src="../images/tmux_alert_message.png" align="right" width=368 height=235>

`C-b M-n` 和 `C-b M-p` 键绑定使用 `-a` 标志移动到下一个或上一个带有警报的窗口，分别对应 `next-window` 和 `previous-window` 命令。

其次，它可能在状态行中显示消息，在外部终端中响铃，或两者兼有。这是铃声还是消息由 `visual-bell`、`visual-activity` 和 `visual-silence` 选项控制。何时采取此操作的选择由 `bell-action`、`activity-action` 和 `silence-action` 选项控制，这些选项可以是：

值|含义
---|---
`any`|会话中任何窗口的警报都会触发操作
`none`|会话中不触发任何操作
`current`|当前窗口中的铃声、活动或静默会触发警报，但其他窗口不会
`other`|除当前窗口外，任何窗口中的铃声、活动或静默都会触发警报

#### 工作目录

每个 tmux 会话都有默认工作目录。这是用于每个新窗格的工作目录。

会话的工作目录在首次创建时设置：

* 可以通过 `-c` 标志提供给 `new-session`，例如：

~~~~
$ tmux new -c/tmp
~~~~

* 如果会话是从键绑定或命令提示符创建的，它是附加会话的工作目录

* 如果会话是从 tmux 内部或外部的 shell 提示符创建的，它是 shell 的工作目录。

会话的工作目录可以通过 `-c` 标志更改为 `attach-session`，例如：

~~~~
:attach -c/tmp
~~~~

当创建窗口或窗格时，可以通过 `-c` 提供给 `new-window` 或 `split-window` 工作目录。这用于代替会话的默认工作目录：

~~~~
:neww -c/tmp
~~~~

或者：

~~~~
:splitw -c/tmp
~~~~

tmux 可以尝试从窗格外读取窗格的当前工作目录。这在 `pane_current_path` 格式中可用。这改变了 `C-b "` 绑定，以创建与活动窗格相同工作目录的新窗格：

~~~~
bind '"' splitw -c '#{pane_current_path}'
~~~~

#### 链接窗口

XXX

#### 重新生成窗格和窗口

重新生成窗格或窗口是一种启动不同程序（或重新启动相同程序）而无需重新创建窗口的方法，保持其大小、位置和索引。

`respawn-pane` 命令重新生成窗格，`respawn-window` 重新生成窗口。默认情况下，它们运行与最初使用 `split-window` 或 `new-window` 创建时相同的程序：

~~~~
:respawn-pane
~~~~

可以提供不同的命令作为参数：

~~~~
:respawn-pane top
~~~~

如果程序仍在窗格或窗口中运行，命令将拒绝工作。`-k` 标志在启动新程序之前杀死窗口中的程序：

~~~~
:respawn-pane -k top
~~~~

像 `split-window` 一样，`respawn-pane` 和 `respawn-window` 有 `-c` 标志来设置工作目录。

<img src="../images/tmux_remain_on_exit.png" align="right" width=368 height=235>

`respawn-pane` 和 `respawn-window` 与 `remain-on-exit` 选项一起使用很有用。当此选项开启时，当运行在其中的程序退出时，窗格不会自动杀死。相反，会显示一条消息，窗格保持原样。这称为死窗格，可以使用 `respawn-pane` 或 `respawn-window` 启动相同或不同的程序。

#### 窗口大小

每个窗口都有一个大小，即其水平和垂直尺寸。窗口的大小由链接到其会话的客户端的大小决定。如何做到这一点由 `window-size` 选项控制，该选项可以是：

值|含义
---|---
`largest`|窗口具有最大附加客户端的大小；在较小的客户端上只显示窗口的一部分
`smallest`|窗口具有最小附加客户端的大小；在较大的客户端上，任何未使用的空间都用 `·` 字符填充
`latest`|窗口具有最近使用的客户端的大小，例如通过在其中键入
`manual`|窗口大小是固定的；新窗口使用 `default-size` 选项，并且可以用 `resize-window` 命令调整大小

当窗口未链接到附加的会话时，窗口大小不会改变。

如果窗口从未链接到附加的会话 - 例如当作为 `new-session` 的一部分使用 `-d` 创建时 - 它从 `default-size` 选项获取其大小。这是一个会话选项，默认为 80x24：

~~~~
$ tmux show -g default-size
80x24
~~~~

创建会话时，可以同时使用 `-x` 和 `-y` 标志设置其 `default-size` 选项：

~~~~~
$ tmux new -smysession -d -x160 -y48
$ tmux show -tmysession default-size
default-size 160x48
$ tmux lsw -tmysession
0: ksh* (1 panes) [160x48] [layout cc01,160x48,0,0,4] @4 (active)
~~~~~

当窗口大于显示它的客户端时，可见区域会跟踪光标位置。可以使用以下键查看窗口的不同区域。

键|功能
---|---
`C-b S-Up`|向上移动可见区域
`C-b S-Down`|向下移动可见区域
`C-b S-Left`|向左移动可见区域
`C-b S-Right`|向右移动可见区域
`C-b DC` (`C-b Delete`)|返回跟踪光标位置

可见区域是客户端的属性，因此分离客户端或更改当前窗口将重置为光标位置。这些键绑定到 `refresh-client` 命令。

可以使用 `resize-window` 命令为现有窗口设置窗口大小。这会设置大小并自动将该窗口的 `window-size` 选项设置为 `manual`。例如：

~~~~
:resizew -x200 -y100
~~~~

要调整大小向上（`-U`）、向下（`-D`）、向左（`-L`）或向右（`-R`）：

~~~~
:resizew -L 20
~~~~

或者返回到从附加客户端计算大小：

~~~~
:resizew -A
~~~~

#### 会话组

XXX

#### 管道窗格更改

tmux 允许将窗格的任何新更改通过管道传输到命令。这可以用于，例如，制作窗格的日志。`pipe-pane` 命令执行此操作：

~~~~
:pipe-pane 'cat >~/mypanelog'
~~~~

没有参数会停止管道传输：

~~~~
:pipe-pane
~~~~

`pipe-pane` 的 `-I` 标志将命令的输出发送到窗格。例如，这将发送 `foo` 到窗格，就像它已被键入一样：

~~~~
:pipe-pane -I 'echo foo'
~~~~

像这样使用时，带有 `-I` 的 `pipe-pane` 类似于稍后部分介绍的 `send-keys` 命令。

`-o` 标志将切换管道传输 - 如果尚未启动则启动，否则停止。这对于从单个键绑定启动和停止很有用：

~~~~
bind P pipe-pane -o 'cat >~/mypanelog'
~~~~

#### 窗格标题和终端标题

tmux 中的每个窗格都有一个标题。窗格的标题可以由在窗格中运行的程序设置。如果程序在 tmux 外部运行，它将设置外部终端标题 - 通常显示在 *X(7)* 窗口标题中。因为 tmux 可以在其中运行多个程序，所以每个窗格都有一个窗格标题，而不是只有一个。窗格标题与窗口名称不同，窗口名称仅由 tmux 使用，并且在窗口中的所有窗格中都是相同的。

tmux 内部的程序可以使用如下转义序列设置窗格标题：

~~~~
$ printf '\033]2;title\007'
~~~~

tmux 在状态行右侧的引号中显示活动窗格的窗格标题。

可以通过 tmux 使用 `-T` 标志更改窗格的窗格标题到 `select-pane` 命令：

~~~~
:selectp -Tmytitle
~~~~

但是没有什么能阻止 tmux 内部的程序在之后再次更改标题。

tmux 可以自己设置外部终端标题，这由 `set-titles` 选项控制：

~~~~
set -g set-titles on
~~~~

默认标题包括附加会话和当前窗口的名称，以及活动窗格的窗格标题和任何带有警报的窗口的索引。这可以通过 `set-titles-string` 选项更改。例如，这仅使用窗格标题：

~~~~
set -g set-titles-string '#{pane_title}'
~~~~

#### 鼠标键绑定

tmux 通过将鼠标事件映射到键绑定来处理大多数鼠标行为。
鼠标键有特殊名称，即事件，后跟按钮编号（如果有），然后是鼠标事件发生的区域。例如：

- `MouseDown1Pane` 表示鼠标在窗格上按下按钮 1；

- `DoubleClick2Status` 表示在状态行上双击鼠标按钮 2；

- `MouseDrag1Pane` 和 `MouseDragEnd1Pane` 表示在窗格上开始和结束鼠标拖动。

- `WheelUpStatusLeft` 表示在状态行左侧向上滚动鼠标滚轮

终端仅支持三个按钮和鼠标滚轮。

可能的鼠标事件有：

事件|描述
---|---
WheelUp|鼠标滚轮向上
WheelDown|鼠标滚轮向下
MouseDown|鼠标按钮按下
MouseUp|鼠标按钮释放
MouseDrag|鼠标拖动开始
MouseDragEnd|鼠标拖动结束
DoubleClick|双击
TripleClick|三击

鼠标事件可能发生的地方有：

区域|描述
---|---
Pane|窗格的内容
Border|窗格边框
Status|状态行窗口列表
StatusLeft|状态行的左侧部分
StatusRight|状态行的右侧部分
StatusDefault|状态行的任何其他部分

绑定到鼠标键绑定的命令可以使用 `-t` 和鼠标目标（`=` 或 `{mouse}`）告诉 tmux 他们想要使用鼠标事件发生的窗格或窗口。例如，这将状态行窗口列表上的双击绑定到缩放窗口的活动窗格：

~~~~
bind -Troot DoubleClick1Status resizep -Zt=
~~~~

当在窗格中运行的程序本身可以处理鼠标时，可以使用带有 `-M` 标志的 `send-keys` 将鼠标事件传递给该程序。如果程序已打开鼠标，则 `mouse_any_flag` 格式变量为真。例如，此绑定使按钮 2 粘贴，除非在处于模式或程序已为自己启用鼠标的窗格上使用：

~~~~
bind -Troot MouseDown2Pane selectp -t= \; if -F "#{||:#{pane_in_mode},#{mouse_any_flag}}" "send -M" "paste -p"
~~~~

#### 环境

XXX

### 脚本化 tmux

#### 脚本基础

tmux 设计为易于脚本化。几乎所有命令在使用 `tmux` 二进制文件运行时的工作方式与从键绑定或 tmux 内部的命令提示符运行时相同。

tmux 通常使用 shell 脚本编写，当然也可以使用其他语言。
本文档中的所有示例都适用于基于 Bourne shell 的 shell。

格式是脚本化 tmux 的重要部分，熟悉它们很有用，参见 [此文档](https://github.com/tmux/tmux/wiki/Formats) 和 [手册页部分](https://man.openbsd.org/tmux#FORMATS)。

脚本在预期用途上可能有很大差异，这会影响它们的编写方式。仅从键绑定交互式运行的脚本可能能够假设当前窗口或活动窗格在脚本运行期间不会改变，因此无需担心目标。设计用于设置新会话或从其他程序运行的脚本可能需要更加小心。

#### 唯一标识符

tmux 中的每个窗格、窗口和会话都有一个由服务器设置的唯一标识符 (ID)。不同的 tmux 服务器可以使用相同的 ID，但在运行的服务器内，每个 ID 永远不会改变或重复使用。

窗格 ID 以 `%` 为前缀（例如 `%0` 或 `%123`），窗口以 `@` 为前缀（例如 `@1` 或 `@99`），会话以 `$` 为前缀（例如 `$3` 或 `$42`）。

ID 允许脚本针对窗格、窗口或会话，并保证它们始终相同，即使它们被杀死、移动或重命名。

ID 可通过 `pane_id`、`window_id` 和 `session_id` 格式变量获得：

~~~~
$ tmux lsp -F '#{session_id} #{window_id} #{pane_id}'
$0 @8 %8
$0 @8 %11
~~~~

#### 特殊环境变量

tmux 在每个窗格中设置两个环境变量，`TMUX` 和 `TMUX_PANE`：

- `TMUX` 由 tmux 用于计算在窗格内运行的命令的服务器套接字路径。这通常用于查看脚本是否在 tmux 内运行：

  ~~~~
  $ [ -n "$TMUX" ] && echo inside tmux
  ~~~~
  
  第一个逗号 (`,`) 之前的内容是套接字路径，其余部分供内部使用。获取套接字路径的一种方法：

  ~~~~
  $ echo $TMUX|awk -F, '{ print $1 }'
  ~~~~

  注意，没有必要这样做来通过 `-S` 将套接字路径提供给 tmux - tmux 可以自己计算出来。

- `TMUX_PANE` 是窗格 ID：

  ~~~~
  $ echo $TMUX_PANE
  %11
  ~~~~

#### 默认目标

运行许多 tmux 命令时，它们必须确定应该影响哪个会话、窗口或窗格。这被称为目标，由会话、窗口和窗格组成。并非所有这些组件都被每个命令使用，例如 `split-window` 需要知道要针对哪个窗口，但不关心会话或窗格。

大多数命令可以使用 `-t` 标志指定目标会话、窗口或窗格，而不是依赖默认目标。这将在下一节中描述。如果未给出 `-t`，则使用默认目标。

tmux 如何确定默认目标取决于命令从何处运行。有三种典型情况：

1) 命令从 tmux 本身交互式运行，例如从键绑定或命令提示符。

    这是最简单的：tmux 知道命令运行的客户端，因为用户必须触发键绑定或在命令提示符处按 `Enter`。
从客户端，它知道附加的会话，从中它知道当前窗口和活动窗格。这就是默认目标。

2) 命令从在 tmux 内运行的程序运行，例如在窗格中的 shell 提示符下键入。

   在这种情况下，tmux 不知道命令键入到哪个客户端，
因为它可能从脚本运行，或由 *sleep(1)* 延迟，或几种其他事情。

   但是，tmux 可能知道运行命令的 *tty(4)* 或 *pty(4)* 的名称。如果知道，它可以用来确定窗格，因为每个 *tty(4)* 或 *pty(4)* 都属于一个窗格。即使 *tty(4)* 或 *pty(4)* 不可用，窗格 ID 也可能在 `TMUX_PANE` 环境变量中。

   如果 tmux 能够找到窗格，那么它也有窗口，因为每个窗格属于一个窗口。如果该窗口只属于一个会话，这会给出默认目标的会话和窗口（tmux 将始终使用它找到的窗口中的活动窗格）。

   如果窗口属于多个会话，则 tmux 会选择最近使用的会话。如果窗口多次链接到会话（因此它有多个窗口索引），则如果窗口是会话中的当前窗口，则使用当前窗口，否则使用最低的窗口索引。

3) 命令从在 tmux 外运行的程序运行，例如在不运行 tmux 的不同 *xterm(1)* 中的 shell 提示符。

   对于这种情况，tmux 对目标没有任何环境信息。
所以它会选择最近使用的会话，并使用其当前窗口和活动窗格。

如果使用命令序列，则为序列中的第一个命令计算默认目标，并且后续命令使用相同的目标，除非这些命令明确更改目标 - 例如，没有 `-d` 的 `split-window` 会将同一命令序列中后续命令的目标更改为新创建的窗格。

#### 命令目标

大多数命令接受 `-t` 参数来指定目标会话、窗口或窗格，而不是依赖默认目标。命令通常需要会话、窗口或窗格。命令的用法显示了这一点；可以通过 `list-commands` 或手册页看到。例如，`send-prefix` 需要一个窗格，所以它说 `-t target-pane`：

~~~~
$ tmux lscm send-prefix
send-prefix [-2] [-t target-pane]
~~~~

目标由三个部分组成：会话、窗口和窗格。会话和窗口由冒号（`:`）分隔，窗口和窗格由句点（`.`）分隔：

~~~~
session:window.pane
~~~~

这三个组件中的任何一个都可以省略，在这种情况下，如果需要，tmux 将计算出最合适的内容，类似于它如何计算默认目标。

如果目标中既没有 `:` 也没有 `.`，tmux 会根据命令的需要以不同方式解释它。如果命令需要 `target-pane`，则 `-t1` 首先会尝试作为窗格，只有在找不到窗格 1 时才会作为窗口；如果命令需要 `target-window`，则 `-t1` 只会查找索引 1 的窗口。例如，请注意在创建窗格 1 后，窗口如何从 `@1` 变为 `@8`：

~~~~
$ tmux display -pt1 -F '#{window_id} #{pane_id}'
@1 %1
$ tmux splitw -d
$ tmux display -pt1 -F '#{window_id} #{pane_id}'
@8 %15
~~~~

当 tmux 交互式使用时，这种行为是有效的，但对于脚本编写，必须确保目标是正确的。这最好通过注意命令需要会话、窗口还是窗格，并使用 ID 和命令需要的完整目标来完成。

在目标中，`session`、`window` 和 `pane` 都可以有几种不同的形式。`session` 可以通过几种方式给出。最有用的是：

1) 会话 ID，例如 `$1`，它将始终匹配一个会话。

2) 会话的确切名称前缀为 `=`，例如 `=mysession`。
这只会匹配名为 `mysession` 的会话。

3) 会话名称的开头。例如，`my` 将匹配 `mysession` 或
`myothersession`。

4) 与会话名称匹配的模式。这可以有 `*` 和 `?`
通配符：`f*` 将匹配 `foo` 但不匹配 `bar`。

`window` 最有用的形式是：

1) 窗口 ID，例如 `@42`。

2) 窗口索引，例如 `1` 表示窗口 1，`99` 表示窗口 `99`。

3) `{start}`（或 `^`）表示最低窗口索引或 `{end}`（或 `$`）表示
最高。

4) `{last}`（或 `!`）表示最后一个窗口，`{next}`（或 `+`）表示下一个和
`{previous}`（或 `-`）表示上一个。

`pane` 可以给出为：

1) 窗格 ID，例如 `%0`。

2) 窗格索引，例如 `3`。

3) 以下特殊标记之一：

    标记|含义
    ---|---
    `{last}`（或 `!`）|最后（之前活动的）窗格
    `{next}`（或 `+`）|按数字的下一个窗格
    `{previous}`（或 `-`）|按数字的上一个窗格
    `{top}`|顶部窗格
    `{bottom}`|底部窗格
    `{left}`|最左边的窗格
    `{right}`|最右边的窗格
    `{top-left}`|左上角窗格
    `{top-right}`|右上角窗格
    `{bottom-left}`|左下角窗格
    `{bottom-right}`|右下角窗格
    `{up-of}`|活动窗格上方的窗格
    `{down-of}`|活动窗格下方的窗格
    `{left-of}`|活动窗格左侧的窗格
    `{right-of}`|活动窗格右侧的窗格
     
一些目标示例：

示例|描述
---|---
`-t1`|根据命令需要的会话、窗口或窗格 1
`-t%1`|ID 为 `%1` 的窗格；如果需要，会话和窗口将由 tmux 选择
`-t:6.%1`|如果窗口 6 中存在 ID 为 `%1` 的窗格；如果需要，会话将由 tmux 选择
`-t:.3`|窗格 3；如果需要，会话和窗口将由 tmux 选择
`-t=mysession:5`|会话 `mysession` 中的窗口 5；如果需要窗格，将使用活动窗格
`-t=mysession:5.2`|会话 `mysession` 中窗口 5 的窗格 2
`-t{last}`|最后一个窗口或最后一个窗格，取决于命令需要窗口还是窗格
`-t:{last}`|最后一个窗口；如果需要，会话和窗格将由 tmux 选择

#### 新窗格、窗口和会话的目标

`split-window`、`new-window` 和 `new-session` 命令都有一个 `-P`
标志，将新窗格、窗口或会话的目标打印到 `stdout`。
这允许脚本可靠地将其作为后续命令的目标。

默认情况下，输出是完整或部分目标，例如：

~~~~
$ tmux new -dP
2:
~~~~

但使用 `-F` 标志获取 ID 更有用：

~~~~
$ S=$(tmux new -dPF '#{session_id}')
$6
$ tmux neww -dPF '#{window_id}' -t$S
@16
~~~~

#### 获取信息

从 tmux 服务器获取信息有三种主要方式：列表
命令、`display-message` 和 `show-options`。

列表命令是 `list-panes`、`list-windows` 和 `list-sessions`。

`list-sessions` 列出服务器中的所有会话。

`list-windows` 可以以下方式使用：

- 不带参数，列出单个会话中的所有窗口。

- 带 `-a` 列出服务器中的所有窗口。

`list-panes` 可以以下方式使用：

- 不带参数，列出单个窗口中的所有窗格。

- 带 `-s` 列出单个会话中所有窗口中的所有窗格。

- 带 `-a` 列出整个服务器中的所有窗格。

这些命令都有一个 `-F` 标志，给出每行输出的格式。例如，列出服务器中的每个窗口及其名称：

~~~~
$ tmux lsw -aF '#{window_id} #{window_name}'
@0 top
@1 emacs
@2 mywindow
@3 ksh
@4 abc🤔def
@5 ksh
~~~~

或者单个窗口中的每个窗格及其大小：

~~~~
$ tmux lsp -t@7 -F '#{pane_id} #{pane_width} #{pane_height}'
%7 107 43
%12 53 42
%14 53 42
~~~~

这些可以与 *sh(1)* 结合使用来循环遍历窗格：

~~~~
$ tmux lsp -F'#{pane_id}'|while read i; do echo pane $i; done
~~~~

`display-message` 命令用于打印单个格式。`-p`
标志将输出发送到 `stdout`。例如：

~~~~
$ tmux display -p '#{pane_id}'
%8
~~~~

或者：

~~~~
$ tmux display -pt@0 '#{window_name}'
top
~~~~

选项使用 `show-options` 命令显示。基础知识在 [此部分](https://github.com/tmux/tmux/wiki/Getting-Started#showing-options) 中有介绍。
此外，`-v` 选项仅显示值：

~~~~
$ tmux show -g history-limit
history-limit 2000
$ tmux show -vg history-limit
2000
~~~~

`-q` 不会显示未知选项的错误：

~~~~
$ tmux show -g no-such-option
invalid option: no-such-option
$ tmux show -gq no-such-option
$
~~~~

#### 发送按键

`send-keys` 命令可用于向窗格发送按键，就像它们已被按下一样。它接受多个参数。tmux 检查每个参数是否是键的名称，如果是，则发送该键的适当转义序列；如果参数不匹配键，则按原样发送。例如：

<img src="../images/tmux_send_keys.png" align="right" width=368 height=235>

~~~~
send hello Enter
~~~~

发送 `hello` 中的五个字符，后跟 Enter 键按下（换行符）。或者这个：

~~~~
send F1 C-F2
~~~~

发送 `F1` 和 `C-F2` 键的转义序列。

`-l` 标志告诉 tmux 不要将参数视为键，而是逐字发送每一个，所以这将发送字面文本 `Enter`：

~~~~
send -l Enter
~~~~

#### 捕获窗格内容

现有窗格内容可以通过 `capture-pane` 命令捕获。这可以将其输出保存到粘贴缓冲区，或者更有用的是，通过给出 `-p` 标志将其写入 `stdout`。

默认情况下，`capture-pane` 捕获整个可见窗格内容：

~~~~
$ tmux capturep -pt%0
~~~~

`-S` 和 `-E` 标志给出起始和结束行号。零是第一个可见行，负行进入历史记录。特殊值 `-` 表示历史记录的开始或可见内容的结束。因此，要捕获包括历史记录在内的整个窗格：

~~~~
$ tmux capturep -p -S- -E-
~~~~

一些附加标志控制输出格式：

* `-e` 包含颜色和属性的转义序列；

* `-C` 将不可打印字符转义为八进制序列；

* `-N` 保留行末的尾随空格；

* `-J` 既保留尾随空格又连接任何换行的行。

#### 空窗格

tmux 允许创建没有运行命令的窗格。有两种方法
使用 `split-window` 命令创建空窗格：

1) 传递空命令：

    ~~~~
    $ tmux splitw ''
    ~~~~

   像这样创建的窗格完全为空。

<img src="../images/tmux_empty_pane.png" align="right" width=376 height=243>

2) 通过使用 `-I` 标志并在 `stdin` 上提供输入：

    ~~~~
    $ echo hello|tmux splitw -I
    ~~~~

现有空窗格可以通过 `-I` 标志写入
`display-message`：

~~~~
P=$(tmux splitw -dPF '#{pane_id}' '')
echo hello again|tmux display -It$P
~~~~

它们接受转义序列，就像在窗格中运行的程序发送它们一样：

~~~~
printf '\033[H\033[2J\033[31mred'|tmux display -It$P
~~~~

#### 等待、信号和锁

XXX

### 高级配置

#### 检查配置文件

`source-file` 命令有两个标志来帮助处理配置
文件：

* `-n` 解析文件但不执行任何命令。

* `-v` 将每个命令的解析形式打印到 `stdout`。

这些可以用于定位配置文件中的问题，例如通过
不带 `.tmux.conf` 启动 tmux，然后手动加载它：

~~~~
$ tmux -f/dev/null new -d
$ tmux source -v ~/.tmux.conf
/home/nicholas/.tmux.conf:1: set-option -g mouse on
/home/nicholas/.tmux.conf:8: unknown command: foobar
~~~~

#### 命令解析

当 tmux 读取配置文件时，它在两个主要步骤中处理：
解析和执行。解析过程是：

1) 配置文件指令被处理，例如 `%if`。这些在
   下一节中有描述。

2) 命令被解析并拆分为一组参数。例如取
   命令：
   ~~~~
   new -A -sfoo top
   ~~~~
   它首先被拆分为四个列表：`new`、`-A`、`-sfoo` 和 `top`。

3) 这个列表再次被处理，tmux 查找命令，所以它知道
   它是带有参数 `-A`、`-s foo` 和 `top` 的 `new-session`。

4) 命令被放置在命令队列的末尾。

一旦所有配置文件被解析，执行就开始：命令
按顺序从命令队列执行。

对于从命令提示符读取的命令或作为
另一个命令的参数（例如 `if-shell`），也会发生类似的过程。这些基本上与只有一个行的配置文件相同。

对于从 shell 运行的命令，跳过步骤 1 和 2 - 不支持配置文件
指令，shell 在将命令提供给 tmux 之前将命令拆分为参数。

这种解析和执行的分离通常不会有任何可见效果
但偶尔会很重要。最明显的效果是环境变量
扩展：

~~~~
setenv -g FOO bar
display $FOO
~~~~

这不会按预期工作，因为 `set-environment` 命令发生在
执行期间，而 `FOO` 的扩展发生在解析期间。
然而，这会起作用：

~~~~
FOO=bar
display $FOO
~~~~

因为 `FOO=bar` 和 `FOO` 的扩展都发生在解析期间。类似地
这会起作用：

~~~~
setenv -g FOO bar
display '#{FOO}'
~~~~

虽然 `set-environment` 发生在执行期间，但 `FOO` 直到
`display-message` 被执行并将其参数扩展为格式时才被使用。

对于将另一个命令作为参数的命令必须小心，
因为可能有多个解析阶段。

#### 条件指令

tmux 在配置文件中支持一些特殊语法，以允许更
灵活的配置。这些都在配置文件被解析时处理。

条件指令允许仅在条件为真时处理配置文件的部分。条件指令看起来像这样：

~~~~
%if #{format}
commands
%endif
~~~~

如果给定的格式为真（扩展后不为空且不为 0），则
执行命令。`%if` 的附加分支可以用
`%elif` 给出，或用 `%else` 给出假分支：

~~~~
%if #{format}
commands
%elif #{format}
more commands
%else
yet more commands
%endif
~~~~

因为这些指令在配置文件被解析时处理，
它们不能使用命令的结果 - 命令（无论是在
条件之外还是在真或假分支中）直到配置文件完全解析后才会执行。

例如，这在不同主机上运行不同的配置文件：

~~~~
%if #{==:#{host_short},firsthost}
source ~/.tmux.conf.firsthost
%elif #{==:#{host_short},secondhost}
source ~/.tmux.conf.secondhost
%endif
~~~~

#### 运行 shell 命令

`run-shell` 命令运行一个 shell 命令：

~~~~
:run 'ls'
~~~~

如果有任何输出，活动窗格会切换到视图模式。格式在 `run-shell` 参数中扩展：

~~~~
:run 'echo window name is #{window_name}'
~~~~

`run-shell` 阻止后续命令的执行，直到命令
完成。`-b` 标志禁用此功能并在后台运行命令。

`run-shell` 最适合从配置文件或键绑定调用 shell 命令或 shell 脚本：

~~~~
bind O run '/path/to/my/script'
~~~~

#### 使用 `if-shell` 的条件

`if-shell` 是一个多功能命令，允许基于 shell 命令或（使用 `-F`）格式在两个 tmux 命令之间进行选择。第一个
参数是条件，第二个是在为真时运行的命令，第三个是在为假时运行的命令。第三个命令可以省略。

如果给出 `-F`，第一个条件参数是格式。如果格式扩展为非空且非 0 的字符串，则格式为真。不带 `-F`，第一个参数是 shell 命令。

例如，一个键绑定，如果窗格处于复制模式则滚动到顶部，否则不执行任何操作：

~~~~
bind T if -F '#{==:#{pane_mode},copy-mode}' 'send -X history-top'
~~~~

或者根据时间重命名窗口：

~~~~
bind A if 'test `date +%H` -lt 12' 'renamew morning' 'renamew afternoon'
~~~~

请注意，`if-shell` 与 `%if` 指令不同。`%if` 在解析配置文件时解释；`if-shell` 是一个与其他命令一起运行的命令，可以在键绑定中使用。

#### 使用 `{}` 引用

tmux 允许使用 `{` 和 `}` 引用配置文件的部分。
这是为了允许更清楚地表达复杂命令和命令序列，特别是当命令将另一个命令作为参数时。`{` 和 `}` 之间的文本被视为未经任何修改的字符串。

因此，举个简单的例子，`bind-key` 命令可以将命令作为其参数：

~~~~
bind K {
	lsk
}
~~~~

或者 `if-shell` 可以绑定到键：

~~~~
bind K {
	if -F '#{==:#{window_name},ksh}' {
		kill-window
	} {
		display 'not killing window'
	}
}
~~~~

这相当于：

~~~~
bind K if -F '#{==:#{window_name},ksh}' 'kill-window' "display 'not killing window'"
~~~~

#### 数组选项

一些 tmux 选项可以设置为多个值，这些称为数组选项。每个值都有一个索引，显示在选项名称后的 `[` 和 `]` 中。数组索引可以有间隙，所以只有索引 0 和 999 的数组是可以的。数组选项是 `command-alias`、`terminal-features`、`terminal-overrides`、`status-format`、`update-environment` 和 `user-keys`。每个钩子也是一个数组选项。

可以设置或显示单个数组索引：

~~~~
$ tmux set -g update-environment[999] FOO
$ tmux show -g update-environment[999]
update-environment[999] FOO
$ tmux set -gu update-environment[999]
~~~~

或者通过省略索引一起显示。`-u` 将整个数组选项恢复为默认值：

~~~~
$ tmux show -g update-environment
update-environment[0] DISPLAY
update-environment[1] KRB5CCNAME
update-environment[2] SSH_ASKPASS
update-environment[3] SSH_AUTH_SOCK
update-environment[4] SSH_AGENT_PID
update-environment[5] SSH_CONNECTION
update-environment[6] WINDOWID
update-environment[7] XAUTHORITY
update-environment[999] FOO
$ tmux set -gu update-environment
$ tmux show -g update-environment
update-environment[0] DISPLAY
update-environment[1] KRB5CCNAME
update-environment[2] SSH_ASKPASS
update-environment[3] SSH_AUTH_SOCK
update-environment[4] SSH_AGENT_PID
update-environment[5] SSH_CONNECTION
update-environment[6] WINDOWID
update-environment[7] XAUTHORITY
~~~~

`set-option` 的 `-a` 标志使用下一个空闲索引追加到数组选项：

~~~~
$ tmux set -ag update-environment 'FOO'
$ tmux show -g update-environment
update-environment[0] DISPLAY
update-environment[1] KRB5CCNAME
update-environment[2] SSH_ASKPASS
update-environment[3] SSH_AUTH_SOCK
update-environment[4] SSH_AGENT_PID
update-environment[5] SSH_CONNECTION
update-environment[6] WINDOWID
update-environment[7] XAUTHORITY
update-environment[8] FOO
~~~~

`-a` 可以接受由逗号分隔的多个值。为了向后兼容旧的 tmux 版本，其中数组保存为字符串，可以给出前导逗号：

~~~~
$ tmux set -ag update-environment ',FOO,BAR'
~~~~

#### 命令别名

tmux 通过定义命令别名允许自定义命令。请注意，这与每个命令的短别名（例如 `lsw` 对于 `list-windows`）不同。
命令别名使用 `command-alias` 服务器选项定义。这是一个数组选项，其中每个条目都有一个数字。

默认设置有一些便利设置和一些向后兼容设置：

~~~~
$ tmux show -s command-alias
command-alias[0] split-pane=split-window
command-alias[1] splitp=split-window
command-alias[2] "server-info=show-messages -JT"
command-alias[3] "info=show-messages -JT"
command-alias[4] "choose-window=choose-tree -w"
command-alias[5] "choose-session=choose-tree -s"
~~~~

以 `command-alias[4]` 为例，这意味着 `choose-window`
命令扩展为 `choose-tree -w`。

通过向数组添加新索引来添加自定义命令别名。因为
默认值从索引 0 开始，所以最好为额外的
命令别名使用更高的数字：

~~~~
:set -s command-alias[100] 'sv=splitw -v'
~~~~

此选项使 `sv` 与 `splitw -v` 相同：

~~~~
:sv
~~~~

输入命令的任何后续标志或参数都会附加到
替换命令。这与 `splitw -v -d` 相同：

~~~~
:sv -d
~~~~

#### 用户选项

tmux 允许设置自定义选项，这些称为用户选项，可以是
窗格、窗口、会话或服务器选项。所有用户选项都是字符串，名称必须以 `@` 为前缀。对值没有其他限制。

用户选项可用于存储来自脚本或键绑定的自定义值。
因为 tmux 还不知道选项名称，所以必须为窗口选项给出 `-w` 标志，或为服务器选项给出 `-s`。例如，在窗口 2 上设置一个选项，窗口名称为：

~~~~
$ tmux set -Fwt:2 @myname '#{window_name}'
$ tmux show -wt:2 @myname
@mytime ksh
~~~~

或者全局会话选项：

~~~~
$ tmux set -g @myoption 'foo bar'
$ tmux show -g @myoption
foo bar
~~~~

用户选项对脚本编写很有用，参见 [此部分](Advanced-Use#getting-information)。

#### 用户键

tmux 允许设置一组自定义键定义。这在终端发送（或可以配置为发送）tmux 默认不识别的不寻常键序列的罕见场合下很有用。

用户键通过 `user-keys` 服务器选项添加。这是一个数组选项，其中每个项目是 tmux 匹配到 `UserN` 键的序列。例如：

~~~~
set -s user-keys[0] '\033[foo'
~~~~

有了这个，当从终端接收到序列 `\033[foo` 时，tmux
将触发一个可以像往常一样绑定的 `User0` 键：

~~~~
bind -n User0 list-keys
~~~~

`user-keys[1]` 映射到 `User1`，`user-keys[2]` 映射到 `User2`，依此类推。

#### 自定义键表

自定义键表是名称不同于四个默认值（`root`、
`prefix`、`copy-mode` 和 `copy-mode-vi`）的键表。在表中绑定键会创建
该表，例如这创建了一个名为 `mytable` 的键表，其中
`list-keys` 绑定到 `x`：

~~~~
bind -Tmytable x list-keys
~~~~

每个客户端都有一个当前键表，可以设置为无键表。当按下键时，键处理的工作方式是：

1) 如果键匹配 `prefix` 或 `prefix2` 选项，客户端会切换
   到 `prefix` 键表，然后 tmux 等待另一个按键按下。

2) 如果不匹配，则在客户端的键表中查找键。如果
   客户端没有键表，则首先切换到由
   `key-table` 选项给出的键表（或在复制模式下为 `copy-mode` 或 `copy-mode-vi` 键表）。

3) 如果在表中找到键绑定，则执行其命令。如果未找到键
   绑定，tmux 会查找 `Any` 键绑定，如果找到则执行其命令。

4) 如果键不重复，则客户端重置为无键表并等待
   下一个按键按下。如果重复，则客户端保留找到键的键
   表，因此下一个按键按下也会首先尝试该表。

`switch-client` 命令的 `-T` 标志可用于显式设置
客户端的键表，因此当下一个键被按下时，它会在该
键表中查找。这可用于绑定命令链或具有不同命令的多个
前缀。例如，要使按下 `C-x` 然后 `x`
执行 `list-keys`，首先创建一个带有 `x` 绑定的键表，然后为 `C-x` 创建一个根
绑定以切换到该键表：

~~~~
bind -Tmytable x list-keys
bind -Troot C-x switch-client -Tmytable
~~~~

要为单个会话完全更改 `root` 键表，可以更改 `key-table`
选项：

~~~~
set -tmysession: key-table mytable
~~~~

#### 自动重命名

XXX

#### *terminfo(5)* 和 `terminal-overrides`

XXX

#### 系统剪贴板

当复制文本时，tmux 可以更新系统剪贴板，有关信息请参见
[此文档](https://github.com/tmux/tmux/wiki/Clipboard)。

#### 无会话运行

XXX

#### 钩子

XXX