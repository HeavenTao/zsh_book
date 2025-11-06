### 概述

构建niri时，请检查`Cargo.toml`以获取构建特性的列表。
例如，您可以使用`cargo build --release --no-default-features --features dinit,dbus,xdp-gnome-screencast`将systemd集成替换为dinit集成。
然而，默认设置应该适用于大多数发行版。

> \[!WARNING]
> 不要使用`--all-features`构建！
>
> 一些特性仅用于开发用途。
> 例如，其中一个特性启用了将分析数据收集到内存缓冲区中，该缓冲区将无限增长直到您耗尽内存。

`niri-visual-tests`子包/二进制文件仅用于开发，不应打包。

打包niri的推荐方式是使其作为独立桌面会话运行。
为此，请根据此表将文件放入正确的目录。

| 文件 | 目标位置 |
| ---- | -------- |
| `target/release/niri` | `/usr/bin/` |
| `resources/niri-session` | `/usr/bin/` |
| `resources/niri.desktop` | `/usr/share/wayland-sessions/` |
| `resources/niri-portals.conf` | `/usr/share/xdg-desktop-portal/` |
| `resources/niri.service` (systemd) | `/usr/lib/systemd/user/` |
| `resources/niri-shutdown.target` (systemd) | `/usr/lib/systemd/user/` |
| `resources/dinit/niri` (dinit) | `/usr/lib/dinit.d/user/` |
| `resources/dinit/niri-shutdown` (dinit) | `/usr/lib/dinit.d/user/` |

这样做将使niri出现在GDM和其他显示管理器中。

有关发行版集成的更多信息，请参见[Integrating niri](./Integrating-niri)页面。

### 运行测试

我们的大部分测试会生成niri合成器实例并测试Wayland客户端。
这不需要图形会话，但由于测试并行性，在高核心数系统上可能会遇到文件描述符限制。

如果您遇到此问题，您可能不仅需要限制Rust测试工具线程数，还需要限制Rayon线程数，因为一些niri测试使用内部Rayon线程：

    $ export RAYON_NUM_THREADS=2
    ...继续运行cargo test，可能带有--test-threads=2

不要忘记在运行测试时排除仅用于开发的`niri-visual-tests`包。

一些测试需要在测试时提供无表面EGL。
如果这有问题，您可以像这样跳过它们：

    $ cargo test -- --skip=::egl

您可能还想设置`RUN_SLOW_TESTS=1`环境变量以运行较慢的测试。

### 版本字符串

niri版本字符串包括其版本和提交哈希：

    $ niri --version
    niri 25.01 (e35c630)

在打包系统中构建时，通常没有仓库，因此提交哈希不可用，版本将显示"unknown commit"。
在这种情况下，请手动设置提交哈希：

    $ export NIRI_BUILD_COMMIT="e35c630"
    ...继续构建niri

您也可以完全覆盖版本字符串，在这种情况下，请确保相应的niri版本保持不变：

    $ export NIRI_BUILD_VERSION_STRING="25.01-1 (e35c630)"
    ...继续构建niri

请记住为`cargo build`和`cargo install`都设置此变量，因为后者在环境更改时会重新构建niri。

### 崩溃

良好的崩溃回溯对于诊断niri崩溃是必需的。
请使用`niri panic`命令测试您的包是否产生良好的回溯。

    $ niri panic
    thread 'main' panicked at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/core/src/time.rs:1142:31:
    overflow when subtracting durations
    stack backtrace:
       0: rust_begin_unwind
                 at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/std/src/panicking.rs:665:5
       1: core::panicking::panic_fmt
                 at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/core/src/panicking.rs:74:14
       2: core::panicking::panic_display
                 at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/core/src/panicking.rs:264:5
       3: core::option::expect_failed
                 at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/core/src/option.rs:2021:5
       4: expect<core::time::Duration>
                 at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/core/src/option.rs:933:21
       5: sub
                 at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/core/src/time.rs:1142:31
       6: cause_panic
                 at /builddir/build/BUILD/niri-0.0.git.1699.279c8b6a-build/niri/src/utils/mod.rs:382:13
       7: main
                 at /builddir/build/BUILD/niri-0.0.git.1699.279c8b6a-build/niri/src/main.rs:107:27
       8: call_once<fn() -> core::result::Result<(), alloc::boxed::Box<dyn core::error::Error, alloc::alloc::Global>>, ()>
                 at /builddir/build/BUILD/rust-1.83.0-build/rustc-1.83.0-src/library/core/src/ops/function.rs:250:5
    note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.

要查找的重要事项：

*   崩溃消息存在："overflow when subtracting durations"。
*   回溯一直追溯到`main`并包括`cause_panic`。
*   回溯包括`cause_panic`的文件和行号：`at /.../src/utils/mod.rs:382:13`。

如果可能，请确保您的niri包本身具有良好的崩溃回溯，即*不*安装debuginfo或其他包。
当用户的合成器首次崩溃时，他们可能没有安装debuginfo，我们真的希望能够立即诊断和修复所有崩溃。

### Rust依赖

每个niri版本都附带来自`cargo vendor`的打包依赖档案。
您可以使用它完全离线构建相应的niri版本。

如果您不想使用打包依赖，请考虑遵循niri版本的`Cargo.lock`。
它包含我在测试版本时使用的精确依赖版本。

如果您需要更改某些依赖的版本，请特别注意`smithay`和`smithay-drm-extras`提交哈希。
这些包目前没有定期稳定版本，因此niri使用git快照。
上游经常有破坏性更改（API和行为），因此强烈建议您使用niri版本的`Cargo.lock`中的确切提交哈希。

### Shell补全

您可以通过`niri completions <SHELL>`为几个shell生成补全，即`niri completions bash`。
请参见`niri completions -h`获取完整列表。