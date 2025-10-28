# Tmux 手册

## 名称
tmux -- 终端复用器

## 概述
```
tmux [-2CDlNuVv] [-c shell-command] [-f file] [-L socket-name] [-S socket-path] [-T features] [command [flags]]
```

## 描述
tmux 是一个终端复用器：它允许创建、访问和控制多个终端，从单个屏幕进行管理。tmux 可以从屏幕分离并在后台继续运行，之后可以重新连接。

当 tmux 启动时，它会创建一个带有单个窗口的新会话并在屏幕上显示。屏幕底部的状态行显示当前会话的信息，并用于输入交互式命令。

会话是 tmux 管理的伪终端的集合。每个会话有一个或多个窗口链接到它。窗口占据整个屏幕，并可能被分割成矩形窗格，每个窗格都是一个单独的伪终端（pty(4) 手册页记录了伪终端的技术细节）。任意数量的 tmux 实例可以连接到同一会话，并且同一会话中可以存在任意数量的窗口。一旦所有会话都被终止，tmux 就会退出。

每个会话都是持久的，即使意外断开连接（例如 ssh(1) 连接超时）或有意分离（使用 `C-b d` 键击），也能继续存在。可以使用以下命令重新连接 tmux：

```
$ tmux attach
```

在 tmux 中，会话由客户端在屏幕上显示，所有会话都由单个服务器管理。服务器和每个客户端是通过 /tmp 中的套接字通信的独立进程。

选项如下：

- `-2` 强制 tmux 假定终端支持 256 色。这相当于 -T 256。
- `-C` 以控制模式启动（参见“控制模式”部分）。给定两次 (-CC) 会禁用回显。
- `-c shell-command` 使用默认 shell 执行 shell 命令。如有必要，将启动 tmux 服务器以检索 default-shell 选项。此选项是为了与 sh(1) 兼容，当 tmux 用作登录 shell 时。
- `-D` 不以守护进程方式启动 tmux 服务器。这也关闭 exit-empty 选项。使用 -D 时，不能指定命令。
- `-f file` 指定替代配置文件。默认情况下，tmux 从 /etc/tmux.conf 加载系统配置文件（如果存在），然后查找用户配置文件 ~/.tmux.conf 或 $XDG_CONFIG_HOME/tmux/tmux.conf。

配置文件是一组 tmux 命令，当服务器首次启动时按顺序执行。tmux 在服务器进程启动时加载配置文件一次。可以使用 source-file 命令稍后加载文件。

tmux 会在创建的第一个会话中显示配置文件中命令的任何错误消息，并继续处理配置文件的其余部分。

- `-L socket-name` tmux 在 TMUX_TMPDIR 或 /tmp（如果未设置）下的目录中存储服务器套接字。默认套接字名为 default。此选项允许指定不同的套接字名，允许多个独立的 tmux 服务器运行。与 -S 不同，不需要完整路径：套接字都在 TMUX_TMPDIR 或 /tmp 下的目录 tmux-UID 中创建。tmux-UID 目录由 tmux 创建，不能被世界读取、写入或执行。

如果套接字意外被删除，可以向 tmux 服务器进程发送 SIGUSR1 信号以重新创建它（注意，如果任何父目录缺失，这将失败）。

- `-l` 行为像登录 shell。此标志目前没有效果，是为了与其他 shell 兼容，当使用 tmux 作为登录 shell 时。
- `-N` 即使命令通常会启动服务器也不启动服务器（例如 new-session 或 start-server）。
- `-S socket-path` 指定服务器套接字的完整替代路径。如果指定了 -S，则不使用默认套接字目录，并忽略任何 -L 标志。
- `-T features` 为客户端设置终端功能。这是逗号分隔的功能列表。参见 terminal-features 选项。
- `-u` 即使设置的第一个环境变量 LC_ALL、LC_CTYPE 或 LANG 不包含 "UTF-8" 或 "UTF8"，也向终端写入 UTF-8 输出。
- `-V` 报告 tmux 版本。
- `-v` 请求详细日志记录。日志消息将保存在当前目录中的 tmux-client-PID.log 和 tmux-server-PID.log 文件中，其中 PID 是服务器或客户端进程的 PID。如果指定两次 -v，还会生成一个额外的 tmux-out-PID.log 文件，其中包含 tmux 写入终端的所有内容的副本。

可以向 tmux 服务器进程发送 SIGUSR2 信号，以在打开（如同给定 -v）和关闭之间切换日志记录。

command [flags]
指定一组用于控制 tmux 的命令，如下节所述。如果未指定命令，则假定为 new-session 命令。

## 默认键绑定
tmux 可以通过附加客户端使用前缀键组合进行控制，默认前缀键是 `C-b` (Ctrl-b)，后跟命令键。

默认命令键绑定如下：

- `C-b` 发送前缀键 (C-b) 到应用程序。
- `C-o` 向前旋转当前窗口中的窗格。
- `C-z` 暂停 tmux 客户端。
- `!` 将当前窗格从窗口中分离。
- `"` 将当前窗格分成上下两部分。
- `#` 列出所有粘贴缓冲区。
- `$` 重命名当前会话。
- `%` 将当前窗格分成左右两部分。
- `&` 杀死当前窗口。
- `'` 提示输入窗口索引以选择。
- `(` 切换附加客户端到上一个会话。
- `)` 切换附加客户端到下一个会话。
- `,` 重命名当前窗口。
- `-` 删除最近复制的文本缓冲区。
- `.` 提示输入索引以移动当前窗口。
- `0 到 9` 选择窗口 0 到 9。
- `:` 进入 tmux 命令提示符。
- `;` 移动到之前活动的窗格。
- `=` 从列表中交互式选择要粘贴的缓冲区。
- `?` 列出所有键绑定。
- `D` 选择要分离的客户端。
- `L` 将附加客户端切换回上一个会话。
- `[` 进入复制模式以复制文本或查看历史。
- `]` 粘贴最近复制的文本缓冲区。
- `c` 创建新窗口。
- `d` 分离当前客户端。
- `f` 提示在打开的窗口中搜索文本。
- `i` 显示当前窗口的一些信息。
- `l` 移动到之前选择的窗口。
- `m` 标记当前窗格（参见 select-pane -m）。
- `M` 清除标记的窗格。
- `n` 切换到下一个窗口。
- `o` 选择当前窗口中的下一个窗格。
- `p` 切换到上一个窗口。
- `q` 简要显示窗格索引。
- `r` 强制重绘附加客户端。
- `s` 交互式选择附加客户端的新会话。
- `t` 显示时间。
- `w` 交互式选择当前窗口。
- `x` 杀死当前窗格。
- `z` 切换当前窗格的缩放状态。
- `{` 将当前窗格与上一个窗格交换。
- `}` 将当前窗格与下一个窗格交换。
- `~` 显示 tmux 的先前消息（如果有）。
- `Page Up` 进入复制模式并向上滚动一页。
- `Up, Down, Left, Right` 更改到当前窗格上方、下方、左侧或右侧的窗格。
- `M-1 到 M-5` 按照七种预设布局之一排列窗格：水平均分、垂直均分、主水平、主水平镜像、主垂直、主垂直镜像或平铺。
- `Space` 按照下一个预设布局排列当前窗口。
- `M-n` 移动到带有铃声或活动标记的下一个窗口。
- `M-o` 向后旋转当前窗口中的窗格。
- `M-p` 移动到带有铃声或活动标记的上一个窗口。
- `C-Up, C-Down, C-Left, C-Right` 以一个单元格为步长调整当前窗格的大小。
- `M-Up, M-Down, M-Left, M-Right` 以五个单元格为步长调整当前窗格的大小。

可以使用 bind-key 和 unbind-key 命令更改键绑定。

## 命令解析和执行
tmux 支持大量可用于控制其行为的命令。每个命令都有名称，并可接受零个或多个标志和参数。它们可以与 bind-key 命令绑定，或从 shell 提示符、shell 脚本、配置文件或命令提示符运行。例如，从 shell 提示符、~/.tmux.conf 和绑定到键的相同 set-option 命令可能如下所示：

```
$ tmux set-option -g status-style bg=cyan

set-option -g status-style bg=cyan

bind-key C set-option -g status-style bg=cyan
```

在这里，命令名称是 `set-option`，`-g` 是标志，`status-style` 和 `bg=cyan` 是参数。

tmux 区分命令解析和执行。为了执行命令，tmux 需要将其拆分为名称和参数。这是命令解析。如果从 shell 运行命令，shell 会解析它；从 tmux 内部或从配置文件运行，tmux 会解析它。tmux 解析命令的例子包括：

- 在配置文件中；
- 在命令提示符中输入（参见 command-prompt）；
- 给定给 bind-key；
- 作为 if-shell 或 confirm-before 的参数传递。

为了执行命令，每个客户端都有一个“命令队列”。在启动时，用于配置文件（如 ~/.tmux.conf）的全局命令队列不附加到任何客户端。添加到队列中的已解析命令按顺序执行。一些命令，如 if-shell 和 confirm-before，解析其参数以创建紧跟在自身之后的新命令。这意味着参数可以被解析两次或更多次——一次在父命令（如 if-shell）被解析时，再次在它解析和执行其命令时。像 if-shell、run-shell 和 display-panes 这样的命令会停止队列上后续命令的执行，直到某些事件发生——if-shell 和 run-shell 直到 shell 命令完成，display-panes 直到按下键。例如，以下命令：

```
new-session; new-window
if-shell "true" "split-window"
kill-session
```

将按顺序执行 new-session、new-window、if-shell、shell 命令 true(1)、split-window 和 kill-session。

“命令”部分列出了 tmux 命令及其参数。

## 解析语法
本节描述由 tmux 解析的命令语法，例如在配置文件或命令提示符中。注意，当从 shell 输入命令时，它们由 shell 解析——参见例如 ksh(1) 或 csh(1)。

每个命令以换行符或分号 (;) 结束。由分号分隔的命令一起形成“命令序列”——如果序列中的命令遇到错误，则不执行后续命令。

建议将用作命令分隔符的分号写为单个标记，例如从 sh(1)：

```
$ tmux neww \; splitw
```

或：

```
$ tmux neww ';' splitw
```

或从 tmux 命令提示符：

```
neww ; splitw
```

但是，尾随分号也被解释为命令分隔符，例如在这些 sh(1) 命令中：

```
$ tmux neww\; splitw
```

或：

```
$ tmux 'neww;' splitw
```

正如这些例子所示，从 shell 运行 tmux 时，必须特别注意正确引用分号：

1. 应该被解释为命令分隔符的分号应该根据 shell 约定进行转义。对于 sh(1)，这通常意味着引用（如 `neww ';' splitw'）或转义（如 `neww \\\\; splitw'）。

2. 应该被解释为参数的单个分号或尾随分号应该转义两次：一次根据 shell 约定，第二次为 tmux；例如：

```
$ tmux neww 'foo\\;' bar
$ tmux neww foo\\\\; bar
```

3. 不是单个标记或尾随另一个标记的分号应该只根据 shell 约定转义一次；例如：

```
$ tmux neww 'foo-;-bar'
$ tmux neww foo-\\;-bar
```

注释由未加引号的 # 字符标记——注释后的任何剩余文本都将被忽略，直到行尾。

如果行的最后一个字符是 \，则该行与下一行连接（\ 和换行符完全被删除）。这称为行延续，适用于引号内外和注释内，但不适用于大括号内。

命令参数可以指定为由单引号 (')、双引号 (") 或大括号 ({}) 包围的字符串。当参数包含任何特殊字符时，这是必需的。单引号和双引号字符串不能跨越多行，除非使用行延续。大括号可以跨越多行。

在引号外和双引号内，执行以下替换：

- 以 $ 开头的环境变量被其全局环境中的值替换（参见“全局和会话环境”部分）。

- 前导 ~ 或 ~user 被当前或指定用户的主目录扩展。

- \uXXXX 或 \uXXXXXXXX 被给定的四或八位十六进制数对应的 Unicode 码点替换。

- 当前面有 \（转义）时，以下字符被替换：\e 由转义字符替换；\r 由回车替换；\n 由换行替换；\t 由制表符替换。

- \ooo 被八进制值 ooo 对应的字符替换。需要三个八进制数字，例如 \001。最大的有效字符是 \377。

- 任何前面有 \ 的其他字符都被其本身替换（即 \ 被删除），并且不被视为具有任何特殊含义——因此例如 \; 不会标记命令序列，\$ 不会扩展环境变量。

大括号被解析为配置文件（因此处理诸如 `%if' 的条件），然后转换为字符串。它们旨在避免在将一组 tmux 命令作为参数传递时需要额外的转义（例如传递给 if-shell）。这两个例子产生相同的命令——注意使用 {} 时不需要转义：

```
if-shell true {
    display -p 'brace-dollar-foo: }$foo'
}

if-shell true "display -p 'brace-dollar-foo: }\$foo'"
```

大括号可以包含在大括号内，例如：

```
bind x if-shell "true" {
    if-shell "true" {
        display "true!"
    }
}
```

可以通过使用语法 `name=value` 设置环境变量，例如 `HOME=/home/user`。在解析期间设置的变量被添加到全局环境。可以使用 `%hidden` 设置隐藏变量，例如：

```
%hidden MYVAR=42
```

隐藏变量不会传递给由 tmux 创建的进程环境。参见“全局和会话环境”部分。

命令可以通过用 `%if`、`%elif`、`%else` 和 `%endif` 包围来有条件地解析。`%if` 和 `%elif` 的参数作为格式扩展（参见“格式”），如果它计算为假（零或空），则忽略后续文本，直到关闭的 `%elif`、`%else` 或 `%endif`。例如：

```
%if "#{==:#{host},myhost}"
set -g status-style bg=red
%elif "#{==:#{host},myotherhost}"
set -g status-style bg=green
%else
set -g status-style bg=blue
%endif
```

如果在 `myhost` 上运行，将状态行更改为红色；如果在 `myotherhost` 上运行，更改为绿色；如果在其他主机上运行，更改为蓝色。条件可以在一行上给出，例如：

```
%if #{==:#{host},myhost} set -g status-style bg=red %endif
```

## 命令
本节描述 tmux 支持的命令。大多数命令接受可选的 `-t`（有时是 `-s`）参数，其中包括 target-client、target-session、target-window 或 target-pane。这些指定命令应该影响的客户端、会话、窗口或窗格。

target-client 应该是客户端的名称，通常是客户端连接到的 pty(4) 文件，例如 /dev/ttyp1 或 ttyp1 用于连接到 /dev/ttyp1 的客户端。如果未指定客户端，tmux 会尝试确定当前正在使用的客户端；如果失败，则报告错误。可以使用 list-clients 命令列出客户端。

target-session 按以下顺序尝试：

1. 以 $ 为前缀的会话 ID。
2. 会话的准确名称（如 list-sessions 命令所列）。
3. 会话名称的开头，例如 `mysess` 将匹配名为 `mysession` 的会话。
4. 与会话名称匹配的 fnmatch(3) 模式。

如果会话名称以 `=` 为前缀，则只接受完全匹配（因此 `=mysess` 只会完全匹配 `mysess`，而不是 `mysession`）。

如果找到单个会话，则将其用作目标会话；多个匹配会产生错误。如果省略会话，则使用当前会话（如果可用）；如果没有当前会话可用，则选择最近使用的会话。

target-window（或 src-window 或 dst-window）以 session:window 的形式指定窗口。session 遵循与 target-session 相同的规则，window 按以下顺序查找：

1. 特殊标记，如下所列。
2. 窗口索引，例如 `mysession:1` 是会话 `mysession` 中的窗口 1。
3. 窗口 ID，例如 @1。
4. 准确的窗口名称，例如 `mysession:mywindow`。
5. 窗口名称的开头，例如 `mysession:mywin`。
6. 与窗口名称匹配的 fnmatch(3) 模式。

与会话一样，`=` 前缀将只进行完全匹配。空的窗口名称在适当时指定下一个未使用的索引（例如 new-window 和 link-window 命令），否则选择会话中的当前窗口。

以下特殊标记可用于指示特定窗口。每个都有单字符替代形式。

标记              含义
{start}       ^    最低编号的窗口
{end}         $    最高编号的窗口
{last}        !    最后（之前当前的）窗口
{next}        +    按数字的下一个窗口
{previous}    -    按数字的上一个窗口

target-pane（或 src-pane 或 dst-pane）可以是窗格 ID 或采用与 target-window 类似的格式，但可选地添加一个句点，后跟窗格索引或窗格 ID，例如：`mysession:mywindow.1`。如果省略窗格索引，则使用指定窗口中的当前活动窗格。以下特殊标记可用于窗格索引：

标记                  含义
{last}            !    最后（之前活动的）窗格
{next}            +    按数字的下一个窗格
{previous}        -    按数字的上一个窗格
{top}                  顶部窗格
{bottom}               底部窗格
{left}                 最左边的窗格
{right}                最右边的窗格
{top-left}             左上角窗格
{top-right}            右上角窗格
{bottom-left}          左下角窗格
{bottom-right}         右下角窗格
{up-of}                活动窗格上方的窗格
{down-of}              活动窗格下方的窗格
{left-of}              活动窗格左侧的窗格
{right-of}             活动窗格右侧的窗格

标记 `+` 和 `-` 可以跟随偏移量，例如：

```
select-window -t:+2
```

此外，target-session、target-window 或 target-pane 可以完全由标记 `{mouse}`（替代形式 `=`）组成，以指定最近鼠标事件发生的会话、窗口或窗格（参见“鼠标支持”部分），或 `{marked}`（替代形式 `~`）以指定标记的窗格（参见 select-pane -m）。

会话、窗口和窗格都用唯一的 ID 编号；会话 ID 以 `$` 为前缀，窗口以 `@` 为前缀，窗格以 `%` 为前缀。这些 ID 是唯一的，并且在 tmux 服务器中会话、窗口或窗格的生命周期内保持不变。窗格 ID 作为 TMUX_PANE 环境变量传递给窗格的子进程。可以使用 `session_id`、`window_id` 或 `pane_id` 格式（参见“格式”部分）以及 display-message、list-sessions、list-windows 或 list-panes 命令显示 ID。

shell-command 参数是 sh(1) 命令。这可以是传递给 shell 的单个参数，例如：

```
new-window 'vi ~/.tmux.conf'
```

将运行：

```
/bin/sh -c 'vi ~/.tmux.conf'
```

此外，new-window、new-session、split-window、respawn-window 和 respawn-pane 命令允许将 shell-command 作为多个参数给出并直接执行（不带 `sh -c`）。这可以避免 shell 引用问题。例如：

```
$ tmux new-window vi ~/.tmux.conf
```

将直接运行 vi(1) 而不调用 shell。

command [argument ...] 指的是 tmux 命令，可以分别传递命令和参数，例如：

```
bind-key F1 set-option status off
```

或者作为单个字符串参数在 .tmux.conf 中传递，例如：

```
bind-key F1 { set-option status off }
```

示例 tmux 命令包括：

```
refresh-client -t/dev/ttyp2

rename-session -tfirst newname

set-option -wt:0 monitor-activity on

new-window ; split-window -d

bind-key R source-file ~/.tmux.conf \; \
        display-message "source-file done"
```

或者从 sh(1)：

```
$ tmux kill-window -t :1

$ tmux new-window \; split-window -d

$ tmux new-session -d 'vi ~/.tmux.conf' \; split-window -d \; attach
```

## 客户端和会话
tmux 服务器管理客户端、会话、窗口和窗格。客户端附加到会话以与它们交互，要么在使用 new-session 命令创建时，要么稍后使用 attach-session 命令。每个会话有一个或多个窗口链接到它。窗口可以链接到多个会话，并由一个或多个窗格组成，每个窗格包含一个伪终端。有关创建、链接和以其他方式操作窗口的命令，请参阅“窗口和窗格”部分。

以下命令可用于管理客户端和会话：

attach-session [-dErx] [-c working-directory] [-f flags] [-t target-session]
                (别名: attach)
如果从 tmux 外部运行，将附加到当前终端中的 target-session。target-session 必须已经存在——要创建新会话，请参见 new-session 命令（使用 -A 创建或附加）。如果从内部运行，则将当前附加的会话切换到 target-session。如果指定 -d，则分离附加到会话的任何其他客户端。如果给定 -x，则向客户端的父进程发送 SIGHUP 以及分离客户端，通常导致其退出。-f 设置逗号分隔的客户端标志列表。标志包括：

active-pane
        客户端具有独立的活动窗格

ignore-size
        客户端不影响其他客户端的大小

no-output
        客户端在控制模式下不接收窗格输出

pause-after=seconds
        在控制模式下，一旦窗格在 seconds 秒后落后，输出将暂停

read-only
        客户端是只读的

wait-exit
        在控制模式下，在退出前等待空行输入

前导的 `!` 会在客户端已经附加时关闭标志。-r 是 -f read-only,ignore-size 的别名。当客户端是只读时，只有绑定到 detach-client 或 switch-client 命令的键才有效果。具有 active-pane 标志的客户端允许独立于没有该标志的客户端使用的窗口的活动窗格选择活动窗格。这只影响光标位置和从客户端发出的命令；其他功能（如钩子和样式）继续使用窗口的活动窗格。

如果未启动服务器，attach-session 将尝试启动它；这将失败，除非在配置文件中创建了会话。

attach-session 的 target-session 规则略有调整：如果 tmux 需要选择最近使用的会话，它将优先选择最近使用的未附加会话。

-c 将会话工作目录（用于新窗口）设置为 working-directory。

如果使用 -E，则不应用 update-environment 选项。

detach-client [-aP] [-E shell-command] [-s target-session] [-t target-client]
                (别名: detach)
如果绑定到键，则分离当前客户端，或指定的客户端（使用 -t），或分离指定会话（使用 -s）的所有客户端。-a 选项杀死除使用 -t 给定的客户端之外的所有客户端。如果给定 -P，则向客户端的父进程发送 SIGHUP，通常导致其退出。使用 -E，运行 shell-command 替换客户端。

has-session [-t target-session]
                (别名: has)
如果指定的会话不存在，则报告错误并以 1 退出。如果存在，则以 0 退出。

kill-server
杀死 tmux 服务器和客户端并销毁所有会话。

kill-session [-aC] [-t target-session]
销毁给定的会话，关闭链接到它的任何窗口和其他会话，并分离附加到它的所有客户端。如果给定 -a，则杀死除指定会话之外的所有会话。-C 标志清除链接到会话的所有窗口中的警报（铃声、活动或静音）。

list-clients [-F format] [-f filter] [-t target-session]
                (别名: lsc)
列出附加到服务器的所有客户端。-F 指定每行的格式，-f 指定过滤器。仅显示过滤器为真的客户端。参见“格式”部分。如果指定 target-session，则仅列出连接到该会话的客户端。

list-commands [-F format] [command]
                (别名: lscm)
列出 command 的语法，或如果省略，则列出 tmux 支持的所有命令。

list-sessions [-F format] [-f filter]
                (别名: ls)
列出服务器管理的所有会话。-F 指定每行的格式，-f 指定过滤器。仅显示过滤器为真的会话。参见“格式”部分。

lock-client [-t target-client]
                (别名: lockc)
锁定 target-client，参见 lock-server 命令。

lock-session [-t target-session]
                (别名: locks)
锁定附加到 target-session 的所有客户端。

new-session [-AdDEPX] [-c start-directory] [-e environment] [-f flags] [-F format] [-n window-name] [-s session-name] [-t group-name] [-x width] [-y height] [shell-command]
                (别名: new)
创建一个名为 session-name 的新会话。

新会话附加到当前终端，除非给定 -d。window-name 和 shell-command 是初始窗口的名称和要执行的 shell 命令。使用 -d，初始大小来自全局 default-size 选项；-x 和 -y 可用于指定不同的大小。`-` 使用当前客户端的大小（如果有的话）。如果给定 -x 或 -y，则为会话设置 default-size 选项。-f 设置逗号分隔的客户端标志列表（参见 attach-session）。

如果从终端运行，任何 termios(4) 特殊字符都会被保存并在新会话的新窗口中使用。

-A 标志使 new-session 行为像 attach-session，如果 session-name 已经存在；如果给定 -A，则 -D 行为像 attach-session 的 -d，-X 行为像 attach-session 的 -x。

如果给定 -t，则指定会话组。同一组中的会话共享同一组窗口——新窗口链接到组中的所有会话，任何关闭的窗口都会从所有会话中删除。当前和上一个窗口以及任何会话选项保持独立，组中的任何会话都可以被杀死而不会影响其他会话。group-name 参数可以是：

1. 现有组的名称，在这种情况下，新会话被添加到该组；
2. 现有会话的名称——新会话被添加到与该会话相同的组，必要时创建一个新组；
3. 仅包含新会话的新组的名称。

如果使用 -t，则 -n 和 shell-command 无效。

-P 选项在创建新会话后打印有关新会话的信息。默认情况下，它使用格式 `#{session_name}:'，但可以使用 -F 指定不同的格式。

如果使用 -E，则不应用 update-environment 选项。-e 采用 `VARIABLE=value` 形式，并为新创建的会话设置环境变量；可以指定多次。

refresh-client [-cDLRSU] [-A pane:state] [-B name:what:format] [-C size] [-f flags] [-l [target-pane]] [-r pane:report] [-t target-client] [adjustment]
                (别名: refresh)
如果绑定到键，则刷新当前客户端，或指定的客户端（使用 -t）。如果指定 -S，则仅更新客户端的状态行。

-U、-D、-L -R 和 -c 标志允许更改大于客户端的窗口的可见部分。-U 将可见部分向上移动 adjustment 行，-D 向下移动，-L 向左移动 adjustment 列，-R 向右移动。-c 返回自动跟踪光标。如果省略 adjustment，则使用 1。请注意，可见位置是客户端的属性，而不是窗口的属性，在附加会话中更改当前窗口将重置它。

-C 为控制模式客户端或控制模式客户端的窗口设置宽度和高度，size 必须是 `widthxheight` 或 `window ID:widthxheight` 之一，例如 `80x24` 或 `@0:80x24`。-A 允许控制模式客户端触发窗格上的操作。参数是窗格 ID（带前导 `%`），冒号，然后是 `on`、`off`、`continue` 或 `pause` 之一。如果为 `off`，tmux 将不会向客户端发送窗格的输出，如果所有客户端都关闭了窗格，将停止从窗格读取。如果为 `continue`，tmux 将在暂停时（手动或使用 pause-after 标志）返回向窗格发送输出。如果为 `pause`，tmux 将暂停窗格。-A 可以多次给出，用于不同的窗格。

-B 为控制模式客户端设置格式订阅。参数由冒号分隔为三个项目：name 是订阅的名称；what 是要订阅的项目类型；format 是格式。添加订阅后，格式更改将使用 %subscription-changed 通知报告，最多每秒一次。如果只给出名称，则删除订阅。what 可以为空，以仅检查附加会话的格式，或为以下之一：窗格 ID（如 `%0`）；`%*` 表示附加会话中的所有窗格；窗口 ID（如 `@0`）；或 `@*` 表示附加会话中的所有窗口。

-f 设置逗号分隔的客户端标志列表，参见 attach-session。-r 允许控制模式客户端通过报告提供窗格信息（如 OSC 10 的响应）。参数是窗格 ID（带前导 `%`），冒号，然后是报告转义序列。

-l 使用 xterm(1) 转义序列从客户端请求剪贴板。如果给定 target-pane，则剪贴板被发送（以编码形式），否则存储在新的粘贴缓冲区中。

-L、-R、-U 和 -D 将窗口的可见部分向左、向右、向上或向下移动 adjustment，如果窗口大于客户端。-c 重置位置以跟随光标。参见 window-size 选项。

rename-session [-t target-session] new-name
                (别名: rename)
将会话重命名为 new-name。

server-access [-adlrw] [user]
更改 user 的访问或读/写权限。运行 tmux 服务器的用户（其所有者）和根用户无法更改，始终允许访问。

-a 和 -d 用于为指定用户授予或撤销访问权限。如果用户已经附加，-d 标志会导致其客户端分离。

-r 和 -w 更改 user 的权限：-r 使其客户端只读，-w 可写。-l 列出当前访问权限。

默认情况下，访问列表为空，tmux 创建的套接字具有文件系统权限，防止除所有者（和根用户）之外的任何用户访问。这些权限必须手动更改。应格外小心，不要允许不受信任的用户访问，即使是只读。

show-messages [-JT] [-t target-client]
                (别名: showmsgs)
显示服务器消息或信息。消息被存储，最多达到由 message-limit 服务器选项设置的限制。-J 和 -T 显示有关作业和终端的调试信息。

source-file [-Fnqv] [-t target-pane] path ...
                (别名: source)
从一个或多个由 path 指定的文件执行命令（可以是 glob(7) 模式）。如果存在 -F，则 path 被扩展为格式。如果给定 -q，则如果 path 不存在，不会返回错误。使用 -n，文件被解析但不执行命令。-v 显示解析的命令和行号（如果可能）。

start-server
                (别名: start)
启动 tmux 服务器（如果尚未运行），而不创建任何会话。

请注意，默认情况下，当没有会话时，tmux 服务器将退出，这只有在 ~/.tmux.conf 中创建会话、关闭 exit-empty 或作为同一命令序列的一部分运行其他命令时才有用。例如：

```
$ tmux start \; show -g
```

suspend-client [-t target-client]
                (别名: suspendc)
通过发送 SIGTSTP（tty stop）暂停客户端。

switch-client [-ElnprZ] [-c target-client] [-t target-session] [-T key-table]
                (别名: switchc)
将 target-client 的当前会话切换到 target-session。作为一种特殊情况，-t 可以引用窗格（包含 `:`、`.` 或 `%` 的目标），以更改会话、窗口和窗格。在这种情况下，-Z 在窗口缩放时保持缩放。如果使用 -l、-n 或 -p，则客户端分别移动到最后一个、下一个或上一个会话。-r 切换客户端的只读和 ignore-size 标志（参见 attach-session 命令）。

如果使用 -E，则不应用 update-environment 选项。

-T 设置客户端的键表；客户端的下一个键将从 key-table 解释。这可用于配置多个前缀键，或将命令绑定到键序列。例如，使输入 `abc` 运行 list-keys 命令：

```
bind-key -Ttable2 c list-keys
bind-key -Ttable1 b switch-client -Ttable2
bind-key -Troot   a switch-client -Ttable1
```

## 窗口和窗格
tmux 显示的每个窗口可以被分割成一个或多个窗格；每个窗格占据显示区域的某个部分，并是一个独立的终端。可以使用 split-window 命令将窗口分割成窗格。窗口可以水平（使用 -h 标志）或垂直分割。可以使用 resize-pane 命令调整窗格大小（默认绑定到 `C-Up`、`C-Down`、`C-Left` 和 `C-Right`），可以使用 select-pane 命令更改当前窗格，rotate-window 和 swap-pane 命令可用于交换窗格而不更改其位置。窗格按其创建顺序从零开始编号。

默认情况下，tmux 窗格允许直接访问窗格中包含的终端。窗格也可以置于以下几种模式之一：

- 复制模式，允许将窗口或其历史的一部分复制到粘贴缓冲区中，以便稍后插入到另一个窗口中。此模式通过 copy-mode 命令进入，默认绑定到 `[`。复制的文本可以通过 paste-buffer 命令粘贴，默认绑定到 `]`。

- 视图模式，类似于复制模式，但从键绑定执行产生输出的命令（如 list-keys）时进入。

- 选择模式，允许从列表中选择项目。这可以是客户端、会话或窗口或窗格，或缓冲区。此模式通过 choose-buffer、choose-client 和 choose-tree 命令进入。

在复制模式中，在窗格右上角显示一个指示器，其中包含当前位置和历史中的行数。

使用 send-keys 命令的 -X 标志将命令发送到复制模式。按下键时，复制模式自动使用两个键表之一，这取决于 mode-keys 选项：emacs 的 copy-mode，或 vi 的 copy-mode-vi。可以使用 list-keys 命令查看键表。

复制模式支持以下命令：

append-selection
        将选择附加到顶部粘贴缓冲区。

append-selection-and-cancel (vi: A)
        将选择附加到顶部粘贴缓冲区并退出复制模式。

back-to-indentation (vi: ^) (emacs: M-m)
        将光标移回缩进。

begin-selection (vi: Space) (emacs: C-Space)
        开始选择。

bottom-line (vi: L)
        移动到最底行。

cancel (vi: q) (emacs: Escape)
        退出复制模式。

clear-selection (vi: Escape) (emacs: C-g)
        清除当前选择。

copy-end-of-line [prefix]
        从光标位置复制到行尾。prefix 用于命名新的粘贴缓冲区。

copy-end-of-line-and-cancel [prefix]
        从光标位置复制并退出复制模式。

copy-pipe-end-of-line [command] [prefix]
        从光标位置复制到行尾并将文本通过管道传输到 command。prefix 用于命名新的粘贴缓冲区。

copy-pipe-end-of-line-and-cancel [command] [prefix]
        与 copy-pipe-end-of-line 相同，但也退出复制模式。

copy-line [prefix]
        复制整行。

copy-line-and-cancel [prefix]
        复制整行并退出复制模式。

copy-pipe-line [command] [prefix]
        复制整行并将文本通过管道传输到 command。prefix 用于命名新的粘贴缓冲区。

copy-pipe-line-and-cancel [command] [prefix]
        与 copy-pipe-line 相同，但也退出复制模式。

copy-pipe [command] [prefix]
        复制选择，清除它并将文本通过管道传输到 command。prefix 用于命名新的粘贴缓冲区。

copy-pipe-no-clear [command] [prefix]
        与 copy-pipe 相同，但不清除选择。

copy-pipe-and-cancel [command] [prefix]
        与 copy-pipe 相同，但也退出复制模式。

copy-selection [prefix]
        复制当前选择。

copy-selection-no-clear [prefix]
        与 copy-selection 相同，但不清除选择。

copy-selection-and-cancel [prefix] (vi: Enter) (emacs: M-w)
        复制当前选择并退出复制模式。

cursor-down (vi: j) (emacs: Down)
        将光标向下移动。

cursor-down-and-cancel
        与 cursor-down 相同，但如果到达底部则退出复制模式。

cursor-left (vi: h) (emacs: Left)
        将光标向左移动。

cursor-right (vi: l) (emacs: Right)
        将光标向右移动。

cursor-up (vi: k) (emacs: Up)
        将光标向上移动。

end-of-line (vi: $) (emacs: C-e)
        将光标移动到行尾。

goto-line line (vi: :) (emacs: g)
        将光标移动到特定行。

halfpage-down (vi: C-d) (emacs: M-Down)
        向下滚动半页。

halfpage-down-and-cancel
        与 halfpage-down 相同，但如果到达底部则退出复制模式。

halfpage-up (vi: C-u) (emacs: M-Up)
        向上滚动半页。

history-bottom (vi: G) (emacs: M->)
        滚动到历史底部。

history-top (vi: g) (emacs: M-<)
        滚动到历史顶部。

jump-again (vi: ;) (emacs: ;)
        重复上次跳转。

jump-backward to (vi: F) (emacs: F)
        向后跳转到指定文本。

jump-forward to (vi: f) (emacs: f)
        向前跳转到指定文本。

jump-reverse (vi: ,) (emacs: ,)
        以相反方向重复上次跳转（向前变为向后，向后变为向前）。

jump-to-backward to (vi: T)
        向后跳转，但少一个字符，将光标放在目标之后的字符上。

jump-to-forward to (vi: t)
        向前跳转，但少一个字符，将光标放在目标之前的字符上。

jump-to-mark (vi: M-x) (emacs: M-x)
        跳转到最后一个标记。

middle-line (vi: M) (emacs: M-r)
        移动到中间行。

next-matching-bracket (vi: %) (emacs: M-C-f)
        移动到下一个匹配括号。

next-paragraph (vi: }) (emacs: M-})
        移动到下一个段落。

next-prompt [-o]
        移动到下一个提示。

next-word (vi: w)
        移动到下一个单词。

next-word-end (vi: e) (emacs: M-f)
        移动到下一个单词的末尾。

next-space (vi: W)
        与 next-word 相同，但使用空格作为单词分隔符。

next-space-end (vi: E)
        与 next-word-end 相同，但使用空格作为单词分隔符。

other-end (vi: o)
        切换选择的光标位置。

page-down (vi: C-f) (emacs: PageDown)
        向下滚动一页。

page-down-and-cancel
        与 page-down 相同，但如果到达底部则退出复制模式。

page-up (vi: C-b) (emacs: PageUp)
        向上滚动一页。

pipe [command]
        将选定的文本通过管道传输到 command 并清除选择。

pipe-no-clear [command]
        与 pipe 相同，但不清除选择。

pipe-and-cancel [command] [prefix]
        与 pipe 相同，但也退出复制模式。

previous-matching-bracket (emacs: M-C-b)
        移动到上一个匹配括号。

previous-paragraph (vi: {) (emacs: M-{)
        移动到上一个段落。

previous-prompt [-o]
        移动到上一个提示。

previous-word (vi: b) (emacs: M-b)
        移动到上一个单词。

previous-space (vi: B)
        与 previous-word 相同，但使用空格作为单词分隔符。

rectangle-on
        打开矩形选择模式。

rectangle-off
        关闭矩形选择模式。

rectangle-toggle (vi: v) (emacs: R)
        切换矩形选择模式。

refresh-from-pane (vi: r) (emacs: r)
        从窗格刷新内容。

scroll-bottom
        向上滚动，直到当前行位于底部，同时保持光标在该行上。

scroll-down (vi: C-e) (emacs: C-Down)
        向下滚动。

scroll-down-and-cancel
        与 scroll-down 相同，但如果光标到达底部则退出复制模式。

scroll-middle (vi: z)
        滚动，使当前行成为中间行，同时保持光标在该行上。

scroll-top
        向下滚动，直到当前行位于顶部，同时保持光标在该行上。

scroll-up (vi: C-y) (emacs: C-Up)
        向上滚动。

search-again (vi: n) (emacs: n)
        重复上次搜索。

search-backward text (vi: ?)
        向后搜索指定文本。

search-backward-incremental text (emacs: C-r)
        向后增量搜索指定文本。应与 command-prompt 命令的 -i 标志一起使用。

search-backward-text text
        向后搜索指定的纯文本。

search-forward text (vi: /)
        向前搜索指定文本。

search-forward-incremental text (emacs: C-s)
        向前增量搜索指定文本。应与 command-prompt 命令的 -i 标志一起使用。

search-forward-text text
        向前搜索指定的纯文本。

search-reverse (vi: N) (emacs: N)
        以相反方向重复上次搜索（向前变为向后，向后变为向前）。

select-line (vi: V)
        选择当前行。

select-word
        选择当前单词。

set-mark (vi: X) (emacs: X)
        标记当前行。

start-of-line (vi: 0) (emacs: C-a)
        将光标移动到行首。

stop-selection
        停止选择而不清除当前选择。

toggle-position (vi: P) (emacs: P)
        切换右上角位置指示器的可见性。

top-line (vi: H) (emacs: M-R)
        移动到顶行。

搜索命令有几种变体：`search-forward` 和 `search-backward` 搜索正则表达式；`-text` 变体搜索纯文本而不是正则表达式；`-incremental` 执行增量搜索，并应与 command-prompt 命令的 -i 标志一起使用。`search-again` 重复上次搜索，`search-reverse` 执行相同操作但反转方向（向前变为向后，向后变为向前）。

`next-prompt` 和 `previous-prompt` 在 shell 提示符之间移动，但需要 shell 发出转义序列 (\033]133;A\033\\) 来告诉 tmux 提示符的位置；如果 shell 不这样做，这些命令将不执行任何操作。-o 标志跳转到命令输出的开头而不是 shell 提示符。

复制命令可以带有可选的缓冲区前缀参数，用于生成缓冲区名称（默认是 `buffer`，因此缓冲区命名为 `buffer0`、`buffer1` 等）。管道命令带有命令参数，这是选定文本通过管道传输到的命令。`copy-pipe` 变体也复制选择。`-and-cancel` 变体的一些命令在完成（对于复制命令）或光标到达底部（对于滚动命令）后退出复制模式。`-no-clear` 变体不清除选择。

下一个和上一个单词键跳过空白字符，并将连续的单词分隔符或字母运行视为单词。单词分隔符可以通过 word-separators 会话选项自定义。下一个单词移动到下一个单词的开头，下一个单词末尾移动到下一个单词的末尾，上一个单词移动到上一个单词的开头。三个下一个和上一个空格键的工作方式类似，但使用空格作为单词分隔符。将 word-separators 设置为空字符串使下一个/上一个单词等同于下一个/上一个空格。

跳转命令允许在行内快速移动。例如，输入 `f` 后跟 `/` 将光标移动到当前行上的下一个 `/` 字符。`;` 将跳转到下一个出现的位置。

复制模式中的命令可以带有可选的重复计数前缀。对于 vi 键绑定，使用数字键输入前缀；对于 emacs，Alt（meta）键和数字开始前缀输入。

copy-mode 命令的概要如下：

copy-mode [-deHMqu] [-s src-pane] [-t target-pane]
        进入复制模式。-u 在进入后也向上滚动一页，-d 在已经处于复制模式时向下滚动一页。-M 开始鼠标拖动（仅在绑定到鼠标键绑定时有效，请参见“鼠标支持”）。-H 隐藏右上角的位置指示器。-q 取消复制模式和任何其他模式。-s 从 src-pane 而不是 target-pane 复制。

        -e 指定滚动到历史底部（到可见屏幕）应退出复制模式。在复制模式中，按除用于滚动的键之外的任何键都会禁用此行为。这是为了允许快速滚动窗格的历史，例如：

```
bind PageUp copy-mode -eu
bind PageDown copy-mode -ed
```

一些预设的窗格排列可用，这些称为布局。可以使用 select-layout 命令选择这些布局，或使用 next-layout（默认绑定到 `Space`）循环使用；选择布局后，可以正常移动和调整其中的窗格。

支持以下布局：

even-horizontal
        窗格从左到右均匀分布在窗口中。

even-vertical
        窗格从上到下均匀分布。

main-horizontal
        一个大（主）窗格显示在窗口顶部，其余窗格从左到右分布在底部剩余空间中。使用 main-pane-height 窗口选项指定顶部窗格的高度。

main-horizontal-mirrored
        与 main-horizontal 相同，但镜像，因此主窗格位于窗口底部。

main-vertical
        一个大（主）窗格显示在窗口左侧，其余窗格从上到下分布在右侧剩余空间中。使用 main-pane-width 窗口选项指定左侧窗格的宽度。

main-vertical-mirrored
        与 main-vertical 相同，但镜像，因此主窗格位于窗口右侧。

tiled   窗格在窗口中尽可能均匀地分布，按行和列排列。

此外，select-layout 可用于应用以前使用的布局——list-windows 命令以适合与 select-layout 一起使用的格式显示每个窗口的布局。例如：

```
$ tmux list-windows
0: ksh [159x48]
    layout: bb62,159x48,0,0{79x48,0,0,79x48,80,0}
$ tmux select-layout 'bb62,159x48,0,0{79x48,0,0,79x48,80,0}'
```

tmux 自动调整布局以适应当前窗口大小。请注意，如果窗口中的窗格数多于从中定义布局的窗口中的窗格数，则无法将布局应用于窗口。

与窗口和窗格相关的命令如下：

break-pane [-abdP] [-F format] [-n window-name] [-s src-pane] [-t dst-window]
                (别名: breakp)
将 src-pane 从其包含的窗口中分离，使其成为 dst-window 中的唯一窗格。使用 -a 或 -b，窗口移动到现有窗口之后或之前的下一个索引（必要时移动现有窗口）。如果给定 -d，则新窗口不会成为当前窗口。-P 选项在创建新窗口后打印有关新窗口的信息。默认情况下，它使用格式 `#{session_name}:#{window_index}.#{pane_index}`，但可以使用 -F 指定不同的格式。

capture-pane [-aAepPqCJN] [-b buffer-name] [-E end-line] [-S start-line] [-t target-pane]
                (别名: capturep)
捕获窗格的内容。如果给定 -p，则输出转到 stdout，否则转到使用 -b 指定的缓冲区或新缓冲区（如果省略）。如果给定 -a，则使用备用屏幕，并且历史不可访问。如果不存在备用屏幕，则返回错误，除非给定 -q。如果给定 -e，则输出包括文本和背景属性的转义序列。-C 还将不可打印字符转义为八进制 \xxx。-T 忽略不包含字符的尾随位置。-N 保留每行末尾的尾随空格，-J 保留尾随空格并连接任何换行的行；-J 隐含 -T。-P 仅捕获窗格已接收的任何输出，这些输出是尚未完成的转义序列的开头。

-S 和 -E 指定起始和结束行号，0 是可见窗格的第一行，负数是历史中的行。`-` 到 -S 是历史的开始，到 -E 是可见窗格的结束。默认情况下，仅捕获窗格的可见内容。

choose-client [-NrZ] [-F format] [-f filter] [-K key-format] [-O sort-order] [-t target-pane] [template]
将窗格置于客户端模式，允许从列表中交互式选择客户端。每个客户端显示在一行上。左侧显示快捷键，允许立即选择，或者可以导航列表并选择或以其他方式操作项目。-Z 缩放窗格。在客户端模式中可以使用以下键：

        键    功能
        Enter  选择选定的客户端
        Up     选择上一个客户端
        Down   选择下一个客户端
        C-s    按名称搜索
        n      向前重复上次搜索
        N      向后重复上次搜索
        t      切换客户端是否被标记
        T      取消标记所有客户端
        C-t    标记所有客户端
        d      分离选定的客户端
        D      分离标记的客户端
        x      分离并挂起选定的客户端
        X      分离并挂起标记的客户端
        z      暂停选定的客户端
        Z      暂停标记的客户端
        f      输入格式以过滤项目
        O      更改排序字段
        r      反向排序
        v      切换预览
        q      退出模式

选择客户端后，`%%` 在 template 中被客户端名称替换，并将结果作为命令执行。如果未给定 template，则使用 "detach-client -t '%%'"。

-O 指定初始排序字段：`name`、`size`、`creation`（时间）或 `activity`（时间）之一。-r 反向排序。-f 指定初始过滤器：过滤器是格式——如果计算结果为零，则列表中的项目不显示，否则显示。如果过滤器会导致空列表，则忽略。-F 指定列表中每个项目的格式，-K 指定每个快捷键的格式；两者都为每行计算一次。-N 开始时不显示预览。此命令仅在至少有一个客户端附加时有效。

choose-tree [-GNrswZ] [-F format] [-f filter] [-K key-format] [-O sort-order] [-t target-pane] [template]
将窗格置于树模式，允许从树中交互式选择会话、窗口或窗格。每个会话、窗口或窗格显示在一行上。左侧显示快捷键，允许立即选择，或者可以导航树并选择或以其他方式操作项目。-s 以会话折叠开始，-w 以窗口折叠开始。-Z 缩放窗格。在树模式中可以使用以下键：

        键    功能
        Enter  选择选定的项目
        Up     选择上一个项目
        Down   选择下一个项目
        +      展开选定的项目
        -      折叠选定的项目
        M-+    展开所有项目
        M--    折叠所有项目
        x      杀死选定的项目
        X      杀死标记的项目
        <      向左滚动预览列表
        >      向右滚动预览列表
        C-s    按名称搜索
        m      设置标记的窗格
        M      清除标记的窗格
        n      向前重复上次搜索
        N      向后重复上次搜索
        t      切换项目是否被标记
        T      取消标记所有项目
        C-t    标记所有项目
        :      为每个标记的项目运行命令
        f      输入格式以过滤项目
        H      跳转到起始窗格
        O      更改排序字段
        r      反向排序
        v      切换预览
        q      退出模式

选择会话、窗口或窗格后，`%%` 和所有 `%1` 实例在 template 中被目标替换，并将结果作为命令执行。如果未给定 template，则使用 "switch-client -t '%%'"。

-O 指定初始排序字段：`index`、`name` 或 `time`（活动）之一。-r 反向排序。-f 指定初始过滤器：过滤器是格式——如果计算结果为零，则列表中的项目不显示，否则显示。如果过滤器会导致空列表，则忽略。-F 指定树中每个项目的格式，-K 指定每个快捷键的格式；两者都为每行计算一次。-N 开始时不显示预览。-G 包括会话组中的所有会话，而不仅仅是第一个。此命令仅在至少有一个客户端附加时有效。

customize-mode [-NZ] [-F format] [-f filter] [-t target-pane] [template]
将窗格置于自定义模式，允许从列表中浏览和修改选项和键绑定。列表中的选项值显示为当前窗口中活动窗格的值。-Z 缩放窗格。在自定义模式中可以使用以下键：

        键    功能
        Enter  设置窗格、窗口、会话或全局选项值
        Up     选择上一个项目
        Down   选择下一个项目
        +      展开选定的项目
        -      折叠选定的项目
        M-+    展开所有项目
        M--    折叠所有项目
        s      设置选项值或键属性
        S      设置全局选项值
        w      设置窗口选项值，如果选项是窗格和窗口
        d      将选项或键设置为默认值
        D      将标记的选项和标记的键设置为默认值
        u      取消设置选项（设置为默认值，如果是全局的）或取消绑定键
        U      取消设置标记的选项和取消绑定标记的键
        C-s    按名称搜索
        n      向前重复上次搜索
        N      向后重复上次搜索
        t      切换项目是否被标记
        T      取消标记所有项目
        C-t    标记所有项目
        f      输入格式以过滤项目
        v      切换选项信息
        q      退出模式

-f 指定初始过滤器：过滤器是格式——如果计算结果为零，则列表中的项目不显示，否则显示。如果过滤器会导致空列表，则忽略。-F 指定树中每个项目的格式。-N 开始时不显示选项信息。此命令仅在至少有一个客户端附加时有效。

display-panes [-bN] [-d duration] [-t target-client] [template]
                (别名: displayp)
显示 target-client 显示的每个窗格的可见指示器。参见 display-panes-colour 和 display-panes-active-colour 会话选项。指示器在按下键（除非给定 -N）或 duration 毫秒过去后关闭。如果未给定 -d，则使用 display-panes-time。duration 为零意味着指示器保持直到按下键。指示器在屏幕上时，可以使用 `0` 到 `9` 键选择窗格，这将导致 template 作为命令执行，`%%` 被窗格 ID 替换。默认 template 是 "select-pane -t '%%'"。使用 -b，其他命令不会被阻止运行，直到指示器关闭。

find-window [-iCNrTZ] [-t target-pane] match-string
                (别名: findw)
在窗口名称、标题和可见内容（但不是历史）中搜索 fnmatch(3) 模式或，使用 -r，正则表达式 match-string。标志控制匹配行为：-C 仅匹配可见窗口内容，-N 仅匹配窗口名称，-T 仅匹配窗口标题。-i 使搜索忽略大小写。默认为 -CNT。-Z 缩放窗格。

此命令仅在至少有一个客户端附加时有效。

join-pane [-bdfhv] [-l size] [-s src-pane] [-t dst-pane]
                (别名: joinp)
类似于 split-window，但不是分割 dst-pane 并创建新窗格，而是分割它并将 src-pane 移动到空间中。这可以用于反转 break-pane。-b 选项使 src-pane 连接到 dst-pane 的左侧或上方。

如果省略 -s 且存在标记的窗格（参见 select-pane -m），则使用标记的窗格而不是当前窗格。

kill-pane [-a] [-t target-pane]
                (别名: killp)
销毁给定的窗格。如果包含窗口中没有剩余窗格，则也会销毁该窗口。-a 选项杀死除使用 -t 给定的窗格外的所有窗格。

kill-window [-a] [-t target-window]
                (别名: killw)
杀死当前窗口或 target-window 处的窗口，将其从链接到它的任何会话中移除。-a 选项杀死除使用 -t 给定的窗口外的所有窗口。

last-pane [-deZ] [-t target-window]
                (别名: lastp)
选择最后一个（之前选择的）窗格。-Z 在窗口缩放时保持缩放。-e 启用或 -d 禁用窗格输入。

last-window [-t target-session]
                (别名: last)
选择最后一个（之前选择的）窗口。如果未指定 target-session，则选择当前会话的最后一个窗口。

link-window [-abdk] [-s src-window] [-t dst-window]
                (别名: linkw)
将 src-window 处的窗口链接到指定的 dst-window。如果指定 dst-window 且不存在这样的窗口，则 src-window 链接到那里。使用 -a 或 -b，窗口移动到 dst-window 之后或之前的下一个索引（必要时移动现有窗口）。如果给定 -k 且 dst-window 存在，则将其杀死，否则生成错误。如果给定 -d，则新链接的窗口不会被选择。

list-panes [-as] [-F format] [-f filter] [-t target]
                (别名: lsp)
如果给定 -a，则忽略 target，并列出服务器上的所有窗格。如果给定 -s，则 target 是会话（或当前会话）。如果两者都未给定，则 target 是窗口（或当前窗口）。-F 指定每行的格式，-f 指定过滤器。仅显示过滤器为真的窗格。参见“格式”部分。

list-windows [-a] [-F format] [-f filter] [-t target-session]
                (别名: lsw)
如果给定 -a，则列出服务器上的所有窗口。否则，列出当前会话或 target-session 中的窗口。-F 指定每行的格式，-f 指定过滤器。仅显示过滤器为真的窗口。参见“格式”部分。

move-pane [-bdfhv] [-l size] [-s src-pane] [-t dst-pane]
                (别名: movep)
与 join-pane 相同。

move-window [-abrdk] [-s src-window] [-t dst-window]
                (别名: movew)
这类似于 link-window，但 src-window 处的窗口移动到 dst-window。使用 -r，会话中的所有窗口按顺序重新编号，尊重 base-index 选项。

new-window [-abdkPS] [-c start-directory] [-e environment] [-F format] [-n window-name] [-t target-window] [shell-command]
                (别名: neww)
创建一个新窗口。使用 -a 或 -b，新窗口插入到指定 target-window 之后或之前的下一个索引，必要时向上移动窗口；否则 target-window 是新窗口位置。

如果给定 -d，则会话不会使新窗口成为当前窗口。target-window 代表要创建的窗口；如果目标已存在，则显示错误，除非使用 -k 标志，在这种情况下将其销毁。如果给定 -S 且名为 window-name 的窗口已存在，则选择它（除非还给定 -d，在这种情况下命令不执行任何操作）。

shell-command 是要执行的命令。如果未指定 shell-command，则使用 default-command 选项的值。-c 指定创建新窗口的工作目录。

当 shell 命令完成时，窗口关闭。参见 remain-on-exit 选项以更改此行为。

-e 采用 `VARIABLE=value` 形式，并为新创建的窗口设置环境变量；可以指定多次。

TERM 环境变量必须设置为 `screen` 或 `tmux`，以便在 tmux 内运行的所有程序。新窗口将自动在其环境中添加 `TERM=screen`，但必须注意不要在 shell 启动文件中或通过 -e 选项重置此设置。

-P 选项在创建新窗口后打印有关新窗口的信息。默认情况下，它使用格式 `#{session_name}:#{window_index}`，但可以使用 -F 指定不同的格式。

next-layout [-t target-window]
                (别名: nextl)
将窗口移动到下一个布局并重新排列窗格以适应。

next-window [-a] [-t target-session]
                (别名: next)
移动到会话中的下一个窗口。如果使用 -a，则移动到带有警报的下一个窗口。

pipe-pane [-IOo] [-t target-pane] [shell-command]
                (别名: pipep)
将 target-pane 中程序发送的输出通过管道传输到 shell 命令，反之亦然。窗格一次只能连接到一个命令，任何现有管道在执行 shell-command 之前关闭。shell-command 字符串可以包含 status-left 选项支持的特殊字符序列。如果未给定 shell-command，则关闭当前管道（如果有的话）。

-I 和 -O 指定将 shell-command 输出流连接到窗格：使用 -I stdout 连接（因此 shell-command 打印的任何内容都写入窗格，就像输入一样）；使用 -O stdin 连接（因此窗格中的任何输出都通过管道传输到 shell-command）。两者可以一起使用，如果两者都未指定，则使用 -O。

-o 选项仅在没有先前管道存在时打开新管道，允许通过单个键切换管道，例如：

```
bind-key C-p pipe-pane -o 'cat >>~/output.#I-#P'
```

previous-layout [-t target-window]
                (别名: prevl)
移动到会话中的上一个布局。

previous-window [-a] [-t target-session]
                (别名: prev)
移动到会话中的上一个窗口。使用 -a，移动到带有警报的上一个窗口。

rename-window [-t target-window] new-name
                (别名: renamew)
将当前窗口或 target-window 处的窗口重命名为 new-name。

resize-pane [-DLMRTUZ] [-t target-pane] [-x width] [-y height] [adjustment]
                (别名: resizep)
使用 -U、-D、-L 或 -R 调整窗格的大小，或使用 -x 或 -y 调整为绝对大小。调整以行或列给出（默认为 1）；-x 和 -y 可以给定为行数或列数，或后跟 `%` 作为窗口大小的百分比（例如 `-x 10%`）。使用 -Z，活动窗格在缩放（占据整个窗口）和未缩放（其在布局中的正常位置）之间切换。

-M 开始鼠标调整大小（仅在绑定到鼠标键绑定时有效，请参见“鼠标支持”）。

-T 修剪当前光标位置以下的所有行，并从历史中移出行以替换它们。

resize-window [-aADLRU] [-t target-window] [-x width] [-y height] [adjustment]
                (别名: resizew)
使用 -U、-D、-L 或 -R 调整窗口的大小，或使用 -x 或 -y 调整为绝对大小。调整以行或单元格给出（默认为 1）。-A 将包含窗口的最大会话的大小设置为窗口大小；-a 将最小会话的大小设置为窗口大小。此命令将自动在窗口选项中将 window-size 设置为 manual。

respawn-pane [-k] [-c start-directory] [-e environment] [-t target-pane] [shell-command]
                (别名: respawnp)
重新激活命令已退出的窗格（参见 remain-on-exit 窗口选项）。如果未给定 shell-command，则执行创建或上次重生窗格时使用的命令。窗格必须已经不活动，除非给定 -k，在这种情况下任何现有命令都会被杀死。-c 为窗格指定新的工作目录。-e 选项与 new-window 命令具有相同的含义。

respawn-window [-k] [-c start-directory] [-e environment] [-t target-window] [shell-command]
                (别名: respawnw)
重新激活命令已退出的窗口（参见 remain-on-exit 窗口选项）。如果未给定 shell-command，则执行创建或上次重生窗口时使用的命令。窗口必须已经不活动，除非给定 -k，在这种情况下任何现有命令都会被杀死。-c 为窗口指定新的工作目录。-e 选项与 new-window 命令具有相同的含义。

rotate-window [-DUZ] [-t target-window]
                (别名: rotatew)
在窗口内旋转窗格的位置，使用 -U 向上（数值较低）或使用 -D 向下（数值较高）。-Z 在窗口缩放时保持缩放。

select-layout [-Enop] [-t target-pane] [layout-name]
                (别名: selectl)
为窗口选择特定布局。如果未给定 layout-name，则重新应用上次使用的预设布局（如果有的话）。-n 和 -p 分别等同于 next-layout 和 previous-layout 命令。-o 应用上次设置的布局（如果可能）（撤消最近的布局更改）。-E 将当前窗格及其相邻窗格均匀展开。

select-pane [-DdeLlMmRUZ] [-T title] [-t target-pane]
                (别名: selectp)
使 target-pane 成为其窗口中的活动窗格。如果使用 -D、-L、-R 或 -U 之一，则分别使用 target-pane 下方、左侧、右侧或上方的窗格。-Z 在窗口缩放时保持缩放。-l 与使用 last-pane 命令相同。-e 启用或 -d 禁用窗格输入。-T 设置窗格标题。

-m 和 -M 用于设置和清除标记的窗格。一次只有一个标记的窗格，设置新的标记窗格会清除最后一个。标记的窗格是 join-pane、move-pane、swap-pane 和 swap-window 的 -s 目标的默认目标。

select-window [-lnpT] [-t target-window]
                (别名: selectw)
选择 target-window 处的窗口。-l、-n 和 -p 分别等同于 last-window、next-window 和 previous-window 命令。如果给定 -T 且选定的窗口已经是当前窗口，则命令行为类似于 last-window。

split-window [-bdfhIvPZ] [-c start-directory] [-e environment] [-l size] [-t target-pane] [shell-command] [-F format]
                (别名: splitw)
通过分割 target-pane 创建新窗格：-h 进行水平分割，-v 进行垂直分割；如果两者都未指定，则假定 -v。-l 选项指定新窗格的大小，以行（垂直分割）或列（水平分割）为单位；size 可以后跟 `%` 以指定可用空间的百分比。-b 选项使新窗格在 target-pane 的左侧或上方创建。-f 选项创建一个跨越整个窗口高度（使用 -h）或整个窗口宽度（使用 -v）的新窗格，而不是分割活动窗格。-Z 如果窗口未缩放则缩放，如果已缩放则保持缩放。

空的 shell-command ('') 将创建一个没有运行命令的窗格。可以使用 display-message 命令将输出发送到这样的窗格。如果未指定或为空 shell-command，-I 标志将创建一个空窗格并将任何来自 stdin 的输出转发给它。例如：

```
$ make 2>&1|tmux splitw -dI &
```

所有其他选项与 new-window 命令具有相同的含义。

swap-pane [-dDUZ] [-s src-pane] [-t dst-pane]
                (别名: swapp)
交换两个窗格。如果使用 -U 且未使用 -s 指定源窗格，则 dst-pane 与上一个窗格（在数值上之前）交换；-D 与下一个窗格（在数值上之后）交换。-d 指示 tmux 不要更改活动窗格，-Z 在窗口缩放时保持缩放。

如果省略 -s 且存在标记的窗格（参见 select-pane -m），则使用标记的窗格而不是当前窗格。

swap-window [-d] [-s src-window] [-t dst-window]
                (别名: swapw)
这类似于 link-window，但源窗口和目标窗口被交换。如果 src-window 处不存在窗口，则是错误。如果给定 -d，则新窗口不会成为当前窗口。

如果省略 -s 且存在标记的窗格（参见 select-pane -m），则使用包含标记窗格的窗口而不是当前窗口。

unlink-window [-k] [-t target-window]
                (别名: unlinkw)
取消链接 target-window。除非给定 -k，否则窗口只能在链接到多个会话时取消链接——窗口不能链接到没有会话；如果指定 -k 且窗口仅链接到一个会话，则取消链接并销毁它。

## 键绑定
tmux 允许将命令绑定到大多数键，有或没有前缀键。指定键时，大多数键代表它们自己（例如 `A` 到 `Z`）。Ctrl 键可以用 `C-` 或 `^` 作为前缀，Shift 键用 `S-`，Alt（meta）用 `M-`。此外，接受以下特殊键名：Up、Down、Left、Right、BSpace、BTab、DC（Delete）、End、Enter、Escape、F1 到 F12、Home、IC（Insert）、NPage/PageDown/PgDn、PPage/PageUp/PgUp、Space 和 Tab。请注意，要绑定 `"` 或 `''` 键，需要引号，例如：

```
bind-key '"' split-window
bind-key "'" new-window
```

绑定到 Any 键的命令将对所有没有更具体绑定的键执行。

与键绑定相关的命令如下：

bind-key [-nr] [-N note] [-T key-table] key command [argument ...]
                (别名: bind)
将键 key 绑定到命令。键绑定在键表中。默认情况下（不使用 -T），键绑定在前缀键表中。此表用于按下前缀键后按下的键（例如，默认情况下 `c` 绑定到 new-window 在前缀表中，因此 `C-b c` 创建一个新窗口）。根表用于不带前缀键按下的键：在根表中将 `c` 绑定到 new-window（不推荐）意味着普通的 `c` 将创建一个新窗口。-n 是 -T 根的别名。键也可以绑定在自定义键表中，并使用 switch-client -T 命令从键绑定切换到它们。-r 标志表示此键可以重复，请参见 repeat-time 选项。-N 将注释附加到键（使用 list-keys -N 显示）。

要查看默认绑定和可能的命令，请参见 list-keys 命令。

list-keys [-1aN] [-P prefix-string -T key-table] [key]
                (别名: lsk)
列出键绑定。有两种形式：默认列出键作为 bind-key 命令；-N 仅列出带有附加注释的键，并且仅显示每个键的键和注释。

使用默认形式，默认情况下列出所有键表。-T 仅列出 key-table 中的键。

使用 -N 形式，默认情况下仅列出根和前缀键表中的键；-T 还仅列出 key-table 中的键。-P 指定在每个键之前打印的前缀，-1 仅列出第一个匹配的键。-a 列出没有注释的键的命令，而不是跳过它们。

send-keys [-FHKlMRX] [-c target-client] [-N repeat-count] [-t target-pane] key ...
                (别名: send)
向窗口或客户端发送一个或多个键。每个参数 key 是要发送的键的名称（例如 `C-a` 或 `NPage`）；如果字符串未被识别为键，则作为一系列字符发送。如果给定 -K，则键发送到 target-client，因此在客户端的键表中查找，而不是发送到 target-pane。所有参数从第一个到最后一个按顺序发送。如果未给定键且命令绑定到键，则使用该键。

-l 标志禁用键名查找并将键作为文字 UTF-8 字符处理。-H 标志期望每个键是 ASCII 字符的十六进制数。

-R 标志导致终端状态重置。

-M 传递鼠标事件（仅在绑定到鼠标键绑定时有效，请参见“鼠标支持”）。

-X 用于将命令发送到复制模式——参见“窗口和窗格”部分。-N 指定重复计数，-F 在适当的地方扩展格式中的参数。

send-prefix [-2] [-t target-pane]
发送前缀键，或使用 -2 发送次前缀键，到窗口，就像按下了一样。

unbind-key [-anq] [-T key-table] key
                (别名: unbind)
取消绑定到键的命令。-n 和 -T 与 bind-key 相同。如果存在 -a，则删除所有键绑定。-q 选项防止返回错误。

## 选项
tmux 的外观和行为可以通过更改各种选项的值来修改。有四种类型的选项：服务器选项、会话选项、窗口选项和窗格选项。

tmux 服务器有一组不适用于任何特定窗口或会话或窗格的全局服务器选项。这些通过 set-option -s 命令更改，或通过 show-options -s 命令显示。

此外，每个单独的会话可能有一组会话选项，还有一组全局会话选项。没有配置特定选项的会话从全局会话选项继承值。会话选项使用 set-option 命令设置或取消设置，可以使用 show-options 命令列出。可用的服务器和会话选项在 set-option 命令下列出。

同样，一组窗口选项附加到每个窗口，一组窗格选项附加到每个窗格。窗格选项从窗口选项继承。这意味着任何窗格选项可以作为窗口选项设置，以将选项应用于未设置选项的所有窗格，例如这些命令将除窗格 0 外的所有窗格的背景颜色设置为红色：

```
set -w window-style bg=red
set -pt:.0 window-style bg=blue
```

还有一组全局窗口选项，从中继承任何未设置的窗口或窗格选项。窗口和窗格选项通过 set-option -w 和 -p 命令更改，并通过 show-option -w 和 -p 显示。

tmux 还支持以 `@` 为前缀的用户选项。用户选项可以有任何名称，只要它们以 `@` 为前缀，并且可以设置为任何字符串。例如：

```
$ tmux set -wq @foo "abc123"
$ tmux show -wv @foo
abc123
```

设置选项的命令如下：

set-option [-aFgopqsuUw] [-t target-pane] option value
                (别名: set)
使用 -p 设置窗格选项，使用 -w 设置窗口选项，使用 -s 设置服务器选项，否则设置会话选项。如果选项不是用户选项，则 -w 或 -s 可能不必要——tmux 将从选项名称推断类型，假设 -w 用于窗格选项。如果给定 -g，则设置全局会话或窗口选项。

-F 在选项值中扩展格式。-u 标志取消设置选项，因此会话从全局选项继承选项（或使用 -g，恢复全局选项为默认值）。-U 取消设置选项（类似于 -u），但如果选项是窗格选项，还会取消设置窗口中任何窗格的选项。value 取决于选项，可以是数字、字符串或标志（on、off 或省略以切换）。

-o 标志防止设置已设置的选项，-q 标志抑制有关未知或模糊选项的错误。

使用 -a，如果选项期望字符串或样式，则 value 附加到现有设置。例如：

```
set -g status-left "foo"
set -ag status-left "bar"
```

将导致 `foobar`。还有：

```
set -g status-style "bg=red"
set -ag status-style "fg=blue"
```

将导致红色背景和蓝色前景。没有 -a，结果将是默认背景和蓝色前景。

show-options [-AgHpqsvw] [-t target-pane] [option]
                (别名: show)
使用 -p 显示窗格选项（如果提供 option，则显示单个选项），使用 -w 显示窗口选项，使用 -s 显示服务器选项，否则显示会话选项。如果选项不是用户选项，则 -w 或 -s 可能不必要——tmux 将从选项名称推断类型，假设 -w 用于窗格选项。使用 -g 列出全局会话或窗口选项。-v 仅显示选项值，不显示名称。如果设置 -q，则如果选项未设置，不会返回错误。-H 包括钩子（默认情况下省略）。-A 包括从父选项集继承的选项，此类选项用星号标记。

可用的服务器选项如下：

backspace key
        设置 tmux 用于退格的键。

buffer-limit number
        设置缓冲区数量；当新缓冲区添加到堆栈顶部时，如果需要，旧的缓冲区会从底部移除以保持此最大长度。

command-alias[] name=value
        这是一个自定义命令别名数组。如果未知命令匹配 name，则替换为 value。例如，之后：

```
set -s command-alias[100] zoom='resize-pane -Z'
```

使用：

```
zoom -t:.1
```

等同于：

```
resize-pane -Z -t:.1
```

请注意，别名在解析命令时扩展，而不是在执行时扩展，因此使用 bind-key 绑定别名将绑定扩展形式。

copy-command shell-command
        如果 copy-pipe 复制模式命令在没有参数的情况下使用，则给出要通过管道传输到的命令。

default-terminal terminal
        设置在此会话中创建的新窗口的默认终端——TERM 环境变量的默认值。为了使 tmux 正常工作，这必须设置为 `screen`、`tmux` 或它们的衍生品。

escape-time time
        设置 tmux 在输入转义后等待的时间（以毫秒为单位），以确定它是否是功能键或元键序列的一部分。

editor shell-command
        设置 tmux 运行编辑器时使用的命令。

exit-empty [on | off]
        如果启用（默认），当没有活动会话时，服务器将退出。

exit-unattached [on | off]
        如果启用，当没有附加客户端时，服务器将退出。

extended-keys [on | off | always]
        控制如何报告修改键（与 Control、Meta 或 Shift 一起按下的键）。这相当于 xterm(1) 的 modifyOtherKeys 资源。

        当设置为 on 时，窗格内的程序可以请求两种模式之一：模式 1，仅更改没有现有知名表示的键的序列；或模式 2，更改所有键的序列。当设置为 always 时，应用程序仍可以请求模式 1 和 2，但模式 1 将被强制而不是标准模式。当设置为 off 时，此功能被禁用，仅报告标准键。

        如果终端支持扩展键，tmux 将始终请求扩展键本身。另请参见 terminal-features 选项的 extkeys 功能、extended-keys-format 选项和 pane_key_mode 变量。

extended-keys-format [csi-u | xterm]
        为向应用程序报告修改键选择两种可能格式之一。这相当于 xterm(1) 的 formatOtherKeys 资源。例如，C-S-a 将被报告为 `^[[27;6;65~`（当设置为 xterm 时）和 `^[[65;6u`（当设置为 csi-u 时）。

focus-events [on | off]
        启用时，如果支持，将从终端请求焦点事件并传递给在 tmux 中运行的应用程序。附加客户端应在更改此选项后分离并重新附加。

history-file path
        如果不为空，则是一个文件，tmux 将在退出时将命令提示历史写入该文件，并在启动时从中加载。

message-limit number
        设置为每个客户端保存在消息日志中的错误或信息消息的数量。

prompt-history-limit number
        设置为每种类型的命令提示保存在历史文件中的历史项目数量。

set-clipboard [on | external | off]
        尝试使用 xterm(1) 转义序列设置终端剪贴板内容，如果 terminfo(5) 描述中有 Ms 条目（参见“TERMINFO 扩展”部分）。

        如果设置为 on，tmux 将接受创建缓冲区的转义序列并尝试设置终端剪贴板。如果设置为 external，tmux 将尝试设置终端剪贴板，但忽略应用程序设置 tmux 缓冲区的尝试。如果 off，tmux 将既不接受剪贴板转义序列，也不尝试设置剪贴板。

        请注意，此功能需要在 xterm(1) 中启用，方法是设置资源：

```
disallowedWindowOps: 20,21,SetXprop
```

        或在需要时从 xterm(1) 交互菜单更改此属性。

terminal-features[] string
        设置从 terminfo(5) 读取的终端类型的终端功能。tmux 有一组命名的终端功能。每个功能将对正在使用的 terminfo(5) 条目应用适当的更改。

        tmux 可以检测一些常见终端的功能；此选项可用于轻松告诉 tmux 支持终端无法检测到的功能。terminal-overrides 选项允许单独设置 terminfo(5) 功能，terminal-features 旨在用于以标准方式支持但 terminfo(5) 未报告的功能类。必须小心仅配置终端实际支持的功能。

        这是一个数组选项，每个条目是由冒号分隔的字符串，由终端类型模式（使用 fnmatch(3) 匹配）和终端功能列表组成。可用的功能包括：

        256     支持使用 SGR 转义序列的 256 色。

        clipboard
                允许设置系统剪贴板。

        ccolour
                允许设置光标颜色。

        cstyle  允许设置光标样式。

        extkeys
                支持扩展键。

        focus   支持焦点报告。

        hyperlinks
                支持 OSC 8 超链接。

        ignorefkeys
                忽略 terminfo(5) 中的功能键，仅使用 tmux 内部集。

        margins
                支持 DECSLRM 边距。

        mouse   支持 xterm(1) 鼠标序列。

        osc7    支持 OSC 7 工作目录扩展。

        overline
                支持 overline SGR 属性。

        rectfill
                支持 DECFRA 矩形填充转义序列。

        RGB     支持使用 SGR 转义序列的 RGB 颜色。

        sixel   支持 SIXEL 图形。

        strikethrough
                支持删除线 SGR 转义序列。

        sync    支持同步更新。

        title   支持 xterm(1) 标题设置。

        usstyle
                允许设置下划线样式和颜色。

terminal-overrides[] string
        允许覆盖使用 terminfo(5) 读取的终端描述。每个条目是由冒号分隔的字符串，由终端类型模式（使用 fnmatch(3) 匹配）和 name=value 条目集组成。

        例如，为所有匹配 `rxvt*` 的终端类型将 `clear` terminfo(5) 条目设置为 `\e[H\e[2J`：

```
rxvt*:clear=\e[H\e[2J
```

        终端条目值在解释之前通过 strunvis(3) 传递。

user-keys[] key
        设置用户定义的键转义序列列表。每个项目与名为 `User0`、`User1` 等的键相关联。

        例如：

```
set -s user-keys[0] "\e[5;30012~"
bind User0 resize-pane -L 3
```

可用的会话选项如下：

activity-action [any | none | current | other]
        设置 monitor-activity 开启时窗口活动的操作。any 表示链接到会话的任何窗口中的活动都会导致该会话当前窗口中的铃声或消息（取决于 visual-activity）；none 表示忽略所有活动（相当于关闭 monitor-activity）；current 表示仅忽略当前窗口以外的窗口中的活动；other 表示忽略当前窗口中的活动，但不忽略其他窗口中的活动。

assume-paste-time milliseconds
        如果键输入速度超过每毫秒一个，则假定它们是粘贴而不是键入，并且不处理 tmux 键绑定。默认为一毫秒，零表示禁用。

base-index index
        设置创建新窗口时应从中搜索未使用索引的基本索引。默认为零。

bell-action [any | none | current | other]
        设置 monitor-bell 开启时窗口铃声的操作。值与 activity-action 相同。

default-command shell-command
        将用于新窗口（如果在创建窗口时未指定）的命令设置为 shell-command，它可以是任何 sh(1) 命令。默认为空字符串，指示 tmux 使用 default-shell 选项的值创建登录 shell。

default-shell path
        指定默认 shell。这用作新窗口的登录 shell，当 default-command 选项设置为空时，并且必须是可执行文件的完整路径。启动时，tmux 尝试从 SHELL 环境变量、getpwuid(3) 返回的 shell 或 /bin/sh 中第一个合适的值设置默认值。当 tmux 用作登录 shell 时，应配置此选项。

default-size XxY
        当 window-size 选项设置为 manual 或使用 new-session -d 创建会话时，设置新窗口的默认大小。值是用 `x` 字符分隔的宽度和高度。默认为 80x24。

destroy-unattached [off | on | keep-last | keep-group]
        如果 on，在最后一个客户端分离后销毁会话。如果 off（默认），则使会话成为孤儿。如果 keep-last，仅当会话在组中且该组中有其他会话时才销毁会话。如果 keep-group，仅当会话在组中且是该组中唯一的会话时才销毁会话。

detach-on-destroy [off | on | no-detached | previous | next]
        如果 on（默认），当附加到的会话被销毁时，客户端分离。如果 off，客户端切换到剩余会话中最最近活动的会话。如果 no-detached，仅当没有分离的会话时才分离客户端；如果有分离的会话，客户端切换到最最近活动的会话。如果 previous 或 next，客户端按字母顺序切换到上一个或下一个会话。

display-panes-active-colour colour
        设置 display-panes 命令用于显示活动窗格指示器的颜色。

display-panes-colour colour
        设置 display-panes 命令用于显示非活动窗格指示器的颜色。

display-panes-time time
        设置 display-panes 命令显示的指示器出现的时间（以毫秒为单位）。

display-time time
        设置状态行消息和其他屏幕指示器显示的时间。如果设置为 0，则消息和指示器将显示直到按下键。time 以毫秒为单位。

history-limit lines
        设置窗口历史中保留的最大行数。此设置仅适用于新窗口——现有窗口历史不会调整大小，并保留创建时的限制。

key-table key-table
        将默认键表设置为 key-table 而不是根。

lock-after-time number
        在 number 秒的非活动时间后锁定会话（如 lock-session 命令）。默认不锁定（设置为 0）。

lock-command shell-command
        锁定每个客户端时运行的命令。默认是使用 -np 运行 lock(1)。

menu-style style
        设置菜单样式。有关如何指定样式的详细信息，请参见“样式”部分。属性被忽略。

menu-selected-style style
        设置选定菜单项的样式。有关如何指定样式的详细信息，请参见“样式”部分。属性被忽略。

menu-border-style style
        设置菜单边框样式。有关如何指定样式的详细信息，请参见“样式”部分。属性被忽略。

menu-border-lines type
        设置用于绘制菜单边框的字符类型。有关 border-lines 的可能值，请参见 popup-border-lines。

message-command-style style
        设置状态行消息命令样式。这用于 vi(1) 键的命令提示符处于命令模式时。有关如何指定样式的详细信息，请参见“样式”部分。

message-line [0 | 1 | 2 | 3 | 4]
        设置状态行消息和命令提示符显示的行。

message-style style
        设置状态行消息样式。这用于消息和命令提示符。有关如何指定样式的详细信息，请参见“样式”部分。

mouse [on | off]
        如果 on，tmux 捕获鼠标并允许将鼠标事件绑定为键绑定。有关详细信息，请参见“鼠标支持”部分。

prefix key
        设置接受为前缀键的键。除了“键绑定”下描述的标准键外，前缀可以设置为特殊键 `None` 以不设置前缀。

prefix2 key
        设置接受为次前缀键的键。与前缀类似，prefix2 可以设置为 `None`。

prefix-timeout time
        设置 tmux 在输入前缀后等待的时间（以毫秒为单位），然后取消它。可以设置为零以禁用任何超时。

renumber-windows [on | off]
        如果 on，当会话中的窗口关闭时，自动按数字顺序重新编号其他窗口。这尊重 base-index 选项（如果已设置）。如果 off，则不重新编号窗口。

repeat-time time
        允许在指定的时间毫秒内不按前缀键再次输入多个命令（默认为 500）。当使用 bind-key 的 -r 标志绑定键时，可以设置键是否重复。默认键绑定到 resize-pane 命令的键重复启用。

set-titles [on | off]
        尝试使用 tsl 和 fsl terminfo(5) 条目设置客户端终端标题（如果存在）。tmux 会自动将这些设置为 \e]0;...\007 序列，如果终端似乎是 xterm(1)。默认情况下此选项关闭。

set-titles-string string
        如果 set-titles 开启，则用于设置客户端终端标题的字符串。格式被扩展，请参见“格式”部分。

silence-action [any | none | current | other]
        设置 monitor-silence 开启时窗口静音的操作。值与 activity-action 相同。

status [off | on | 2 | 3 | 4 | 5]
        显示或隐藏状态行或指定其大小。使用 on 给出一行高的状态行；2、3、4 或 5 更多行。

status-format[] format
        指定用于状态行每行的格式。默认构建顶部状态行来自下面的各种单独状态选项。

status-interval interval
        每 interval 秒更新一次状态行。默认情况下，每 15 秒更新一次。设置为零会禁用间隔重绘。

status-justify [left | centre | right | absolute-centre]
        设置状态行中窗口列表的位置：left、centre 或 right。centre 将窗口列表放在可用空闲空间的相对中心；absolute-centre 使用整个水平空间的中心。

status-keys [vi | emacs]
        在状态行中使用 vi 或 emacs 风格的键绑定，例如在命令提示符处。默认为 emacs，除非 VISUAL 或 EDITOR 环境变量设置并包含字符串 `vi`。

status-left string
        在状态行左侧显示字符串（默认为会话名称）。string 将通过 strftime(3) 传递。另请参见“格式”和“样式”部分。

        有关如何设置名称和标题的详细信息，请参见“名称和标题”部分。

        示例包括：

```
#(sysctl vm.loadavg)
#[fg=yellow,bold]#(apm -l)%%#[default] [#S]
```

        默认为 `[#S] `。

status-left-length length
        设置状态行左侧组件的最大长度。默认为 10。

status-left-style style
        设置状态行左侧部分的样式。有关如何指定样式的详细信息，请参见“样式”部分。

status-position [top | bottom]
        设置状态行的位置。

status-right string
        在状态行右侧显示字符串。默认情况下，显示当前窗格标题（用双引号括起来）、日期和时间。与 status-left 一样，string 将传递给 strftime(3) 和字符对被替换。

status-right-length length
        设置状态行右侧组件的最大长度。默认为 40。

status-right-style style
        设置状态行右侧部分的样式。有关如何指定样式的详细信息，请参见“样式”部分。

status-style style
        设置状态行样式。有关如何指定样式的详细信息，请参见“样式”部分。

update-environment[] variable
        设置在创建新会话或附加到现有会话时复制到会话环境中的环境变量列表。源环境中不存在的任何变量都设置为从会话环境中移除（就像给 set-environment 命令一样 -r）。

visual-activity [on | off | both]
        如果 on，当为 monitor-activity 窗口选项启用的窗口中发生活动时，显示消息而不是发送铃声。如果设置为 both，则产生铃声和消息。

visual-bell [on | off | both]
        如果 on，当为 monitor-bell 窗口选项启用的窗口中发生铃声时，显示消息而不是传递给终端（通常会发出声音）。如果设置为 both，则产生铃声和消息。另请参见 bell-action 选项。

visual-silence [on | off | both]
        如果启用 monitor-silence，则在给定窗口的间隔到期后打印消息，而不是发送铃声。如果设置为 both，则产生铃声和消息。

word-separators string
        设置会话对哪些字符被认为是单词分隔符的概念，用于复制模式中的下一个和上一个单词命令。

可用的窗口选项如下：

aggressive-resize [on | off]
        积极调整选定窗口的大小。这意味着 tmux 将窗口调整为附加到它的最小或最大会话（参见 window-size 选项）的大小，而不是附加到它的会话。当在另一个会话上更改当前窗口时，窗口可能会调整大小；此选项适用于支持 SIGWINCH 且交互式程序（如 shell）较差的全屏程序。

automatic-rename [on | off]
        控制自动窗口重命名。启用此设置时，tmux 将使用 automatic-rename-format 指定的格式自动重命名窗口。此标志在创建时使用 new-window 或 new-session 指定名称，或稍后使用 rename-window，或使用终端转义序列时自动为单个窗口禁用。可以全局关闭：

```
set-option -wg automatic-rename off
```

automatic-rename-format format
        启用 automatic-rename 选项时使用的格式（参见“格式”）。

clock-mode-colour colour
        设置时钟颜色。

clock-mode-style [12 | 24]
        设置时钟小时格式。

fill-character character
        设置用于填充窗口未使用区域的字符。

main-pane-height height
main-pane-width width
        在 main-horizontal、main-horizontal-mirrored、main-vertical 或 main-vertical-mirrored 布局中设置主（左或上）窗格的高度或宽度。如果后缀为 `%'，则为窗口大小的百分比。

copy-mode-match-style style
        设置复制模式中搜索匹配的样式。有关如何指定样式的详细信息，请参见“样式”部分。

copy-mode-mark-style style
        设置复制模式中标记行的样式。有关如何指定样式的详细信息，请参见“样式”部分。

copy-mode-current-match-style style
        设置复制模式中当前搜索匹配的样式。有关如何指定样式的详细信息，请参见“样式”部分。

mode-keys [vi | emacs]
        在复制模式中使用 vi 或 emacs 风格的键绑定。默认为 emacs，除非 VISUAL 或 EDITOR 包含 `vi`。

mode-style style
        设置窗口模式样式。有关如何指定样式的详细信息，请参见“样式”部分。

monitor-activity [on | off]
        监视窗口中的活动。有活动的窗口在状态行中突出显示。

monitor-bell [on | off]
        监视窗口中的铃声。有铃声的窗口在状态行中突出显示。

monitor-silence [interval]
        监视窗口中的静音（无活动）在 interval 秒内。在间隔内静音的窗口在状态行中突出显示。间隔为零会禁用监视。

other-pane-height height
        在 main-horizontal 和 main-horizontal-mirrored 布局中设置其他窗格（不是主窗格）的高度。如果此选项设置为 0（默认），则无效。如果同时设置 main-pane-height 和 other-pane-height 选项，则主窗格会增长更高以使其他窗格达到指定高度，但不会缩小以这样做。如果后缀为 `%'，则为窗口大小的百分比。

other-pane-width width
        与 other-pane-height 类似，但在 main-vertical 和 main-vertical-mirrored 布局中设置其他窗格的宽度。

pane-active-border-style style
        为当前活动窗格设置窗格边框样式。有关如何指定样式的详细信息，请参见“样式”部分。属性被忽略。

pane-base-index index
        与 base-index 类似，但设置窗格编号的起始索引。

pane-border-format format
        设置窗格边框状态行中显示的文本。

pane-border-indicators [off | colour | arrows | both]
        通过仅在恰好有两个窗格的窗口中为边框的一半着色、显示箭头标记、绘制两者或都不来指示活动窗格。

pane-border-lines type
        设置用于绘制窗格边框的字符类型。type 可以是以下之一：

        single  使用 ACS 或 UTF-8 字符的单线

        double  使用 UTF-8 字符的双线

        heavy   使用 UTF-8 字符的粗线

        simple  简单的 ASCII 字符

        number  窗格编号

        `double` 和 `heavy` 将在不支持 UTF-8 时回退到标准 ACS 线绘制。

pane-border-status [off | top | bottom]
        关闭窗格边框状态行或将它们的位置设置为顶部或底部。

pane-border-style style
        为除活动窗格外的窗格设置窗格边框样式。有关如何指定样式的详细信息，请参见“样式”部分。属性被忽略。

popup-style style
        设置弹出样式。有关如何指定样式的详细信息，请参见“样式”部分。属性被忽略。

popup-border-style style
        设置弹出边框样式。有关如何指定样式的详细信息，请参见“样式”部分。属性被忽略。

popup-border-lines type
        设置用于绘制弹出边框的字符类型。type 可以是以下之一：

        single  使用 ACS 或 UTF-8 字符的单线（默认）

        rounded
                使用 UTF-8 字符的单线变体，带有圆角

        double  使用 UTF-8 字符的双线

        heavy   使用 UTF-8 字符的粗线

        simple  简单的 ASCII 字符

        padded  简单的 ASCII 空格字符

        none    无边框

        `double` 和 `heavy` 将在不支持 UTF-8 时回退到标准 ACS 线绘制。

window-status-activity-style style
        为有活动警报的窗口设置状态行样式。有关如何指定样式的详细信息，请参见“样式”部分。

window-status-bell-style style
        为有铃声警报的窗口设置状态行样式。有关如何指定样式的详细信息，请参见“样式”部分。

window-status-current-format string
        与 window-status-format 类似，但用于窗口是当前窗口时使用的格式。

window-status-current-style style
        为当前活动窗口设置状态行样式。有关如何指定样式的详细信息，请参见“样式”部分。

window-status-format string
        设置状态行窗口列表中显示窗口的格式。参见“格式”和“样式”部分。

window-status-last-style style
        为上一个活动窗口设置状态行样式。有关如何指定样式的详细信息，请参见“样式”部分。

window-status-separator string
        设置状态行中窗口之间绘制的分隔符。默认为单个空格字符。

window-status-style style
        为单个窗口设置状态行样式。有关如何指定样式的详细信息，请参见“样式”部分。

window-size largest | smallest | manual | latest
        配置 tmux 如何确定窗口大小。如果设置为 largest，使用最大附加会话的大小；如果设置为 smallest，使用最小的。如果设置为 manual，新窗口的大小由 default-size 选项设置，并且窗口自动调整大小。使用 latest，tmux 使用具有最近活动的客户端的大小。另请参见 resize-window 命令和 aggressive-resize 选项。

wrap-search [on | off]
        如果启用此选项，搜索将环绕窗格内容的末尾。默认为 on。

可用的窗格选项如下：

allow-passthrough [on | off | all]
        允许窗格中的程序使用终端转义序列 (\ePtmux;...\e\\) 绕过 tmux。如果设置为 on，仅当窗格可见时才允许传递序列。如果设置为 all，即使窗格不可见也允许。

allow-rename [on | off]
        允许窗格中的程序使用终端转义序列 (\ek...\e\\) 更改窗口名称。

allow-set-title [on | off]
        允许窗格中的程序使用终端转义序列 (\e]2;...\e\\ 或 \e]0;...\e\\) 更改标题。

alternate-screen [on | off]
        此选项配置窗格中运行的程序是否可以使用终端备用屏幕功能，这允许 smcup 和 rmcup terminfo(5) 功能。备用屏幕功能在交互式应用程序启动时保留窗口内容，并在退出时恢复，以便应用程序启动前可见的任何输出在退出后不变地重新出现。

cursor-colour colour
        设置光标颜色。

pane-colours[] colour
        默认调色板。数组中的每个条目定义当请求该索引的颜色时 tmux 使用的颜色。索引可以从 0 到 255。

cursor-style style
        设置光标样式。可用样式包括：default、blinking-block、block、blinking-underline、underline、blinking-bar、bar。

remain-on-exit [on | off | failed]
        设置此标志的窗格在其中运行的程序退出时不会被销毁。如果设置为 failed，则仅当程序退出状态不为零时。可以使用 respawn-pane 命令重新激活窗格。

remain-on-exit-format string
        设置启用 remain-on-exit 时在退出窗格底部显示的文本。

scroll-on-clear [on | off]
        当整个屏幕被清除且此选项开启时，在清除前将屏幕内容滚动到历史中。

synchronize-panes [on | off]
        将输入复制到同一窗口中也开启此选项的所有其他窗格（仅适用于未处于任何模式的窗格）。

window-active-style style
        设置活动窗格时的窗格样式。有关如何指定样式的详细信息，请参见“样式”部分。

window-style style
        设置窗格样式。有关如何指定样式的详细信息，请参见“样式”部分。

## 钩子
tmux 允许命令在各种触发器上运行，称为钩子。大多数 tmux 命令都有一个 after 钩子，并且有一些钩子不与命令关联。

钩子作为数组选项存储，数组成员按顺序在触发钩子时执行。与选项不同，不同的钩子可以是全局的，也可以属于会话、窗口或窗格。可以使用 set-hook 或 set-option 命令配置钩子，并使用 show-hooks 或 show-options -H 显示。以下两个命令是等效的：

```
set-hook -g pane-mode-changed[42] 'set -g status-left-style bg=red'
set-option -g pane-mode-changed[42] 'set -g status-left-style bg=red'
```

设置钩子而不指定数组索引会清除钩子并设置数组的第一个成员。

命令的 after 钩子在命令完成后运行，除非命令本身作为钩子的一部分运行。它们以 `after-` 前缀命名。例如，以下命令添加一个钩子，在每次 split-window 后选择 even-vertical 布局：

```
set-hook -g after-split-window "selectl even-vertical"
```

如果命令失败，则会触发 `command-error` 钩子。例如，这可以用于写入日志文件：

```
set-hook -g command-error "run-shell \"echo 'a tmux command failed' >>/tmp/log\""
```

“控制模式”部分列出的所有通知都是钩子（没有任何参数），除了 %exit。以下附加钩子可用：

alert-activity          在窗口有活动时运行。参见 monitor-activity。

alert-bell              在窗口收到铃声时运行。参见 monitor-bell。

alert-silence           在窗口静音时运行。参见 monitor-silence。

client-active           在客户端成为其会话的最新活动客户端时运行。

client-attached         在客户端附加时运行。

client-detached         在客户端分离时运行。

client-focus-in         在焦点进入客户端时运行。

client-focus-out        在焦点退出客户端时运行。

client-resized          在客户端调整大小时运行。

client-session-changed  在客户端的附加会话更改时运行。

command-error           在命令失败时运行。

pane-died               在窗格中运行的程序退出时运行，但 remain-on-exit 开启，因此窗格尚未关闭。

pane-exited             在窗格中运行的程序退出时运行。

pane-focus-in           在焦点进入窗格时运行，如果 focus-events 选项开启。

pane-focus-out          在焦点退出窗格时运行，如果 focus-events 选项开启。

pane-set-clipboard      在使用 xterm(1) 转义序列设置终端剪贴板时运行。

session-created         在创建新会话时运行。

session-closed          在会话关闭时运行。

session-renamed         在会话重命名时运行。

window-linked           在窗口链接到会话时运行。

window-renamed          在窗口重命名时运行。

window-resized          在窗口调整大小时运行。这可能在 client-resized 钩子运行后发生。

window-unlinked         在窗口从会话取消链接时运行。

钩子通过以下命令管理：

set-hook [-agpRuw] [-t target-pane] hook-name command
        不使用 -R，设置（或使用 -u 取消设置）钩子 hook-name 到 command。标志与 set-option 相同。

        使用 -R，立即运行 hook-name。

show-hooks [-gpw] [-t target-pane]
        显示钩子。标志与 show-options 相同。

## 鼠标支持
如果 mouse 选项开启（默认关闭），tmux 允许将鼠标事件绑定为键。每个键的名称由鼠标事件（例如 `MouseUp1`）和位置后缀组成，位置后缀如下之一：

        Pane             窗格的内容
        Border           窗格边框
        Status           状态行窗口列表
        StatusLeft       状态行的左侧部分
        StatusRight      状态行的右侧部分
        StatusDefault    状态行的任何其他部分

以下鼠标事件可用：

        WheelUp       WheelDown
        MouseDown1    MouseUp1      MouseDrag1   MouseDragEnd1
        MouseDown2    MouseUp2      MouseDrag2   MouseDragEnd2
        MouseDown3    MouseUp3      MouseDrag3   MouseDragEnd3
        SecondClick1  SecondClick2  SecondClick3
        DoubleClick1  DoubleClick2  DoubleClick3
        TripleClick1  TripleClick2  TripleClick3

        `SecondClick` 事件在双击的第二次点击时触发，即使可能存在第三次点击，这将触发 `TripleClick` 而不是 `DoubleClick`。

每个事件应后缀一个位置，例如 `MouseDown1Status`。

特殊标记 `{mouse}` 或 `=` 可以在绑定到鼠标键绑定的命令中用作 target-window 或 target-pane。它解析为鼠标事件发生的窗口或窗格（例如，对于 `MouseUp1Status` 绑定，按钮 1 释放时状态行中的窗口，或对于 `WheelDownPane` 绑定，滚动滚轮的窗格）。

send-keys -M 标志可用于将鼠标事件转发到窗格。

默认键绑定允许使用鼠标选择和调整窗格大小、复制文本以及使用状态行更改窗口。如果 mouse 选项开启，这些将生效。

## 格式
某些命令接受带有格式参数的 -F 标志。这是一个控制命令输出格式的字符串。格式变量用 `#{` 和 `}` 包围，例如 `#{session_name}`。可能的变量在下表中列出，或者可以使用 tmux 选项的名称作为选项的值。一些变量有较短的别名，例如 `#S`；`##` 被 `#` 替换，`#` 被 `,` 替换，`#}` 被 `}` 替换。

条件可通过前缀 `?` 并用逗号分隔两个替代方案来实现；如果指定变量存在且不为零，则选择第一个替代方案，否则使用第二个。例如 `#{?session_attached,attached,not attached}` 将包括字符串 `attached` 如果会话附加，否则包括字符串 `not attached`，或 `#{?automatic-rename,yes,no}` 将包括 `yes` 如果启用 automatic-rename，否则包括 `no`。条件可以任意嵌套。在条件内部，`,` 和 `}` 必须转义为 `#` 和 `#}`，除非它们是 `#{...}` 替换的一部分。例如：

```
#{?pane_in_mode,#[fg=white#,bg=red],#[fg=red#,bg=white]}#W .
```

字符串比较可通过前缀两个逗号分隔的替代方案 `==`、`!=`、`<`、`>`、`<=` 或 `>=` 和冒号来表达。例如 `#{==:#{host},myhost}` 将被替换为 `1` 如果在 `myhost` 上运行，否则为 `0`。`||` 和 `&&` 如果两个逗号分隔的替代方案中的一个或两个为真，则计算为真，例如 `#{||:#{pane_in_mode},#{alternate_on}}`。

`m` 指定 fnmatch(3) 或正则表达式比较。第一个参数是模式，第二个是要比较的字符串。可选参数指定标志：`r` 表示模式是正则表达式而不是默认的 fnmatch(3) 模式，`i` 表示忽略大小写。例如：`#{m:*foo*,#{host}}` 或 `#{m/ri:^A,MYVAR}`。`C` 在窗格内容中搜索 fnmatch(3) 模式或正则表达式，如果未找到则计算为零，如果找到则计算为行号。像 `m` 一样，`r` 标志表示搜索正则表达式，`i` 忽略大小写。例如：`#{C/r:^Start}`

数值运算可通过前缀两个逗号分隔的替代方案 `e` 和运算符来执行。可选的 `f` 标志可在运算符后给出，以使用浮点数，否则使用整数。这可以后跟一个数字，给出结果的小数位数。可用的运算符包括：加法 `+`、减法 `-`、乘法 `*`、除法 `/`、模数 `m` 或 `%`（注意在由 strftime(3) 扩展的格式中，`%` 必须转义为 `%%`）和数值比较运算符 `==`、`!=`、`<`、`<=`、`>` 和 `>=`。例如，`#{e|*|f|4:5.5,3}` 将 5.5 乘以 3，结果保留四位小数，`#{e|%%:7,3}` 返回 7 和 3 的模数。`a` 用其 ASCII 等价物替换数值参数，因此 `#{a:98}` 结果为 `b`。`c` 用其六位十六进制 RGB 值替换 tmux 颜色。

可以通过前缀 `=`、数字和冒号来限制结果字符串的长度。正数从字符串的开头计数，负数从结尾计数，因此 `#{=5:pane_title}` 将最多包括窗格标题的前五个字符，或 `#{=-5:pane_title}` 最后五个字符。可以给定后缀或前缀作为第二个参数——如果提供，则在长度被修剪时附加或前置到字符串，例如 `#{=/5/...:pane_title}` 如果窗格标题超过五个字符，则附加 `...`。同样，`p` 将字符串填充到给定宽度，例如 `#{p10:pane_title}` 将导致至少 10 个字符的宽度。正宽度在左侧填充，负宽度在右侧填充。`n` 扩展为变量的长度，`w` 扩展为其显示时的宽度，例如 `#{n:window_name}`。

用 `t:` 前缀时间变量将将其转换为字符串，因此如果 `#{window_activity}` 给出 `1445765102`，`#{t:window_activity}` 给出 `Sun Oct 25 09:25:02 2015`。添加 `p (` ``t/p``) 将使用较短但不太精确的时间格式表示过去的时

间。可以使用 `f` 后缀给出自定义格式（注意，如果格式单独通过 strftime(3) 传递，例如在 status-left 选项中，`%` 必须转义为 `%%`）：`#{t/f/%%H#:%%M:window_activity}`，参见 strftime(3)。

`b:` 和 `d:` 前缀分别是变量的 basename(3) 和 dirname(3)。`q:` 将转义 sh(1) 特殊字符，或使用 `h` 后缀转义哈希字符（因此 `#` 变为 `##`）。`E:` 将扩展格式两次，例如 `#{E:status-left}` 是扩展 status-left 选项内容的结果，而不是选项本身。`T:` 与 `E:` 类似，但也扩展 strftime(3) 指定符。`S:`、`W:`、`P:` 或 `L:` 将遍历每个会话、窗口、窗格或客户端，并为每个插入一次格式。对于窗口和窗格，可以给出两个逗号分隔的格式：第二个用于当前窗口或活动窗格。例如，获取类似状态行的窗口列表：

```
#{W:#{E:window-status-format} ,#{E:window-status-current-format} }
```

`N:` 检查窗口（没有后缀或带有 `w` 后缀）或会话（带有 `s` 后缀）名称是否存在，例如 ``N/w:foo`` 如果存在名为 `foo` 的窗口，则替换为 1。

形式为 `s/foo/bar/:` 的前缀将用 `bar` 替换 `foo`。第一个参数可以是扩展正则表达式，最终参数可以是 `i` 以忽略大小写，例如 `s/a(.)/\1x/i:` 将 `abABab` 更改为 `bxBxbx`。可以使用不同的分隔符字符，以避免与模式中的文字斜杠冲突。例如，`s|foo/|bar/|:` 将用 `bar/` 替换 `foo/`。

此外，可以使用 `#()` 插入 shell 命令输出的最后一行。例如，`#(uptime)` 将插入系统的运行时间。在构建格式时，tmux 不等待 `#()` 命令完成；而是使用上次运行相同命令的结果，或者如果命令之前未运行则使用占位符。如果命令尚未退出，则使用输出的最新行，但状态行不会更新超过每秒一次。命令使用 /bin/sh 执行，并设置 tmux 全局环境（参见“全局和会话环境”部分）。

`l` 指定字符串应按字面解释而不扩展。例如 `#{l:#{?pane_in_mode,yes,no}}` 将被替换为 `#{?pane_in_mode,yes,no}`。

以下变量在适当情况下可用：

变量名          别名    替换内容
active_window_index             会话中活动窗口的索引
alternate_on                    如果窗格在备用屏幕中则为 1
alternate_saved_x               备用屏幕中的保存光标 X
alternate_saved_y               备用屏幕中的保存光标 Y
buffer_created                  缓冲区创建时间
buffer_name                     缓冲区名称
buffer_sample                   缓冲区开始的样本
buffer_size                     指定缓冲区的大小（以字节为单位）
client_activity                 客户端最后活动时间
client_cell_height              每个客户端单元格的高度（以像素为单位）
client_cell_width               每个客户端单元格的宽度（以像素为单位）
client_control_mode             如果客户端处于控制模式则为 1
client_created                  客户端创建时间
client_discarded                客户端落后时丢弃的字节数
client_flags                    客户端标志列表
client_height                   客户端高度
client_key_table                当前键表
client_last_session             客户端的上一个会话名称
client_name                     客户端名称
client_pid                      客户端进程的 PID
client_prefix                   如果按下了前缀键则为 1
client_readonly                 如果客户端是只读的则为 1
client_session                  客户端的会话名称
client_termfeatures             客户端的终端功能（如果有）
client_termname                 客户端的终端名称
client_termtype                 客户端的终端类型（如果可用）
client_tty                      客户端的伪终端
client_uid                      客户端进程的 UID
client_user                     客户端进程的用户
client_utf8                     如果客户端支持 UTF-8 则为 1
client_width                    客户端宽度
client_written                  写入客户端的字节数
command                         使用中的命令名称（如果有）
command_list_alias              如果列出命令则为命令别名
command_list_name               如果列出命令则为命令名称
command_list_usage              如果列出命令则为命令用法
config_files                    加载的配置文件列表
copy_cursor_hyperlink           复制模式中光标下的超链接
copy_cursor_line                复制模式中的光标行
copy_cursor_word                复制模式中光标下的单词
copy_cursor_x                   复制模式中的光标 X 位置
copy_cursor_y                   复制模式中的光标 Y 位置
current_file                    当前配置文件
cursor_character                窗格中的光标字符
cursor_flag                     窗格光标标志
cursor_x                        窗格中的光标 X 位置
cursor_y                        窗格中的光标 Y 位置
history_bytes                   窗口历史中的字节数
history_limit                   窗口历史最大行数
history_size                    历史大小（以行数为单位）
hook                            运行中的钩子名称（如果有）
hook_client                     运行钩子的客户端名称（如果有）
hook_pane                       运行钩子的窗格 ID（如果有）
hook_session                    运行钩子的会话 ID（如果有）
hook_session_name               运行钩子的会话名称（如果有）
hook_window                     运行钩子的窗口 ID（如果有）
hook_window_name                运行钩子的窗口名称（如果有）
host                   #H       本地主机的主机名
host_short             #h       本地主机的主机名（无域名）
insert_flag                     窗格插入标志
keypad_cursor_flag              窗格键盘光标标志
keypad_flag                     窗格键盘标志
last_window_index               会话中最后一个窗口的索引
line                            列表中的行号
mouse_all_flag                  窗格鼠标所有标志
mouse_any_flag                  窗格鼠标任何标志
mouse_button_flag               窗格鼠标按钮标志
mouse_hyperlink                 鼠标下的超链接（如果有）
mouse_line                      鼠标下的行（如果有）
mouse_sgr_flag                  窗格鼠标 SGR 标志
mouse_standard_flag             窗格鼠标标准标志
mouse_status_line               鼠标事件发生的状态行
mouse_status_range              状态行上鼠标事件的范围类型或参数
mouse_utf8_flag                 窗格鼠标 UTF-8 标志
mouse_word                      鼠标下的单词（如果有）
mouse_x                         鼠标 X 位置（如果有）
mouse_y                         鼠标 Y 位置（如果有）
next_session_id                 下一个新会话的唯一会话 ID
origin_flag                     窗格原点标志
pane_active                     如果是活动窗格则为 1
pane_at_bottom                  如果窗格在窗口底部则为 1
pane_at_left                    如果窗格在窗口左侧则为 1
pane_at_right                   如果窗格在窗口右侧则为 1
pane_at_top                     如果窗格在窗口顶部则为 1
pane_bg                         窗格背景颜色
pane_bottom                     窗格底部
pane_current_command            当前命令（如果可用）
pane_current_path               当前路径（如果可用）
pane_dead                       如果窗格已死则为 1
pane_dead_signal                死窗格中进程的退出信号
pane_dead_status                死窗格中进程的退出状态
pane_dead_time                  死窗格中进程的退出时间
pane_fg                         窗格前景颜色
pane_format                     如果格式是为窗格
pane_height                     窗格高度
pane_id                #D       唯一窗格 ID
pane_in_mode                    如果窗格处于模式中则为 1
pane_index             #P       窗格索引
pane_input_off                  如果窗格输入被禁用则为 1
pane_key_mode                   此窗格中的扩展键报告模式
pane_last                       如果是最后一个窗格则为 1
pane_left                       窗格左侧
pane_marked                     如果这是标记的窗格则为 1
pane_marked_set                 如果设置了标记的窗格则为 1
pane_mode                       窗格模式名称（如果有）
pane_path                       窗格路径（可以由应用程序设置）
pane_pid                        窗格中第一个进程的 PID
pane_pipe                       如果窗格正在被管道传输则为 1
pane_right                      窗格右侧
pane_search_string              复制模式中的上次搜索字符串
pane_start_command              窗格启动时的命令
pane_start_path                 窗格启动时的路径
pane_synchronized               如果窗格是同步的则为 1
pane_tabs                       窗格制表位
pane_title             #T       窗格标题（可以由应用程序设置）
pane_top                        窗格顶部
pane_tty                        窗格的伪终端
pane_unseen_changes             如果在模式中窗格有更改则为 1
pane_width                      窗格宽度
pid                             服务器 PID
rectangle_toggle                如果矩形选择已激活则为 1
scroll_position                 复制模式中的滚动位置
scroll_region_lower             窗格中滚动区域的底部
scroll_region_upper             窗格中滚动区域的顶部
search_count                    搜索结果计数
search_count_partial            如果搜索计数是部分计数则为 1
search_match                    搜索匹配（如果有）
search_present                  如果在复制模式中开始搜索则为 1
selection_active                如果在复制模式中开始选择并且随光标更改则为 1
selection_end_x                 选择结束的 X 位置
selection_end_y                 选择结束的 Y 位置
selection_present               如果在复制模式中开始选择则为 1
selection_start_x               选择开始的 X 位置
selection_start_y               选择开始的 Y 位置
server_sessions                 会话数
session_activity                会话最后活动时间
session_alerts                  有警报的窗口索引列表
session_attached                会话附加的客户端数
session_attached_list           会话附加的客户端列表
session_created                 会话创建时间
session_format                  如果格式是为会话
session_group                   会话组名称
session_group_attached          会话组附加的客户端数
session_group_attached_list     会话组附加的客户端列表
session_group_list              会话组中的会话列表
session_group_many_attached     如果会话组中的多个客户端附加则为 1
session_group_size              会话组大小
session_grouped                 如果会话在组中则为 1
session_id                      唯一会话 ID
session_last_attached           会话最后附加时间
session_many_attached           如果多个客户端附加则为 1
session_marked                  如果此会话包含标记的窗格则为 1
session_name           #S       会话名称
session_path                    会话工作目录
session_stack                   按最近顺序的窗口索引
session_windows                 会话中的窗口数
socket_path                     服务器套接字路径
start_time                      服务器启动时间
uid                             服务器 UID
user                            服务器用户
version                         服务器版本
window_active                   如果窗口活动则为 1
window_active_clients           查看此窗口的客户端数
window_active_clients_list      查看此窗口的客户端列表
window_active_sessions          此窗口活动的会话数
window_active_sessions_list     此窗口活动的会话列表
window_activity                 窗口最后活动时间
window_activity_flag            如果窗口有活动则为 1
window_bell_flag                如果窗口有铃声则为 1
window_bigger                   如果窗口大于客户端则为 1
window_cell_height              每个单元格的高度（以像素为单位）
window_cell_width               每个单元格的宽度（以像素为单位）
window_end_flag                 如果窗口具有最高索引则为 1
window_flags           #F       窗口标志，# 转义为 ##
window_format                   如果格式是为窗口
window_height                   窗口高度
window_id                       唯一窗口 ID
window_index           #I       窗口索引
window_last_flag                如果窗口是最后使用的则为 1
window_layout                   窗口布局描述，忽略缩放的窗口窗格
window_linked                   如果窗口跨会话链接则为 1
window_linked_sessions          此窗口链接的会话数
window_linked_sessions_list     此窗口链接的会话列表
window_marked_flag              如果窗口包含标记的窗格则为 1
window_name            #W       窗口名称
window_offset_x                 如果窗口大于客户端则为窗口中的 X 偏移
window_offset_y                 如果窗口大于客户端则为窗口中的 Y 偏移
window_panes                    窗口中的窗格数
window_raw_flags                窗口标志，不转义
window_silence_flag             如果窗口有静音警报则为 1
window_stack_index              会话最近堆栈中的索引
window_start_flag               如果窗口具有最低索引则为 1
window_visible_layout           窗口布局描述，尊重缩放的窗口窗格
window_width                    窗口宽度
window_zoomed_flag              如果窗口缩放则为 1
wrap_flag                       窗格换行标志

## 样式
tmux 提供各种选项来指定界面各方面的颜色和属性，例如 status-style 用于状态行。此外，可以在格式选项（如 status-left）中指定嵌入样式，通过将它们包含在 `#[' 和 `]' 中。

样式可以是单个术语 `default` 以指定默认样式（可能来自选项，例如状态行中的 status-style）或由以下组成的空格或逗号分隔列表：

fg=colour
        设置前景颜色。颜色是以下之一：black、red、green、yellow、blue、magenta、cyan、white；如果支持，则为 brightred、brightgreen、brightyellow 的亮变体；从 256 色集中为 colour0 到 colour255；default 为默认颜色；terminal 为终端默认颜色；或十六进制 RGB 字符串，如 `#ffffff`。

bg=colour
        设置背景颜色。

us=colour
        设置下划线颜色。

none    不设置任何属性（关闭任何活动属性）。

acs, bright (或 bold), dim, underscore, blink, reverse, hidden, italics, overline, strikethrough, double-underscore, curly-underscore, dotted-underscore, dashed-underscore
        设置属性。任何属性都可以用 `no` 前缀取消设置。acs 是终端备用字符集。

align=left (或 noalign), align=centre, align=right
        如果适当，将文本左对齐、居中或右对齐到可用空间。

fill=colour
        如果适当，用背景颜色填充可用空间。

list=on, list=focus, list=left-marker, list=right-marker, nolist
        标记状态格式选项中各种窗口列表组件的位置：list=on 标记列表的开始；list=focus 是列表中应保持焦点的部分，如果整个列表无法适应可用空间（通常是当前窗口）；list=left-marker 和 list=right-marker 标记从左侧或右侧修剪文本时使用的文本，如果空间不足。

push-default, pop-default
        存储当前颜色和属性作为默认值或重置为之前的默认值。push-default 影响任何后续使用默认术语，直到 pop-default。只能推送一个默认值（每个 push-default 替换之前保存的默认值）。

range=left, range=right, range=session|X, range=window|X, range=pane|X, range=user|X, norange
        标记状态格式选项中的鼠标事件范围。当在 range=left 或 range=right 范围内发生鼠标事件时，触发 `StatusLeft` 和 `StatusRight` 键绑定。

        range=session|X, range=window|X 和 range=pane|X 是会话、窗口或窗格的范围。这些触发 `Status` 鼠标键，目标会话、窗口或窗格由 `X` 参数给出。`X` 是会话 ID、当前会话中的窗口索引或窗格 ID。对于这些，mouse_status_range 格式变量将设置为 `session`、`window` 或 `pane`。

        range=user|X 是用户定义的范围；它触发 `Status` 鼠标键。参数 `X` 将在 mouse_status_range 格式变量中可用。`X` 的长度最多为 15 字节。

示例包括：

```
fg=yellow bold underscore blink
bg=black,fg=default,noreverse
```

## 名称和标题
tmux 区分会话和窗口的名称和标题。窗口和会话有名称，可用于在目标中指定它们，并显示在状态行和各种列表中：名称是 tmux 的窗口或会话标识符。只有窗格有标题。窗格的标题通常由在窗格内运行的程序使用转义序列设置（就像它在 X(7) 中设置 xterm(1) 窗口标题一样）。窗口本身没有标题——窗口的标题是其活动窗格的标题。tmux 本身可能会设置运行客户端的终端的标题，请参见 set-titles 选项。

会话的名称通过 new-session 和 rename-session 命令设置。窗口的名称通过以下方式之一设置：

1. 命令参数（如 new-window 或 new-session 的 -n）。

2. 转义序列（如果 allow-rename 选项打开）：

```
$ printf '\033kWINDOW_NAME\033\\'
```

3. 自动重命名，将名称设置为窗口活动窗格中的活动命令。参见 automatic-rename 选项。

当窗格首次创建时，其标题是主机名。可以使用标题设置转义序列修改窗格的标题，例如：

```
$ printf '\033]2;My Title\033\\'
```

也可以使用 select-pane -T 命令修改。

## 全局和会话环境
当服务器启动时，tmux 将环境复制到全局环境中；此外，每个会话都有一个会话环境。创建窗口时，会话和全局环境会合并。如果变量在两者中都存在，则使用会话环境的值。结果是传递给新进程的初始环境。

update-environment 会话选项可用于在创建新会话或重新附加旧会话时从客户端更新会话环境。tmux 还使用一些内部信息初始化 TMUX 变量，以允许从内部执行命令，并使用 `screen` 的正确终端设置 TERM 变量。

会话和全局环境中的变量可以标记为隐藏。隐藏变量不会传递到新进程的环境中，只能由 tmux 本身使用（例如在格式中，请参见“格式”部分）。

用于更改和查看环境的命令如下：

set-environment [-Fhgru] [-t target-session] name [value]
                (别名: setenv)
设置或取消设置环境变量。如果使用 -g，则更改在全局环境中进行；否则，应用于 target-session 的会话环境。如果存在 -F，则 value 被扩展为格式。-u 标志取消设置变量。-r 表示在启动新进程之前从环境中移除变量。-h 将变量标记为隐藏。

show-environment [-hgs] [-t target-session] [variable]
                (别名: showenv)
显示 target-session 或全局环境（使用 -g）。如果省略 variable，则显示所有变量。从环境中移除的变量以 `-` 为前缀。如果使用 -s，则输出格式为一组 Bourne shell 命令。-h 显示隐藏变量（默认情况下省略）。

## 状态行
tmux 包含一个可选的状态行，显示在每个终端的底行。

默认情况下，状态行启用且高度为一行（可以使用 status 会话选项禁用或设置为多行），并包含从左到右：当前会话的名称（用方括号括起来）；窗口列表；活动窗格的标题（用双引号括起来）；以及时间和日期。

状态行的每一行都通过 status-format 选项配置。默认由三部分组成：可配置的左右部分（可能包含动态内容，如时间或 shell 命令的输出，请参见 status-left、status-left-length、status-right 和 status-right-length 选项），以及中央窗口列表。默认情况下，窗口列表按升序数字显示当前会话中存在的窗口的索引、名称和（如果有）标志。可以使用 window-status-format 和 window-status-current-format 选项自定义。标志是附加到窗口名称的以下符号之一：

        符号    含义
        *         表示当前窗口。
        -         标记最后一个窗口（之前选择的）。
        #         监视窗口活动并检测到活动。
        !         监视窗口铃声并在窗口中发生铃声。
        ~         窗口在 monitor-silence 间隔内保持静音。
        M         窗口包含标记的窗格。
        Z         窗口的活动窗格已缩放。

        # 符号与 monitor-activity 窗口选项相关。如果存在警报（铃声、活动或静音），窗口名称将以反色打印。

        可以配置状态行的颜色和属性，使用 status-style 会话选项配置整个状态行，使用 window-status-style 窗口选项配置各个窗口。

        状态行在更改时自动刷新，间隔可以使用 status-interval 会话选项控制。

        与状态行相关的命令如下：

clear-prompt-history [-T prompt-type]
                (别名: clearphist)
清除 prompt-type 类型的状态提示历史。如果省略 -T，则清除所有类型的历史。有关 prompt-type 的可能值，请参见 command-prompt。

command-prompt [-1bFikN] [-I inputs] [-p prompts] [-t target-client] [-T prompt-type] [template]
在客户端中打开命令提示符。这可以在 tmux 内部用于交互式执行命令。

如果指定 template，则用作命令。使用 -F，template 被扩展为格式。

如果存在，-I 是每个提示的初始文本的逗号分隔列表。如果给定 -p，则 prompts 是要按顺序显示的提示的逗号分隔列表；否则显示单个提示，如果存在则从 template 构造，否则为 `:`。

在执行命令之前，响应的第一个出现的字符串 `%%` 和所有出现的 `%1` 被第一个提示的响应替换，所有 `%2` 被第二个提示的响应替换，依此类推进一步的提示。最多可以替换九个提示响应（`%1` 到 `%9`）。`%%%` 与 `%%` 相同，但任何引号都被转义。

-1 使提示只接受一次按键，在这种情况下，结果输入是单个字符。-k 与 -1 相同，但按键被转换为键名。-N 使提示只接受数字按键。-i 在每次提示输入更改时执行命令，而不是在用户退出命令提示时执行。

-T 告诉 tmux 提示类型。这会影响按下 Tab 时提供的补全。可用类型包括：`command`、`search`、`target` 和 `window-target`。

命令提示符中的以下键具有特殊含义，取决于 status-keys 选项的值：

        功能                             vi        emacs
        取消命令提示符                q         Escape
        从光标删除到单词开始            C-w
        删除整个命令                d         C-u
        从光标删除到结束            D         C-k
        执行命令                      Enter     Enter
        从历史获取下一个命令                  Down
        从历史获取上一个命令                  Up
        插入顶部粘贴缓冲区              p         C-y
        查找补全                         Tab       Tab
        光标左移                     h         Left
        光标右移                     l         Right
        光标移至结束                   $         C-e
        光标移至下一个单词             w         M-f
        光标移至上一个单词             b         M-b
        光标移至开始                   0         C-a
        交换字符                           C-t

使用 -b，提示在后台显示，调用客户端不会退出，直到它被关闭。

confirm-before [-by] [-c confirm-key] [-p prompt] [-t target-client] command
                (别名: confirm)
在执行命令之前要求确认。如果给定 -p，则 prompt 是要显示的提示；否则从 command 构造提示。它可能包含 status-left 选项支持的特殊字符序列。使用 -b，提示在后台显示，调用客户端不会退出，直到它被关闭。-y 更改提示的默认行为（如果仅按下 Enter），则运行命令。-c 更改确认键为 confirm-key；默认为 `y`。

display-menu [-OM] [-b border-lines] [-c target-client] [-C starting-choice] [-H selected-style] [-s style] [-S border-style] [-t target-pane] [-T title] [-x position] [-y position] name key command [argument ...]
                (别名: menu)
在 target-client 上显示菜单。target-pane 给出从菜单运行的命令的目标。

菜单作为一系列参数传递：首先是菜单项名称，其次是快捷键（或空表示无），第三是选择菜单项时运行的命令。名称和命令是格式，请参见“格式”和“样式”部分。如果名称以连字符 (-) 开头，则项目被禁用（显示为暗淡）且不能被选择。如果名称为空，则为分隔线，在这种情况下，快捷键和命令应省略。

-b 设置用于绘制菜单边框的字符类型。有关 border-lines 的可能值，请参见 popup-border-lines。

-H 设置选定菜单项的样式（参见“样式”）。

-s 设置菜单的样式，-S 设置菜单边框的样式（参见“样式”）。

-T 是菜单标题的格式（参见“格式”）。

-C 设置默认选择的菜单项，如果菜单未绑定到鼠标键绑定。

-x 和 -y 给出菜单的位置。两者可以是行号或列号，或以下特殊值之一：

        值    标志    含义
        C        Both    终端的中心
        R        -x      终端的右侧
        P        Both    窗格的左下角
        M        Both    鼠标位置
        W        Both    状态行中的窗口位置
        S        -y      状态行上方或下方的行

或格式，其中包括以下附加变量：

        变量名                 替换内容
        popup_centre_x                在客户端中居中
        popup_centre_y                在客户端中居中
        popup_height                  菜单或弹出窗口的高度
        popup_mouse_bottom            鼠标底部
        popup_mouse_centre_x          鼠标水平居中
        popup_mouse_centre_y          鼠标垂直居中
        popup_mouse_top               鼠标顶部
        popup_mouse_x                 鼠标 X 位置
        popup_mouse_y                 鼠标 Y 位置
        popup_pane_bottom             窗格底部
        popup_pane_left               窗格左侧
        popup_pane_right              窗格右侧
        popup_pane_top                窗格顶部
        popup_status_line_y           状态行上方或下方
        popup_width                   菜单或弹出窗口的宽度
        popup_window_status_line_x    状态行中的窗口位置
        popup_window_status_line_y    显示窗口的状态行

每个菜单由项目组成，后跟括号中显示的快捷键。如果菜单太大而无法适应终端，则不会显示。按下快捷键选择相应的项目。如果启用鼠标且菜单从鼠标键绑定打开，则释放鼠标按钮时选择项目，如果未选择项目则关闭菜单。-O 更改此行为，因此如果释放鼠标按钮时未选择项目，则菜单不会关闭，并且必须单击鼠标按钮以选择项目。

-M 告诉 tmux 菜单应处理鼠标事件；默认情况下，只有从鼠标键绑定打开的菜单才这样做。

菜单中可用以下键：

        键    功能
        Enter  选择选定的项目
        Up     选择上一个项目
        Down   选择下一个项目
        q      退出菜单

display-message [-aIlNpv] [-c target-client] [-d delay] [-t target-pane] [message]
                (别名: display)
显示消息。如果给定 -p，则输出打印到 stdout，否则显示在 target-client 状态行中，最多 delay 毫秒。如果未给定 delay，则使用 display-time 选项；delay 为零等待按键。`N` 忽略按键并在 delay 到期后关闭。如果给定 -l，则 message 不变地打印。否则，message 的格式在“格式”部分中描述；如果给定 -t，则从 target-pane 获取信息，否则从活动窗格获取。

-v 打印解析格式时的详细日志，-a 列出格式变量及其值。

-I 将从 stdin 读取的任何输入转发到 target-pane 给出的空窗格。

display-popup [-BCE] [-b border-lines] [-c target-client] [-d start-directory] [-e environment] [-h height] [-s border-style] [-S style] [-t target-pane] [-T title] [-w width] [-x position] [-y position] [shell-command]
                (别名: popup)
在 target-client 上显示运行 shell-command 的弹出窗口。弹出窗口是一个矩形框，绘制在任何窗格的顶部。存在弹出窗口时，窗格不会更新。

-E 在 shell-command 退出时自动关闭弹出窗口。两个 -E 仅在 shell-command 退出成功时关闭弹出窗口。

-x 和 -y 给出弹出窗口的位置，它们与 display-menu 命令具有相同的含义。-w 和 -h 给出宽度和高度——两者都可以是百分比（后跟 `%`）。如果省略，则使用终端大小的一半。

-B 不用边框包围弹出窗口。

-b 设置用于绘制弹出窗口边框的字符类型。当指定 -B 时，-b 选项被忽略。有关 border-lines 的可能值，请参见 popup-border-lines。

-s 设置弹出窗口的样式，-S 设置弹出窗口边框的样式（参见“样式”）。

-e 采用 `VARIABLE=value` 形式，并为弹出窗口设置环境变量；可以指定多次。

-T 是弹出窗口标题的格式（参见“格式”）。

-C 标志关闭客户端上的任何弹出窗口。

show-prompt-history [-T prompt-type]
                (别名: showphist)
显示 prompt-type 类型的状态提示历史。如果省略 -T，则显示所有类型的历史。有关 prompt-type 的可能值，请参见 command-prompt。

## 缓冲区
tmux 维护一组命名的粘贴缓冲区。每个缓冲区可以是显式命名或自动命名。显式命名的缓冲区在使用 set-buffer 或 load-buffer 命令创建时命名，或通过使用 set-buffer -n 重命名自动命名的缓冲区。自动命名的缓冲区被命名为 `buffer0001`、`buffer0002` 等。当达到 buffer-limit 选项时，最旧的自动命名缓冲区被删除。显式命名的缓冲区不受 buffer-limit 的限制，可以使用 delete-buffer 命令删除。

可以使用复制模式或 set-buffer 和 load-buffer 命令添加缓冲区，并使用 paste-buffer 命令将缓冲区粘贴到窗口中。如果使用缓冲区命令且未指定缓冲区，则假定为最近添加的自动命名缓冲区。

还为每个窗口维护一个可配置的历史缓冲区。默认情况下，保留最多 2000 行；可以使用 history-limit 选项更改（参见上面的 set-option 命令）。

缓冲区命令如下：

choose-buffer [-NZr] [-F format] [-f filter] [-K key-format] [-O sort-order] [-t target-pane] [template]
将窗格置于缓冲区模式，可以从列表中交互式选择缓冲区。每个缓冲区显示在一行上。左侧显示快捷键，允许立即选择，或者可以导航列表并选择或以其他方式操作项目。-Z 缩放窗格。在缓冲区模式中可以使用以下键：

        键    功能
        Enter  粘贴选定的缓冲区
        Up     选择上一个缓冲区
        Down   选择下一个缓冲区
        C-s    按名称或内容搜索
        n      向前重复上次搜索
        N      向后重复上次搜索
        t      切换缓冲区是否被标记
        T      取消标记所有缓冲区
        C-t    标记所有缓冲区
        p      粘贴选定的缓冲区
        P      粘贴标记的缓冲区
        d      删除选定的缓冲区
        D      删除标记的缓冲区
        e      在编辑器中打开缓冲区
        f      输入格式以过滤项目
        O      更改排序字段
        r      反向排序
        v      切换预览
        q      退出模式

选择缓冲区后，`%%` 在 template 中被缓冲区名称替换，并将结果作为命令执行。如果未给定 template，则使用 "paste-buffer -p -b '%%'"。

-O 指定初始排序字段：`time`（创建）、`name` 或 `size` 之一。-r 反向排序。-f 指定初始过滤器：过滤器是格式——如果计算结果为零，则列表中的项目不显示，否则显示。如果过滤器会导致空列表，则忽略。-F 指定列表中每个项目的格式，-K 指定每个快捷键的格式；两者都为每行计算一次。-N 开始时不显示预览。此命令仅在至少有一个客户端附加时有效。

clear-history [-H] [-t target-pane]
                (别名: clearhist)
删除并释放指定窗格的历史。-H 还删除所有超链接。

delete-buffer [-b buffer-name]
                (别名: deleteb)
删除名为 buffer-name 的缓冲区，或如果未指定，则删除最近添加的自动命名缓冲区。

list-buffers [-F format] [-f filter]
                (别名: lsb)
列出全局缓冲区。-F 指定每行的格式，-f 指定过滤器。仅显示过滤器为真的缓冲区。参见“格式”部分。

load-buffer [-w] [-b buffer-name] [-t target-client] path
                (别名: loadb)
从 path 加载指定粘贴缓冲区的内容。如果给定 -w，则缓冲区也使用 xterm(1) 转义序列发送到 target-client 的剪贴板（如果可能）。如果 path 是 `-'，则从 stdin 读取内容。

paste-buffer [-dpr] [-b buffer-name] [-s separator] [-t target-pane]
                (别名: pasteb)
将指定粘贴缓冲区的内容插入到指定窗格中。如果未指定，则粘贴到当前窗格中。使用 -d，也删除粘贴缓冲区。输出时，粘贴缓冲区中的任何换行 (LF) 字符都会被分隔符替换，默认为回车 (CR)。可以使用 -s 标志指定自定义分隔符。-r 标志表示不进行替换（相当于分隔符为 LF）。如果指定 -p，则如果应用程序已请求括号粘贴模式，则在缓冲区周围插入括号控制代码。

save-buffer [-a] [-b buffer-name] path
                (别名: saveb)
将指定粘贴缓冲区的内容保存到 path。-a 选项附加到而不是覆盖文件。如果 path 是 `-'，则从 stdin 读取内容。

set-buffer [-aw] [-b buffer-name] [-t target-client] [-n new-buffer-name] data
                (别名: setb)
将指定缓冲区的内容设置为 data。如果给定 -w，则缓冲区也使用 xterm(1) 转义序列发送到 target-client 的剪贴板（如果可能）。-a 选项附加到而不是覆盖缓冲区。-n 选项将缓冲区重命名为 new-buffer-name。

show-buffer [-b buffer-name]
                (别名: showb)
显示指定缓冲区的内容。

## 杂项
杂项命令如下：

clock-mode [-t target-pane]
显示一个大时钟。

if-shell [-bF] [-t target-pane] shell-command command [command]
                (别名: if)
如果 shell-command（使用 /bin/sh 运行）成功，则执行第一个命令，否则执行第二个命令。在执行之前，shell-command 使用“格式”部分中指定的规则扩展，包括与 target-pane 相关的规则。使用 -b，shell-command 在后台运行。

如果给定 -F，则不执行 shell-command，但如果扩展后既不为空也不为零，则视为成功。

lock-server
                (别名: lock)
通过运行 lock-command 选项指定的命令单独锁定每个客户端。

run-shell [-bC] [-c start-directory] [-d delay] [-t target-pane] [shell-command]
                (别名: run)
使用 /bin/sh（或使用 -C）在后台执行 shell-command 而不创建窗口。在执行之前，shell-command 使用“格式”部分中指定的规则扩展。使用 -b，命令在后台运行。-d 等待 delay 秒后开始命令。如果给定 -c，则当前工作目录设置为 start-directory。如果未给定 -C，则任何 stdout 输出在命令完成后显示在视图模式中（在 -t 指定的窗格中或省略时在当前窗格中）。如果命令失败，则也显示退出状态。

wait-for [-L | -S | -U] channel
                (别名: wait)
在不使用选项时，防止客户端退出，直到使用 wait-for -S 使用同一通道唤醒。使用 -L 时，通道被锁定，任何尝试锁定同一通道的客户端都会等待通道被 wait-for -U 解锁。

## 退出消息
当 tmux 客户端分离时，它会打印一条消息。这可能是以下之一：

detached (from session ...)
        客户端正常分离。

detached and SIGHUP
        客户端分离，其父进程发送了 SIGHUP 信号（例如使用 detach-client -P）。

lost tty
        客户端的 tty(4) 或 pty(4) 意外被销毁。

terminated
        客户端被 SIGTERM 杀死。

too far behind
        客户端处于控制模式，并且无法跟上来自 tmux 的数据。

exited  服务器在没有会话时退出。

server exited
        服务器在收到 SIGTERM 时退出。

server exited unexpectedly
        服务器崩溃或以其他方式退出，而未告诉客户端原因。

## TERMINFO 扩展
tmux 理解 terminfo(5) 的一些非官方扩展。通常不需要手动设置这些，而应使用 terminal-features 选项。

AX      现有扩展，告诉 tmux 终端支持默认颜色。

Bidi    告诉 tmux 终端支持 VTE 双向文本扩展。

Cs, Cr  设置光标颜色。第一个采用单个字符串参数，用于设置颜色；第二个不采用参数，恢复默认光标颜色。如果设置，可以使用如下序列从 tmux 内部更改光标颜色：

```
$ printf '\033]12;red\033\\'
```

颜色是 X(7) 颜色，请参见 XParseColor(3)。

Cmg, Clmg, Dsmg, Enmg
设置、清除、禁用或启用 DECSLRM 边距。如果终端报告其与 VT420 兼容，则自动设置这些。

Dsbp, Enbp
禁用和启用括号粘贴。如果存在 XT 功能，则自动设置这些。

Dseks, Eneks
禁用和启用扩展键。

Dsfcs, Enfcs
禁用和启用焦点报告。如果存在 XT 功能，则自动设置这些。

Hls     设置或清除超链接注释。

Nobr    告诉 tmux 终端不使用亮色显示粗体。

Rect    告诉 tmux 终端支持矩形操作。

Smol    启用 overline 属性。

Smulx   设置样式下划线。单个参数是以下之一：0 表示无下划线，1 表示正常下划线，2 表示双下划线，3 表示波浪下划线，4 表示点状下划线，5 表示虚线下划线。

Setulc, Setulc1, ol
设置下划线颜色或重置为默认。Setulc 用于 RGB 颜色，Setulc1 用于 ANSI 或 256 颜色。Setulc 参数是 (red * 65536) + (green * 256) + blue，其中每个都在 0 到 255 之间。

Ss, Se  设置或重置光标样式。如果设置，可以使用如下序列将光标更改为下划线：

```
$ printf '\033[4 q'
```

如果未设置 Se，则使用参数 0 的 Ss 重置光标样式。

Swd     设置工作目录通知的开始序列。使用标准 fsl 功能终止序列。

Sxl     表示终端支持 SIXEL。

Sync    开始（参数为 1）或结束（参数为 2）同步更新。

Tc      表示终端支持 `direct colour` RGB 转义序列（例如，\e[38;2;255;255;255m）。

如果支持，则用于初始化颜色转义序列（可以通过将 `initc` 和 `ccc` 功能添加到 tmux terminfo(5) 条目中启用）。

这相当于 RGB terminfo(5) 功能。

Ms      将当前缓冲区存储在主机终端的选择（剪贴板）中。请参见上面的 set-clipboard 选项和 xterm(1) 手册页。

XT      这是现有的扩展功能，tmux 使用它表示终端支持 xterm(1) 标题设置序列，并自动设置上述一些功能。

## 控制模式
tmux 提供一种称为控制模式的文本接口。这允许应用程序使用简单的纯文本协议与 tmux 通信。

在控制模式中，客户端在标准输入上发送以换行符终止的 tmux 命令或命令序列。每个命令将在标准输出上产生一个输出块。输出块由 %begin 行组成，后跟输出（可能为空）。输出块以 %end 或 %error 结束。%begin 和匹配的 %end 或 %error 有三个参数：整数时间（自纪元以来的秒数）、命令号和标志（目前未使用）。例如：

```
%begin 1363006971 2 1
0: ksh* (1 panes) [80x24] [layout b25f,80x24,0,0,2] @2 (active)
%end 1363006971 2 1
```

refresh-client -C 命令可用于设置控制模式下客户端的大小。

在控制模式中，tmux 输出通知。通知永远不会出现在输出块内部。

定义了以下通知：

%client-detached client
        客户端已分离。

%client-session-changed client session-id name
        客户端现在附加到 ID 为 session-id 的会话，名为 name。

%config-error error
        配置文件中发生了错误。

%continue pane-id
        窗格在暂停后已继续（如果设置了 pause-after 标志，请参见 refresh-client -A）。

%exit [reason]
        tmux 客户端立即退出，要么因为它未附加到任何会话，要么发生了错误。如果存在，reason 描述客户端退出的原因。

%extended-output pane-id age ... : value
        新形式的 ...[content truncated]

%unlinked-window-renamed window-id
        ID 为 window-id 的窗口（未链接到当前会话）已重命名。

%window-add window-id
        ID 为 window-id 的窗口已链接到当前会话。

%window-close window-id
        ID 为 window-id 的窗口已关闭。

%window-pane-changed window-id pane-id
        ID 为 window-id 的窗口中的活动窗格已更改为 ID 为 pane-id 的窗格。

%window-renamed window-id name
        ID 为 window-id 的窗口已重命名为 name。

## 环境
当 tmux 启动时，它会检查以下环境变量：

EDITOR    如果此变量中指定的命令包含字符串 `vi` 且未设置 VISUAL，则使用 vi 风格的键绑定。被 mode-keys 和 status-keys 选项覆盖。

HOME      用户的登录目录。如果未设置，则查询 passwd(5) 数据库。

LC_CTYPE  字符编码 locale(1)。它用于两个独立的目的。对于输出到终端，如果给定 -u 选项或 LC_CTYPE 包含 "UTF-8" 或 "UTF8"，则使用 UTF-8。否则，仅写入 ASCII 字符，非 ASCII 字符被下划线 (`_`) 替换。对于输入，tmux 始终以 UTF-8 locale 运行。如果操作系统提供 en_US.UTF-8，则使用它，并忽略 LC_CTYPE 用于输入。否则，LC_CTYPE 告诉 tmux 当前系统上的 UTF-8 locale 叫什么。如果 LC_CTYPE 指定的 locale 不可用或不是 UTF-8 locale，tmux 会以错误消息退出。

LC_TIME   日期和时间格式 locale(1)。它用于 locale 相关的 strftime(3) 格式指定符。

PWD       在全局环境中设置的当前工作目录。如果它包含符号链接，这可能很有用。如果变量的值与当前工作目录不匹配，则忽略该变量并使用 getcwd(3) 的结果。

SHELL     新窗口的默认 shell 的绝对路径。有关详细信息，请参见 default-shell 选项。

TMUX_TMPDIR
        包含服务器套接字的目录的父目录。有关详细信息，请参见 -L 选项。

VISUAL    如果此变量中指定的命令包含字符串 `vi`，则使用 vi 风格的键绑定。被 mode-keys 和 status-keys 选项覆盖。

## 文件
~/.tmux.conf
$XDG_CONFIG_HOME/tmux/tmux.conf
~/.config/tmux/tmux.conf
                   默认 tmux 配置文件。
/etc/tmux.conf     系统范围的配置文件。

## 示例
要创建运行 vi(1) 的新 tmux 会话：

```
$ tmux new-session vi
```

大多数命令有较短的形式，称为别名。对于 new-session，这是 new：

```
$ tmux new vi
```

或者，接受命令的最短明确形式。如果有几个选项，则列出它们：

```
$ tmux n
ambiguous command: n, could be: new-session, new-window, next-window
```

在活动会话中，可以通过键入 `C-b c`（Ctrl 后跟 `b` 键后跟 `c` 键）创建新窗口。

可以使用以下方式导航窗口：`C-b 0`（选择窗口 0），`C-b 1`（选择窗口 1）等；`C-b n` 选择下一个窗口；`C-b p` 选择上一个窗口。

会话可以使用 `C-b d` 分离（或通过外部事件如 ssh(1) 断开连接），并使用以下命令重新附加：

```
$ tmux attach-session
```

键入 `C-b ?` 列出当前窗口中的当前键绑定；可使用上和下导航列表或 `q` 退出。

在 tmux 服务器启动时运行的命令可以放在 ~/.tmux.conf 配置文件中。常见示例包括：

更改默认前缀键：

```
set-option -g prefix C-a
unbind-key C-b
bind-key C-a send-prefix
```

关闭状态行，或更改其颜色：

```
set-option -g status off
set-option -g status-style bg=blue
```

设置其他选项，例如默认命令，或在 30 分钟不活动后锁定：

```
set-option -g default-command "exec /bin/ksh"
set-option -g lock-after-time 1800
```

创建新键绑定：

```
bind-key b set-option status
bind-key / command-prompt "split-window 'exec man %%'"
bind-key S command-prompt "new-window -n %1 'ssh %1'"
```

## 另请参见
pty(4)

## 作者
Nicholas Marriott <nicholas.marriott@gmail.com>