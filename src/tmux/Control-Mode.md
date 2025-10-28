## 控制模式

控制模式是一种特殊模式，允许 tmux 客户端使用简单的纯文本协议与 tmux 通信。它由 George Nachman 设计和编写，允许他的 [iTerm2](https://www.iterm2.com/) 终端与 tmux 接口并使用 iTerm2 UI 显示 tmux 窗格。

控制模式客户端就像普通 tmux 客户端一样，只是不绘制终端，tmux 使用文本进行通信。因为控制模式是纯文本的，所以可以轻松地通过 *ssh(1)* 解析和使用。

控制模式客户端接受标准 tmux 命令并返回其输出，并且
另外发送仅控制模式的信息（主要是异步
通知），以 `%` 为前缀。其想法是控制模式的用户使用 tmux
命令（`new-window`、`list-sessions`、`show-options` 等）来控制
tmux，而不是为控制模式复制单独的命令集。

### 进入控制模式

tmux 的 `-C` 标志启动控制模式客户端。这只有在
附加客户端时才真正有用
使用 `attach-session` 或 `new-session` 命令。

`-C` 有两种形式。单个 `-C` 不会更改任何终端属性 -
保持终端处于规范模式，因此例如回显仍然启用。
这适用于测试：输入将出现，控制字符如
删除和终止仍然工作。

两个 `-C`（即 `-CC`）禁用规范模式和大多数其他终端功能
适用于应用程序（例如，不需要回显）。此外，`-CC` 形式发送一个 `\033P1000p` DSC 序列（类似于 ReGIS）
监听终端可以用来检测已进入控制模式并
发送 `%exit` 行和相应的 `ST`（`\033\`）序列当
客户端退出。

使用任一形式，输入空行（仅按 `Enter`）将分离
客户端。

例如，这显示了在名为 `test` 的套接字上启动新的 tmux 服务器的输出
带有新会话（运行 `new-session`）并附加客户端
在控制模式下，然后杀死服务器：

~~~~
$ tmux -Ltest -C new
%begin 1578920019 258 0
%end 1578920019 258 0
%window-add @1
%sessions-changed
%session-changed $1 1
%window-renamed @1 tmux
%output %1 nicholas@yelena:~$
%window-renamed @1 ksh
kill-server
%begin 1578920028 265 1
%end 1578920028 265 1
~~~~

在此示例中，`kill-server` 是用户输入的命令，其余以 `%` 开头的行由 tmux 服务器发送。

### 命令

tmux 命令或命令序列可以发送到控制模式客户端，例如
创建新窗口：

~~~~
new -n mywindow
%begin 1578920529 257 1
%end 1578920529 257 1
~~~~

每个命令产生一个输出块。这被两行保护行包围：
如果命令成功，则为 `%begin` 和 `%end`，或者
如果失败，则为 `%begin` 和 `%error`。

每个 `%begin`、`%end` 或 `%error` 都有三个参数：

1. 从纪元开始的秒数时间；

2. 唯一的命令编号；

3. 标志，目前这始终为一。

`%begin` 的时间和命令编号将始终与相应的
`%end` 或 `%error` 匹配，尽管 tmux 永远不会混合不同命令的输出
因此没有要求使用这些。

命令的输出按命令在内部使用时的方式发送
tmux 或从 shell 提示符，例如：

~~~~
lsp -a
%begin 1578922740 269 1
0:0.0: [80x24] [history 0/2000, 0 bytes] %0 (active)
1:0.0: [80x24] [history 0/2000, 0 bytes] %1 (active)
1:1.0: [80x24] [history 0/2000, 0 bytes] %2 (active)
%end 1578922740 269 1
~~~~

或者：

~~~~
abcdef
%begin 1578923149 270 1
parse error: unknown command: abcdef
%error 1578923149 270 1
~~~~~

### 获取信息

大多数控制模式用户希望从 tmux 服务器获取信息。最有用的命令是 `list-sessions`、`list-windows`、
`list-panes` 和 `show-options`。

应尽可能使用 `-F` 标志以已知格式输出，而不是
依赖默认值。`q` 格式修饰符对转义很有用。

例如列出所有会话及其 ID 和名称：

~~~~
ls -F '#{session_id} "#{q:session_name}"'
%begin 1578925957 337 1
$4 "\"quoted\""
$2 "abc\ def"
$0 "bar"
$1 "foo"
$3 "😀"
%end 1578925957 337 1
~~~~

### 窗格输出

像普通 tmux 客户端一样，控制模式客户端可以附加到单个
会话（可以使用 `switch-client`、
`attach-session` 或 `kill-session` 等命令更改）。附加会话中任何窗口中的任何窗格的任何输出都会发送到控制客户端。这采用
`%output` 通知的形式，带有两个参数：

1. 窗格 ID（*不是*窗格编号）。

2. 输出。

输出中任何小于 ASCII 32 的字符和 `\` 字符都会被替换
为其八进制等效值，因此 `\` 变为 `\134`。否则，它就是
在窗格中运行的应用程序发送给 tmux 的内容。它可能不是有效的
UTF-8，并且可能包含 tmux 期望的转义序列（因此
对于 `TERM=screen` 或 `TERM=tmux`）。

例如，创建新窗口并发送 *ls(1)* 命令：

~~~~
neww
%begin 1578923903 256 1
%end 1578923903 256 1
%output %1 nicholas@yelena:~$
send 'ls /' Enter
%begin 1578923910 261 1
%end 1578923910 261 1
%output %1 ls /\015\015\012
%output %1 altroot/     bsd.booted   dev/         obsd*        sys@\015\012bin/         bsd.rd       etc/         reboot*      tmp/\015\012
%output %1 boot         bsd.sp       home/        root/        usr/\015\012bsd          cvs@         mnt/         sbin/        var/\015\012
%output %1 nicholas@yelena:~$
~~~~~

请注意，tmux 本身生成的输出（例如在复制或选择模式中）
不会发送到控制模式客户端。

### 通知

当进行更改时，通知会发送到控制模式客户端，无论是由
另一个客户端还是由 tmux 服务器本身。

支持以下通知：

通知|描述
---|---
`%pane-mode-changed %pane`|窗格的模式已更改。
`%window-pane-changed @window %pane`|窗口的活动窗格已更改。
`%window-close @window`|附加会话中的窗口已关闭。
`%unlinked-window-close @window`|另一个会话中的窗口已关闭。
`%window-add @window`|窗口已添加到附加会话。
`%unlinked-window-add @window`|窗口已添加到另一个会话。
`%window-renamed @window new-name`|附加会话中的窗口已重命名。
`%unlinked-window-renamed @window new-name`|另一个会话中的窗口已重命名。
`%session-changed $session session-name`|附加会话已更改。
`%client-session-changed client $session session-name`|另一个客户端的附加会话已更改。
`%session-renamed $session new-name`|会话已重命名。
`%sessions-changed`|会话已创建或销毁。
`%session-window-changed $session @window`|会话的当前窗口已更改。

`$session`、`@window` 和 `%pane` 是会话、窗口和窗格 ID。

### 特殊命令

tmux 为控制模式客户端提供了两个特殊参数到 `refresh-client` 命令，以执行正常客户端不需要的操作。这些是：

- `refresh-client -C` 设置控制模式客户端的大小。如果未
  使用，控制模式客户端不会影响其他客户端的大小，无论
  `window-size` 选项的值如何。如果使用，则它们被视为
  具有给定大小的任何其他客户端，并且可能设置窗口大小。

- `refresh-client -f`（`-F` 也接受向后兼容）设置
  控制模式客户端的标志。一些标志是通用的，但有几个
  仅适用于控制模式客户端：
  - `no-output` 不发送任何 `%output` 通知；
  - `wait-exit` 在 `%exit` 之后等待空行再实际退出；
  - `pause-after` 用于流量控制。

- `refresh-client -A` 用于流量控制，`-B` 用于格式
  订阅。

此外，`send-keys` 有一个 `-H` 标志，允许以十六进制形式输入 Unicode 键。

一些命令如 `suspend-client` 在与控制模式客户端一起使用时没有效果。

### 流量控制

tmux 为控制模式客户端提供了一种流量控制机制。流量
控制通过允许 tmux 在窗格落后太多时暂停输出到客户端来工作。一旦窗格暂停，客户端可以要求 tmux
在准备就绪时继续发送输出。由客户端更新
窗格内容（如果需要），例如使用 `capture-pane`。

通过使用 `refresh-client -f` 设置 `pause-after` 标志来启用流量控制。这需要一个参数，即
窗格应该暂停的秒数：

~~~~
refresh-client -f pause-after=30
~~~~

当窗格暂停时，将发送 `%pause` 通知，其中包含窗格 ID。
可以使用 `refresh-client -A` 继续窗格：

~~~~
refresh-client -A '%0:continue'
~~~~

继续后，将发送 `%continue` 通知。

启用流量控制后，不会发送 `%output` 通知；而是
使用 `%extended-output` 通知。这有额外的参数
以单个 `:` 参数结束。目前有两个参数：窗格 ID 和窗格落后的毫秒数。例如：

~~~~
%extended-output %0 1234 : abcdef
~~~~

`refresh-client -A` 也可用于手动暂停窗格（`-A '%0:pause'`）
或打开或关闭它。关闭窗格告诉 tmux 客户端不需要
发送窗格输出，并允许 tmux 选择停止读取
窗格（如果可能）。

### 格式订阅

控制模式客户端可以订阅格式，并在每次其扩展值更改时得到通知。使用 `refresh-client -B` 命令添加或删除订阅。这需要一个参数，该参数由三个部分组成，用冒号分隔：

1) 订阅名称。

2) 要订阅的项目类型，其中之一：

   类型|描述
   ----|---
   空|附加会话。
   `%n`|单个窗格 ID。
   `%*`|附加会话中的所有窗格。
   `@n`|单个窗口 ID。
   `@*`|附加会话中的所有窗口。

3) 格式。

如果省略类型和格式，仅给出订阅名称，则
删除订阅。

tmux 为给定类型的每个匹配项扩展一次格式，如果
结果值已更改，则发送 `%subscription-changed` 通知。
这最多每秒发生一次。

### 一般说明

其他一些说明：

- 强烈建议使用会话、窗口和窗格 ID 而不是名称或索引，因为它们是明确的。

- 用户选项可用于在 tmux 服务器中存储和检索自定义选项（它们可以设置为服务器、会话、窗口或窗格）。`show-options -v @foo` 仅显示用户选项 `@foo` 的选项值。

- 格式是检查会话、窗口或窗格属性或 tmux 服务器本身的主方法。`display-message -p` 命令对此很有用，以及列表命令的 `-F` 标志。