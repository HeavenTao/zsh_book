~~~~
请注意：大多数显示问题都是由于 TERM 设置不正确！在报告问题之前，请确保 tmux 内外的 TERM 设置是正确的。

在 tmux 内，TERM 必须是 "screen"、"tmux" 或类似（如 "tmux-256color"）。
如果不符合此要求，请不要报告问题！

在 tmux 外，它应该与您的终端匹配：特别是，为 rxvt 和其衍生版本使用 "rxvt"。
~~~~
### 什么是 `TERM` 以及它的作用是什么？

环境变量 `TERM` 告诉应用程序从 *terminfo(5)* 数据库中读取的终端描述的名称。每个描述都包含许多命名功能，这些功能告诉应用程序发送什么内容来控制终端。例如，`cup` 功能包含用于移动光标向上的转义序列。

重要的是，`TERM` 指向应用程序运行的终端的正确描述 - 如果不是这样，应用程序可能会出现异常行为。

*infocmp(1)* 命令显示终端描述的内容，*tic(1)* 命令从文件构建和安装描述（通常需要 `-x` 标志）。

### 我在 tmux 中发现了一个错误！我该怎么办？

检查 Git 中的最新版本的 tmux，看看问题是否仍然存在。

请通过电子邮件发送错误报告至 *nicholas.marriott@gmail.com* 或 *tmux-users@googlegroups.com* 或打开 GitHub 问题。请参见 [CONTRIBUTING](https://github.com/tmux/tmux/blob/master/.github/CONTRIBUTING.md) 文件，了解包含信息的信息。

### 为什么 tmux 不做 $x？

请通过电子邮件发送功能请求至 *tmux-users@googlegroups.com* 或打开 GitHub 问题。

### tmux 多久发布一次？版本号方案是什么？

tmux 发布现在大约每六个月进行一次，与 OpenBSD 发布（tmux 是 OpenBSD 的一部分）大致同时进行，但不一定在同一天。

tmux 版本号（由 `tmux -V` 和 `display -p '#{version}'` 报告）匹配以下之一：

- 主要版本有一个小数点后一位的版本，如 2.9 或 3.0。数字随着每次发布而增加，因此是 2.8、2.9、3.0、3.1 等。

- 补丁版本在最后一位数字后跟一个字母，如 2.9a 或 3.0a。这些包含少量更改以修复发布后不久发现的错误。

- 开发源代码树（Git 中的 master）有一个以 "next-" 为前缀的版本，例如 next-3.1 是最终将成为 3.1 版本的代码。

- 发布候选版本有一个 "-rc" 后缀，可选地带一个数字。因此，第一个 3.1 发布候选版本是 3.1-rc，第二个是 3.1-rc2 等。

- OpenBSD 中的 tmux 没有版本号。在 OpenBSD 6.6 之前，它不支持 `-V` 标志；在 6.6 及以后版本中，它支持并报告以 "openbsd-" 为前缀的 OpenBSD 版本号，例如 openbsd-6.6。

除了上述补丁版本和发布候选版本之外，tmux 版本号没有特殊意义。tmux 3.0 是 tmux 2.9 之后的版本，仅此而已。

### 为什么在 tmux 内部使用 screen 终端描述？

它已经广泛可用。现代 *ncurses(3)* 提供了 `tmux` 和 `tmux-256color` 条目，并且可以通过设置 `default-terminal` 选项来使用它们。

### tmux 以 `server exited unexpectedly` 或 `lost server` 退出。这意味着什么？

此消息通常表示 tmux 服务器已崩溃。

如果可以重现问题，请打开一个问题报告 - [CONTRIBUTING](https://github.com/tmux/tmux/blob/master/.github/CONTRIBUTING.md) 文件中有关于包含内容和如何获取 tmux 日志的信息。

启用核心转储以查看 tmux 崩溃的原因也很有用。在大多数平台上，这可以通过在启动 tmux 之前运行 `ulimit -c unlimited` 来完成；如果它再次崩溃，它可能会在启动目录中生成一个核心文件。这可能被称为 `tmux.core` 或 `core.*` 或只是 `core`。它可以像这样加载到 `gdb` 中：

~~~~
$ gdb `which tmux` /path/to/core/file
...
(gdb) bt full
~~~~

如果这有效，请在问题中包含输出。

### 当我尝试附加时，tmux 说 `no sessions`，但我肯定有会话！

使用 `pgrep` 或 `ps` 检查 tmux 是否仍在运行。如果不是，则服务器可能已崩溃或被杀死，会话已消失。

如果 tmux 仍在运行，很可能某些东西删除了 `/tmp` 中的套接字。可以通过向 tmux 服务器发送 `USR1` 信号来要求它重新创建套接字，例如：`pkill -USR1 tmux`

### 我在我的终端中看不到任何颜色！救命！

在少数平台上，常见的终端描述（如 `xterm`）不包含颜色。screen 忽略这一点，tmux 不会。如果使用的终端模拟器支持颜色，请使用正确列出此内容的 `TERM` 值，如 `xterm-color`。

### 当我附加时，tmux 冻结了我的终端。我必须 `kill -9` 启动它的 shell 才能恢复！

某些控制台不喜欢尝试设置窗口标题。通过关闭 `set-titles` 选项告诉 tmux 不要这样做（您可以在 `.tmux.conf` 中这样做：

~~~~
set -g set-titles off
~~~~

如果这不能解决它，请发送错误报告。

### 为什么 C-b 是前缀键？我如何更改它？

默认键是 C-b，因为 tmux 的原型最初是在 screen 内开发的，C-b 是为了不与 screen 元键冲突而选择的。

要更改它，请更改 `prefix` 选项，如果需要，请将 `send-prefix` 命令的绑定从 C-b（C-b C-b 默认发送 C-b）移动到新键。例如：

~~~~
set -g prefix C-a
unbind C-b
bind C-a send-prefix
~~~~

### 如何使用 UTF-8？

tmux 需要支持 UTF-8 的系统（即 C 库具有 UTF-8 区域设置），如果缺少支持则不会启动。

tmux 将尝试通过查看 `LC_ALL`、`LC_CTYPE` 和 `LANG` 环境变量来检测它运行的终端是否支持 UTF-8。

如果它认为终端与 UTF-8 不兼容，任何 UTF-8 字符都将被下划线替换。`-u` 标志明确告诉 tmux 终端支持 UTF-8：

~~~~
$ tmux -u new
~~~~

### 如何使用 256 色终端？

如果底层终端支持 256 色，通常只需将以下内容之一添加到 `~/.tmux.conf`：

~~~~
set -g default-terminal "screen-256color"
~~~~

或：

~~~~
set -g default-terminal "tmux-256color"
~~~~

并确保 tmux 外部的 `TERM` 也显示 256 色，或者使用 tmux 的 `-2` 标志。

### 如何使用 RGB 颜色？

必须告诉 tmux 外部终端支持 RGB 颜色。这是通过指定 `RGB` 或 `Tc` *terminfo(5)* 标志完成的。`RGB` 是官方标志，`Tc` 是 tmux 扩展。

从 tmux 3.2 及以后版本，这可以通过 `terminal-features` 选项添加：

~~~~
set -as terminal-features ",gnome*:RGB"
~~~~

或对于任何 tmux 版本的 `terminal-overrides` 选项：

~~~~
set -as terminal-overrides ",gnome*:Tc"
~~~~

对于 tmux 本身，颜色可以用十六进制指定，例如 `bg=#ff0000`。

### 为什么 tmux 窗格分隔符是虚线而不是连续线？

某些终端或某些字体（特别是某些日文字体）不能正确处理 UTF-8 线条绘制字符。

`U8` 功能强制 tmux 使用 ACS 而不是 UTF-8 线条绘制：

~~~~
set -as terminal-overrides ",*:U8=0"
~~~~

### 我想用鼠标选择窗格，但用终端复制！怎么办？

终端不提供细粒度的鼠标支持 - tmux 可以打开鼠标并接收所有鼠标事件（点击、滚动、所有事件），或者它可以关闭鼠标并接收无事件。

因此，无法配置 tmux 使其处理某些鼠标行为而终端处理其他行为，这是全有或全无（`mouse` 选项 `on` 或 `off`）。

但是，当应用程序打开鼠标时，大多数终端提供了一种绕过它的方式。在许多 Linux 终端上，这是按住 `Shift` 键；对于 iTerm2，这是 `option` 键。

请注意，tmux 不尝试保持终端回滚一致（在多个窗口或窗格中无法做到这一点），所以它很可能是不完整的。

在 tmux 中禁用鼠标行为而不是让终端处理它是通过取消绑定适当的键绑定完成的，例如，要停止 tmux 在点击状态行时更改当前窗口，请在 root 表中取消绑定 `MouseDown1Status`：

~~~~
unbind -Troot MouseDown1Status
~~~~

### 如何将 `-fg`、`-bg` 和 `-attr` 选项转换为 `-style` 选项？

在 tmux 1.9 之前，样式（各种事物的颜色和属性）都是用三个选项配置的 - 一个用于前景色（如 `mode-fg`），一个用于背景色（如 `mode-bg`），一个用于属性（如 `mode-attr`）。

在 tmux 1.9 中，每组三个选项被合并为一个选项（所以 `mode-fg`、`mode-bg` 和 `mode-attr` 变成了 `mode-style`），在 tmux 2.9 中，旧选项被移除。所以例如：

~~~~
set -g mode-fg yellow
set -g mode-bg red
set -g mode-attr blink,underline
~~~~

应该改为：

~~~~
set -g mode-style fg=yellow,bg=red,blink,underline
~~~~

样式选项的格式在 [手册](https://man.openbsd.org/tmux.1#STYLES) 中有描述。

### 什么是 `escape-time` 选项？零是一个好的值吗？

像 tmux 这样的终端应用程序将按键作为字节流接收，特殊按键用 ASCII ESC 字符（`\033`）标记。问题 - 以及 escape-time 的原因 - 是除了标记特殊按键外，相同的 ASCII ESC 也用于 Escape 键本身。

如果 tmux 得到一个 `\033` 字节，然后在很短的时间后得到一个 x，用户是按了 Escape 然后是 x，还是按了 M-x？没有保证的方法知道。

tmux 和大多数其他终端应用程序用来解决这个问题的解决方案是引入延迟。当 tmux 接收到 `\033` 时，它启动一个计时器 - 如果计时器在没有任何后续字节的情况下过期，则该键是 Escape。缺点是 Escape 键按下识别之前有延迟。

如果 tmux 与终端在同台计算机上运行，或通过快速网络运行，则通常表示按键的字节会一起到达，因此 `escape-time` 为零可能没问题。在较慢的网络上，较大的值会更好。

### 如何使修改后的功能键和箭头键（如 C-Up、M-PageUp）在 tmux 内部工作？

tmux 使用 *xterm(1)* 风格的转义序列发送修改后的功能键。这可以通过 `cat` 验证，例如按 M-Left：

~~~~
$ cat
^[[1;3D
~~~~

如果这不同，则 tmux 外部的 `TERM` 可能不正确，tmux 无法识别来自外部终端的按键。

如果正确，则 tmux 内部的某些应用程序在 `TERM` 设置为 `screen` 或 `screen-256color` 时无法识别这些按键，因为这些终端描述缺少功能。`tmux` 和 `tmux-256color` 描述确实有这些功能，所以使用这些可能有效。在 `.tmux.conf` 中：

~~~~
set -g default-terminal tmux-256color
~~~~

### `#(command)` 转义字符的正确方法是什么？

使用 `#(command)` 结构在状态行中包含命令输出时，命令将被解析两次。第一次，当它被配置文件或命令提示符解析器读取时，第二次当状态行被绘制且命令传递给 shell 时。例如，要将字符串 "(test)" 回显到状态行，可以使用单引号或双引号：

~~~~
set -g status-right "#(echo \\(test\\))"
set -g status-right '#(echo \\(test\\))'
~~~~

在这两种情况下，status-right 选项将设置为字符串 `#(echo
\\(test\\))`，执行的命令将是 `echo \(test\)`。

### tmux 使用太多 CPU。我该怎么办？

自动窗口重命名可能使用大量 CPU，特别是在慢速计算机上：如果这是问题，请使用 `setw -g automatic-rename off` 关闭它。如果这不能解决它，请报告问题。

### 显示负载平均值的最佳方法是什么？为什么没有 `#L`？

在代码中可移植地获取负载平均值是不可能的，最好不添加可移植性代码。以下内容在至少 Linux、*BSD 和 OS X 上工作：

~~~~
uptime|awk '{split(substr($0, index($0, "load")), a, ":"); print a[2]}'
~~~~

### 如何将同一会话附加到多个客户端但具有不同的当前窗口，如 `screen -x`？

可以使用 `link-window` 手动将一个或多个窗口链接到多个会话，或者使用 `new-session -t` 创建包含所有窗口的分组会话。

### 我看不到斜体！或者斜体和反显是相反的！

GNU screen 不支持斜体，`screen` 终端描述错误地使用斜体转义序列。

从 tmux 2.1 开始，如果 `default-terminal` 设置为 `screen` 或匹配 `screen-*`，tmux 将像 screen 一样行为，斜体将被禁用。

要启用斜体，请确保您使用的是 tmux 终端描述：

~~~~
set -g default-terminal "tmux"
~~~~

### 如何查看默认配置？

通过启动没有配置文件的新 tmux 服务器来显示默认会话选项：

~~~~
$ tmux -Lfoo -f/dev/null start\; show -g
~~~~

或默认窗口选项：

~~~~
$ tmux -Lfoo -f/dev/null start\; show -gw
~~~~

### 如何将 tmux 中的选择复制到系统的剪贴板？

参见 [此页面](https://github.com/tmux/tmux/wiki/Clipboard)。

### 为什么当我附加到会话时看到会话周围有点？

直到版本 2.9，tmux 将窗口大小限制为最小附加客户端。如果不这样做，将无法看到整个窗口。点标记 tmux 可以显示的窗口大小。

为避免这种情况，请在附加时分离所有其他客户端：

~~~~
$ tmux attach -d
~~~~

或在 tmux 内部通过使用 C-b D 分离各个客户端或全部使用：

~~~~
C-b : attach -d
~~~~

从 2.9 或更高版本开始，将 `window-size` 选项设置为 `largest` 将使用最大的附加客户端而不是最小的。

### 如何将 *ssh-agent(1)* 与 tmux 一起使用？

*ssh-agent(1)* 设置一个环境变量（`SSH_AUTH_SOCK`），它需要在每个 shell 进程中存在。

可以在运行 tmux 之前确保设置了 `SSH_AUTH_SOCK`，然后它将在全局环境中，并将为在 tmux 中创建的每个窗格设置。`update-environment` 选项默认包含 `SSH_AUTH_SOCK`，因此当附加会话或创建新会话时，它将更新会话环境中的 `SSH_AUTH_SOCK`。但是，如果附加会话时未设置 `SSH_AUTH_SOCK`，`update-environment` 将导致 `SSH_AUTH_SOCK` 从环境中*移除*，并且不会为新窗格设置。参见
[此处](https://man.openbsd.org/OpenBSD-current/man1/tmux.1#GLOBAL_AND_SESSION_ENVIRONMENT)
和
[此处](https://man.openbsd.org/OpenBSD-current/man1/tmux.1#update-environment__)。

实际上，在 shell 配置文件中设置 *ssh-agent(1)* 更可靠，该配置文件为每个 shell 运行，无论 tmux 如何。对于像 *ksh(1)* 或 *bash(1)* 这样的 Bourne 风格 shell，在 `.profile`、`.kshrc`、`.bash_profile` 或 `.bashrc` 中类似这样的内容：

~~~~
[ ! -f ~/.ssh.agent ] && ssh-agent -s >~/.ssh.agent
eval `cat ~/.ssh.agent` >/dev/null
if ! kill -0 $SSH_AGENT_PID 2>/dev/null; then
        ssh-agent -s >~/.ssh.agent
        eval `cat ~/.ssh.agent` >/dev/null
fi
~~~~

### 什么是 passthrough 转义序列以及如何使用它？

tmux 小心不要发送终端不会理解的转义序列，因为它无法预测它会如何反应。

但是，可以通过将转义序列包装在 DCS 序列的特殊形式中并以 `tmux;` 为前缀来强制传递它。包装序列中的任何 `\033` 字符都必须加倍，例如：

~~~~
\033Ptmux;\033\033]1337;SetProfile=NewProfileName\007\033\\
~~~~

将传递此 iTerm2 特殊转义序列
`\033]1337;SetProfile=NewProfileName\007` 到终端而不被 tmux 丢弃。

此功能应谨慎使用。请注意，由于 tmux 不知道 passthrough 转义序列对终端状态所做的任何更改，它可能会撤销它们。

passthrough 转义序列对于更改光标颜色或样式不再是必需的，因为 tmux 现在有自己的支持（参见 `Cs`、`Cr`、`Ss` 和 `Se` 功能）。

从 tmux 3.3 开始，`allow-passthrough` 选项必须设置为 `on` 或 `all` 才能使用 passthrough 序列。

### 如何使 .tmux.conf 在 tmux 版本之间可移植？

对于 `set-option`，`-q` 标志会抑制关于未知选项的警告，例如：

~~~~
set -gq mode-bg red
~~~~

从 tmux 2.3 开始，运行服务器版本在 `version` 格式变量中可用。这可以与 `if-shell`、`if-shell -F`（从 tmux 2.0 开始）或 `%if`（从 tmux 2.4 开始）一起使用来检查特定服务器版本。

`m`（从 tmux 2.6 开始）和 `m/r`（从 tmux 3.0 开始）修饰符对此最有用。例如，如果运行开发构建显示绿色状态行，如果版本 3.1 或更高显示蓝色，否则显示红色：

~~~~
%if #{m/r:^next-,#{version}}
set -g status-style bg=green
%elif #{&&:#{m/r:^[0-9]+\.[0-9]+$,#{version}},#{e|>=|f:#{version},3.1}}
set -g status-style bg=blue
%else
set -g status-style bg=red
%endif
~~~~

对于早于 tmux 2.3 的版本，必须使用 `if-shell` 和 `tmux -V`。

请注意，在 OpenBSD 版本号中，tmux 版本号跟踪 OpenBSD 版本，参见 [此 FAQ
条目](https://github.com/tmux/tmux/wiki/FAQ#how-often-is-tmux-released-what-is-the-version-number-scheme)
了解 tmux 版本号信息。

### 为什么 XMODEM、YMODEM 和 ZMODEM 在 tmux 内部不起作用？

tmux 不是文件传输程序，支持这些协议比其剩余流行度值得的努力更多。在尝试使用它们之前分离 tmux。