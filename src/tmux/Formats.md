## 使用格式

格式是一种强大的方式来获取正在运行的 tmux 服务器的信息并
控制各种命令的输出。

它们被广泛使用，例如：

- 显示信息 (`display-message`)。

- 状态行、终端标题上的文本格式
  (`set-titles-string`)，以及自动重命名 (`automatic-rename-format`)。

- 列出命令的输出 (`-F` 标志用于 `list-clients`、`list-commands`、
  `list-sessions`、`list-windows`、`list-panes`、`list-buffers`)。

- 新窗口和会话打印的目标和传递的参数 (`-F` 和
  `-c` 标志用于 `new-session`、`break-pane`、`new-window`、`split-window`)。

- 选择模式中显示的文本和过滤器 (`-F` 和 `-f` 用于 `choose-client`、
  `choose-tree`、`choose-buffer`，以及 `f` 键)。

- 设置选项值 (`-F` 标志用于 `set-option`)。

- 运行命令 (`-F` 标志用于 `if-shell` 和 `run-shell`)。

- 菜单内容 (`display-menu`)。

- 配置文件中的解析时条件 (`%if`)。

本文档描述了它们的语法并附有示例。

格式也在手册中有所记录
[这里](https://man.openbsd.org/tmux.1#FORMATS)，以及所有
变量的列表。

请注意，这些功能中的一些仅在 tmux 3.1 及更高版本中可用（最
显著的是：填充和多个 `s` 修饰符）。

### 基本用法

格式是一个包含在 `#{}` 中的特殊指令的字符串，tmux 将展开这些指令（注意这与用于
嵌入样式的 `#[]` 不同）。每个 `#{}` 可以引用命名变量，其中包含一些
关于服务器、会话、客户端或类似的信息。并非所有变量都
始终存在 - 未知或缺失的变量将被替换为空。

最简单的用法是显示一些信息，例如使用 `display-message` 获取 tmux
服务器 PID（`-p` 标志打印结果而不是
在状态行中显示）：

~~~~
$ tmux display -p '#{pid}'
98764
~~~~

或者修改 `list-windows` 的输出：

~~~~
$ tmux lsw -F '#{window_id} #{window_name}'
@0 irssi
@1 mutt
@3 emacs
@4 ksh
~~~~

`display-message` 是处理格式的有用命令。`-a` 标志
列出它知道的格式：

~~~~
$ tmux display -a|fgrep width=
client_width=143
pane_width=143
window_width=143
~~~~

`-v` 打印详细信息，说明格式如何评估以帮助发现
错误：

~~~~
$ tmux display -vp '#{pid}'
# expanding format: #{pid}
# found #{}: pid
# format 'pid' found: 98764
# replaced 'pid' with '98764'
# result is: 98764
98764
~~~~

因为 `#` 在格式中是特殊的，所以需要双写 (`##`) 来包含
`#`：

~~~~
$ tmux display -p '##{pid}'
#{pid}
~~~~

许多格式都有单字符别名，例如 `#S` 对应 `#{session_name}`，
但这些不能与修饰符一起使用，也不鼓励使用。

### 选项和环境变量

除了格式变量，格式中的 `#{}` 还可以包含 tmux
选项的名称，例如用户选项：

~~~~
$ tmux set @foo 'hello'
$ tmux display -p 'hello #{@foo}'
hello hello
~~~~

或者全局环境中的环境变量名称：

~~~~
$ tmux showenv -g USER
USER=nicholas
$ tmux display -p '#{USER}'
nicholas
~~~~

下面的大多数示例使用用户选项（例如 `@v`），但任何格式
变量、选项或环境变量都可以替代使用。

### 简单修饰符

扩展变量的结果可以通过修饰符更改。

有几种不同的修饰符形式，但大多数由一个操作符
组成，带有零个或多个由标点字符分隔的参数（`|` 或 `/`
最常用），后跟一个冒号和一个或多个附加参数，这些参数是
应用修饰符的变量或格式。

最简单的修饰符不带参数，只修改单个变量。`t` 修饰符
将时间变量显示为人类可读的字符串：

~~~~
$ tmux lsw -F '#{t:window_activity}'
Fri Nov 29 13:52:35 2019
Fri Nov 29 13:37:53 2019
Fri Nov 29 12:06:46 2019
Thu Nov 28 15:51:20 2019
Thu Nov 28 07:41:05 2019
~~~~

`b` 和 `d` 从路径中修剪文件名和目录名：

~~~~
$ tmux set @p `pwd`
$ tmux display -p '#{d:@p}'
/usr/src/usr.bin
$ tmux display -p '#{b:@p}'
tmux
~~~~

### 修剪和填充

格式变量可以被修剪或填充。`=` 修饰符修剪，`p`
修饰符填充。它们都至少带有一个参数，即宽度 - 正
宽度表示在左侧修剪或在右侧填充，负
宽度表示相反：

~~~~
$ tmux set @v "foobar"
$ tmux display -p '#{=3:@v}'
foo
$ tmux display -p '#{=-3:@v}'
bar
$ tmux display -p '#{p9:@v}baz'
foobar   baz
$ tmux display -p '#{p-9:@v}baz'
   foobarbaz
~~~~

可以通过用 `;` 分隔来一起应用多个修饰符，
例如：

~~~~
$ tmux set @v "foobar"
$ tmux display -p '#{=3;p-6:@v}'
   foo
~~~~

`=` 修剪修饰符接受第二个参数，这是一个字符串，用于追加或
预置到结果中以显示它已被修剪。当给出多个
参数给修饰符时，它们必须由标点字符分隔，
包括操作符后面的一个。`/` 或 `|` 是最常见的分隔符：

~~~~
$ tmux set @v "foobar"
$ tmux display -p '#{=|6|...:@v}'  # 没有修剪
foobar
$ tmux display -p '#{=|5|...:@v}'  # 在右侧修剪
fooba...
$ tmux display -p '#{=|-5|...:@v}' # 在左侧修剪
...oobar
~~~~

### 比较

出于比较目的，tmux 认为格式的结果是
真或假。如果结果不为空且不是单个零
（`0`），则结果为真，否则为假。

有几种比较操作符可用。与 `t` 和 `b` 不同，这些操作符
需要两个变量进行比较，用逗号分隔。
这两个变量本身可以是格式而不是变量。

`==`、`!=`、`<`、`>`、`<=` 和 `>=` 是字符串比较：

~~~~
$ tmux set @v foo
$ tmux display -p '#{==:#{@v}bar,foobar}'
1
$ tmux display -p '#{!=:#{@v}bar,foobar}'
0
$ tmux display -p '#{<:#{@v},bar}'
0
~~~~

`||` 如果其任一参数为真则为真，`&&` 如果两者都为真则为真：

~~~~
$ tmux set @v foo
$ tmux display -p '#{||:0,#{@v}}'
1
$ tmux display -p '#{&&:0,#{@v}}'
0
~~~~

三元选择操作符 `?` 也可用。这与
其他比较修饰符略有不同（它实现得更早）并且在操作符和要检查的条件之间没有
冒号。条件
后跟两个结果格式，如果条件为真
则选择第一个，如果为假则选择第二个。任一结果可以为空。例如：

~~~~
$ tmux set @v 0
$ tmux display -p '#{?@v,yes,no}'
no
$ tmux display -p '#{?#{==:#{@v},0},yes,no}'
yes
$ tmux display -p '#{?#{==:#{@v},0},#{@v} is true,#{@v} is false}'
0 is true
~~~~

在选择操作符的结果内部，可以通过转义
为 `#,` 来插入逗号。

### 替换

格式通过 `s` 修饰符支持替换。这类似于
*sed(1)* 替换，接受两个或三个参数 - 要搜索的正则表达式，
要替换它的字符串和一组标志。要搜索的正则表达式和
要替换的字符串本身都可以是格式。
括号中的模式在替换中按数字展开
（`\1`、`\2` 等）。

与 `t`、`b` 和 `d` 类似，被替换的变量不能是格式，而
必须是变量。示例如下：

~~~~
$ tmux set @v foobar
$ tmux display -p '#{s|foo|bar|:@v}'
barbar
$ tmux display -p '#{s|(foo)(bar)|\2\1|:@v}'
barfoo
$ tmux set @w foo
$ tmux display -p '#{s|#{@w}|xxx|:@v}'
xxxbar
~~~~

第三个参数支持一个标志 `i`，表示正则表达式
不区分大小写：

~~~~
$ tmux set @v foobar
$ tmux display -p '#{s|FOO|xxx|:@v}'
foobar
$ tmux display -p '#{s|FOO|xxx|i:@v}'
xxxbar
~~~~

可以通过用 `;` 分隔来连续进行多次替换，
如下所示：

~~~~
$ tmux set @v foobar
$ tmux display -p '#{s|foo|xxx|;s|bar|yyy|:@v}'
xxxyyy
~~~~

### 表达式

在 tmux 3.2 及更高版本中，某些数学运算可用作
格式修饰符。这些通过 `e` 修饰符给出。第一个参数是
以下之一：

参数|操作
---|---
`+`|加法
`-`|减法
`*`|乘法
`/`|除法
`m`|取模

第二个参数可以是标志 `f` 以使用浮点数，否则
使用整数。第三个参数是结果中显示的小数位数
- 整数默认为零，浮点数默认为两位。

例如：

~~~~
$ tmux display -p '#{e|+|:1,1}'
2
$ tmux display -p '#{e|/|f|4:10,3}'
3.3333
~~~~

### 匹配和搜索

匹配修饰符 `m` 类似于比较修饰符并比较
两个格式 - 第一个是匹配第二个的模式。默认情况下
这期望一个 *fnmatch(3)* 模式，但 `r` 标志指定一个正则
表达式。`i` 标志表示不区分大小写。

~~~~
$ tmux set @v foobar
$ tmux display -p '#{m:*foo*,#{@v}}'
1
$ tmux display -p '#{m|ri:^FOO,#{@v}}'
1
~~~~

`C` 修饰符在窗格内容中搜索给定的格式并返回
行号，如果未找到则返回零。

~~~~
$ # xyz
$ tmux display -p '#{C:x*z}'
4
$ tmux display -p '#{C|r:x.z}'
2
~~~~

### 循环

`S`、`W` 和 `P` 修饰符遍历每个会话、当前会话中的每个窗口和当前窗口中的每个窗格，并为每个
扩展给定
格式。`W` 或 `P` 可以给出第二个格式，用于
当前窗口和活动窗格。例如：

~~~~
$ tmux display -p '#{W:#{window_name} }'
irssi mutt emacs ksh ksh ksh ksh ksh ksh emacs emacs ksh ksh ksh ksh ksh
$ tmux display -p '#{W:#{window_name} ,--- }'
irssi mutt emacs ksh ksh ksh ksh ksh --- emacs emacs ksh ksh ksh ksh ksh
~~~~

### 字面量和引用

`l` 修饰符产生给定的字面量字符串，这可以用于
作为 `?` 选择修饰符的第一个参数，例如：

~~~~
$ tmux display -p '#{?#{l:1},a,b}'
a
~~~~

`q` 修饰符引用任何特殊字符，通常用于如果结果是
作为另一个命令的参数传递。

~~~~
$ tmux set @v '()'
$ tmux display -p '#{q:@v}'
\(\)
~~~~

### 多次扩展

tmux 有两种修饰符可以将其结果扩展两次：`E` 和 `T`。
`#{E:status-left}` 将扩展 `status-left` 选项的内容。`T` 是
相同的，但也会扩展 *strftime(3)* 转换说明符（如 `%H` 和
`%M`）。

~~~~
$ tmux display -p '#{status-left}'
[#{session_name}]
$ tmux display -p '#{T:status-left}'
[0]
~~~~

### 组合使用格式

通常格式是嵌套的，并以比
此处示例更复杂的方式组合使用。例如，这里是默认 `status-format[0]` 的一部分：

~~~~
#{?#{&&:#{||:#{window_activity_flag},#{window_silence_flag}},#{!=:#{window-status-activity-style},default}}, #{window-status-activity-style},}
~~~~

如果
`window_activity_flag` 或 `window_silence_flag` 为真且
`window-status-activity-style` 选项不是 `default`，则扩展为 `window-status-activity-style` 选项的内容。

### 选择模式和格式

格式在三种选择模式中用于两个目的：用于
每行的格式和用于过滤器。

每行的格式通过 `-F` 指定给 `choose-buffer`、
`choose-tree` 或 `choose-client`。默认格式本身在格式中可用：

~~~~
$ tmux display -p '#{client_mode_format}'
session #{session_name} (#{client_width}x#{client_height}, #{t:client_activity})
$ tmux lsc -F '#{E:client_mode_format}'
session 0 (143x44, Mon Dec  2 12:51:29 2019)
session 0 (81x24, Mon Dec  2 12:51:26 2019)
~~~~

`choose-tree` 是最复杂的，因为它可以用于包含会话、窗口或窗格的行。通过
使用 `pane_format`、`window_format` 和 `session_format` 变量，相同的格式可以完成这三项工作。第一个
对所有三种类型的行都为真；第二个仅对窗格和
窗口为真；第三个仅对会话为真。

所有三种模式都支持使用 `-f` 标志或按 `f` 键进行过滤。
过滤器是一个为每行扩展的格式，如果为真，则
行包含在列表中，如果为假则不包含。例如，仅显示
名为 `mysession` 的会话：

~~~~
$ tmux choose-tree -sf '#{==:#{session_name},mysession}'
~~~~

或者仅显示包含文本 `foo` 的窗格的窗口：

~~~~
$ tmux choose-tree -wf '#{C:foo}'
~~~~

`find-window` 命令通过自动生成这样的过滤器来工作。

### 修饰符摘要

下表显示了修饰符的语法。使用以下字段：

- `variable` 表示仅变量名，例如 `session_name` 或 `@foo`。这
  不展开，不能包含 `#{}`。

- `format` 表示完全展开为格式的字符串，例如 `abc` 或
  `abc#{@foo}`。变量必须包含在 `#{}` 中，否则不展开：
  `session_name` 是字面量字符串 `session_name`，必须是
  `#{session_name}` 才能展开。

- `single` 表示单个格式或变量，但不是字符串，因此 `#{session_name}`
  或 `session_name` 都会展开，但 `abc#{session_name}` 不会展开。

- `string` 表示不展开的字符串，例如 `abc`。

- `flags` 是可选的单字符标志集。

格式|描述
---|---
`#{t:variable}`|时间转换为人类可读字符串
`#{b:variable}`|路径的文件名
`#{d:variable}`|路径的目录名
`#{=N:variable}`|修剪到宽度 N
`#{=/N/format:variable}`|修剪到宽度 N 并在修剪时添加标记
`#{pN:variable}`|填充到宽度 N
`#{==:format,format}`|比较两个格式（也包括 `!=` `<` `>` `<=` `>=`）
`#{?single,format,format}`|从两个格式中选择
`#{s/format/format/flags:variable}`|用字符串替换模式，标志 `i`
`#{m/flags:format,format}`|匹配模式与格式，标志 `r` 和 `i`
`#{C/flags:format}`|搜索格式，标志 `r` 和 `i`
`#{P:format,format}`|遍历每个窗格（也包括 `S`、`W`）
`#{l:string}`|字面量字符串
`#{E:variable}`|扩展变量内容
`#{T:variable}`|扩展变量内容并包含时间转换说明符
`#{q:variable}`|引用特殊字符