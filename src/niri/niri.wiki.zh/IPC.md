您可以通过IPC套接字与运行中的niri实例通信。
检查`niri msg --help`以获取可用命令。

`--json`标志以JSON格式打印响应，而不是格式化格式。
例如，`niri msg --json outputs`。

> \[!TIP]
> 如果您在升级niri后从`niri msg`收到解析错误，请确保您已重新启动niri本身。
> 您可能正在尝试将较新版本的`niri msg`与较旧版本的`niri`合成器一起运行。

### 事件流

<sup>自：0.1.9</sup>

虽然大多数niri IPC请求返回单个响应，但事件流请求将使niri在IPC连接关闭之前持续将事件流传输到IPC连接。
这对于实现各种在发生某些事件时立即更新的栏和指示器很有用，而无需连续轮询。

事件流IPC设计为首先提供完整的当前状态，然后对状态进行更新。
这样，您的状态永远不会与niri"不同步"，您无需进行任何其他IPC信息请求。

在合理的情况下，事件流状态更新是原子的，但情况并非总是如此。
例如，窗口可能最终具有已删除工作区的工作区ID。
如果相应的workspace-changed事件在相应的window-changed事件之前到达，可能会发生这种情况。

要了解事件，运行`niri msg event-stream`。
不过，这更多是一个调试功能。
您可以从`niri msg --json event-stream`获取原始事件，或通过连接到niri套接字并手动请求事件流。

您可以在[这里](https://yalter.github.io/niri/niri_ipc/enum.Event.html)找到完整事件列表及其文档。

### 程序化访问

`niri msg --json`是对套接字进行读写操作的薄包装器。
在实现更复杂的脚本和模块时，建议您直接访问套接字。

连接到位于文件系统中`$NIRI_SOCKET`的UNIX域套接字。
在单行上写入JSON编码的请求，后跟换行符，或通过刷新并关闭连接的写入端。
以JSON格式读取回复，也是在单行上。

您可以使用`socat`直接测试与niri的通信：

```sh
$ socat STDIO "$NIRI_SOCKET"
"FocusedWindow"
{"Ok":{"FocusedWindow":{"id":12,"title":"t socat STDIO /run/u ~","app_id":"Alacritty","workspace_id":6,"is_focused":true}}}
```

回复是`Ok`或`Err`，包装与`niri msg --json`相同的JSON对象。

对于更复杂的请求，您可以使用`socat`找到`niri msg`如何格式化它们：

```sh
$ socat STDIO UNIX-LISTEN:temp.sock
# 然后，在另一个终端中：
$ env NIRI_SOCKET=./temp.sock niri msg action focus-workspace 2
# 然后，查看socat终端：
{"Action":{"FocusWorkspace":{"reference":{"Index":2}}}}
```

您可以在[niri-ipc子包文档](https://yalter.github.io/niri/niri_ipc/)中找到所有可用请求和响应类型。

### 向后兼容性

JSON输出*应该*保持稳定，如：

*   现有字段和枚举变量不应重命名
*   非可选的现有字段不应删除

但是，将添加新字段和枚举变量，因此您应该在合理的情况下优雅地处理未知字段或变量。

格式化/人类可读输出（即没有`--json`标志）**不**被视为稳定。
请在脚本中选择JSON输出，因为我保留对人类可读输出进行任何更改的权利。

`niri-ipc`子包（像其他niri子包一样）在Rust semver方面*不是*API稳定的；相反，它遵循niri本身的版本。
特别是，将添加新的结构字段和枚举变量。