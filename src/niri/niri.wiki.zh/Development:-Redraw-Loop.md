在TTY上，一次只能向输出提交一帧，合成器必须等待输出重绘（由VBlank指示）才能提交下一帧。
在niri中，我们通过`RedrawState`枚举跟踪此状态，您可以在`OutputState`中找到它。

以下是`RedrawState`状态机的状态转换图：

<picture>
    <source media="(prefers-color-scheme: dark)" srcset="./img/RedrawState-dark.drawio.png">
    <img alt="RedrawState状态转换图" src="./img/RedrawState-light.drawio.png">
</picture>

`Idle`是默认状态，当输出不需要重绘时。
可能导致屏幕更新的任何操作都会调用`queue_redraw()`，这会将输出移动到`Queued`状态。
然后，在事件循环调度结束时，niri为每个`Queued`输出调用`redraw()`。

如果重绘造成损坏（即输出上的某些内容已更改），我们会进入`WaitingForVBlank`状态，因为我们在收到VBlank事件之前无法重绘。
但是，如果没有损坏，我们不会立即返回到`Idle`。
相反，我们设置一个定时器，在下一个VBlank大约发生时触发，并转换到`WaitingForEstimatedVBlank`状态。

这是必要的，以限制发送到应用程序的帧回调最多每个输出刷新周期一次。
没有此限制，应用程序可以开始持续重绘而无损坏（例如，如果应用程序窗口部分在屏幕外，只有屏幕外部分在变化），并在过程中消耗大量CPU。

然后，要么估计的VBlank定时器完成，我们返回到`Idle`，或者我们再次调用`queue_redraw()`并尝试再次重绘。