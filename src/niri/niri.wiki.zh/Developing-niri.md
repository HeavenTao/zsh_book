## 运行本地构建

开发期间测试niri的主要方法是以嵌套窗口的方式运行它。第二步通常是切换到不同的TTY并在那里运行niri。

一旦功能或修复基本完成，通常希望将本地构建作为主要合成器运行以进行适当测试。最简单的方法是正常安装niri（例如从发行版软件包），然后用`sudo cp ./target/release/niri /usr/bin/niri`覆盖二进制文件。不过请确保您知道在一切出错时如何恢复到工作版本。

如果您使用基于RPM的发行版，可以使用`cargo generate-rpm`为本地构建生成RPM包。

## 日志级别

Niri使用[`tracing`](https://lib.rs/crates/tracing)进行日志记录。日志级别使用方式如下：

*   `error!`：可恢复的编程错误和bug。通常会使用`unwrap()`的地方。然而，当Wayland合成器崩溃时，它会带来整个会话的崩溃，所以在合理的情况下最好恢复并记录`error!`。如果您在niri日志中看到`ERROR`，那总是表示*bug*。
*   `warn!`：发生了一些坏但仍然*可能*的事情。通知用户他们做了错误的事情，或他们的硬件做了奇怪的事情，属于此类别。例如，配置解析错误应该用`warn!`表示。
*   `info!`：与正常操作相关的重要消息。使用`RUST_LOG=niri=info`运行niri不应使用户想要完全禁用日志记录。
*   `debug!`：与正常操作相关的不太重要的消息。隐藏`debug!`消息运行niri不应对用户体验产生负面影响。
*   `trace!`：所有对调试有用但否则过于繁杂或性能密集的内容。`trace!`消息在发布构建中*被编译掉*。

## 测试

我们有一些单元测试，最突出的是布局代码和配置解析的测试。

在向布局添加新操作时，在`src/layout/mod.rs`底部的`Op`枚举中添加它们（这将自动将其包含在随机测试中），如果适用，添加到下面的`every_op`数组中。

在添加新配置选项时，将其包含在配置解析测试中。

### 运行测试

请确保运行`cargo test --all`以运行子包中的测试。

一些测试运行速度太慢而无法正常运行，比如布局代码的随机测试，因此通常跳过它们。设置`RUN_SLOW_TESTS`变量以运行它们：

    env RUN_SLOW_TESTS=1 cargo test --all

通常还会有助于运行更长时间的随机测试，以便它们可以探索更多输入。您可以使用环境变量控制这一点。这是我在推送前通常运行测试的方式：

    env RUN_SLOW_TESTS=1 PROPTEST_CASES=200000 PROPTEST_MAX_GLOBAL_REJECTS=200000 RUST_BACKTRACE=1 cargo test --release --all

### 视觉测试

`niri-visual-tests`子包是一个GTK应用程序，运行硬编码的测试用例，以便您可以直观地检查它们是否看起来正确。它使用带有真实布局和渲染代码的模拟窗口。在处理动画时特别有用。

## 性能分析

我们与[Tracy](https://github.com/wolfpld/tracy)分析器集成，您可以通过使用功能标志构建niri来启用它：

    cargo build --release --features=profile-with-tracy-ondemand

然后您可以打开Tracy（您需要最新的稳定版本）并连接到运行中的niri实例以收集性能分析数据。性能分析数据是"按需"收集的——即，仅在Tracy连接时。如果您愿意，可以将这样构建的niri作为您的主要合成器运行。

> \[!NOTE]
> 如果您需要分析niri启动或niri CLI，您可以选择"始终开启"的性能分析，使用此功能标志：
>
>     cargo build --release --features=profile-with-tracy
>
> 以这种方式编译时，niri将**始终**收集性能分析数据，因此您不能将这样的构建作为您的主要合成器运行。

要使niri函数在Tracy中显示，请这样进行检测：

```rust
pub fn some_function() {
    let _span = tracy_client::span!("some_function");

    // 函数代码。
}
```

您还可以使用`--features=profile-with-tracy-allocations`启用Rust内存分配分析。