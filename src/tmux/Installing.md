## 安装 tmux

### 二进制包

许多平台提供预构建的 tmux 包，尽管这些包通常已经过时。发现和安装这些包的命令详情可以在平台包管理工具的文档中找到，例如：

平台|安装命令
---|---
Arch Linux|`pacman -S tmux`
Debian 或 Ubuntu|`apt install tmux`
Fedora|`dnf install tmux`
RHEL 或 CentOS|`yum install tmux`
macOS (使用 Homebrew)|`brew install tmux`
macOS (使用 MacPorts)|`port install tmux`
openSUSE|`zypper install tmux`

一些第三方二进制包可用：[AppImage](Installing#appimage-package) 和
[RPMs](Installing#red-hat-enterprise-linux--centos-rpms)。

### 从源代码压缩包

tmux 需要两个可用的库：

1. [libevent](https://libevent.org/)

2. [ncurses](https://invisible-island.net/ncurses/ncurses.html)

此外，tmux 需要 C 编译器、make、yacc（或 bison）和 pkg-config。

在大多数平台上，这些都可以作为包获得。此表列出了运行或构建 tmux 所需的包：

平台|命令|运行包|构建包
---|---|---|---
Debian|`apt-get install`|`libevent ncurses`|`libevent-dev ncurses-dev build-essential bison pkg-config`
RHEL 或 CentOS|`yum install`|`libevent ncurses`|`libevent-devel ncurses-devel gcc make bison pkg-config`

如果 libevent 和 ncurses 不作为包可用，可以从源代码构建，参见 [此节](#building-dependencies)。

tmux 使用 autoconf，因此它提供了一个 configure 脚本。要使用 sudo 构建并安装到 `/usr/local`，运行：

~~~~
tar -zxf tmux-*.tar.gz
cd tmux-*/
./configure
make && sudo make install
~~~~

要安装到其他位置，请向 configure 添加 `--prefix`，例如对于 `/usr` 添加
`--prefix=/usr`。

### 构建依赖项

如果依赖项不可用，可以从源代码构建并本地安装。如果可以从系统包安装依赖项，则不推荐这样做。

构建需要安装 C 编译器、make、automake、autoconf 和 pkg-config。通常需要构建 libevent 而不是 ncurses。

完整的说明可以在项目网站上找到，但这是将 libevent 和 ncurses 安装到 `~/local` 供单个用户的摘要。要安装到系统范围的 `/opt` 或 `/usr/local` 目录下，
在每种情况下将所需路径替换为 `$HOME/local` 并以 root 身份运行 `make install`（例如使用 sudo：`make && sudo make install`）。

对于 libevent：

~~~~
tar -zxf libevent-*.tar.gz
cd libevent-*/
./configure --prefix=$HOME/local --enable-shared
make && make install
~~~~

对于 ncurses：

~~~~
tar -zxf ncurses-*.tar.gz
cd ncurses-*/
./configure --prefix=$HOME/local --with-shared --with-termlib --enable-pc-files --with-pkg-config-libdir=$HOME/local/lib/pkgconfig
make && make install
~~~~

然后 tmux 配置脚本需要使用 `PKG_CONFIG_PATH` 指向本地库：

~~~~
tar -zxf tmux-*.tar.gz
cd tmux-*/
PKG_CONFIG_PATH=$HOME/local/lib/pkgconfig ./configure --prefix=$HOME/local
make && make install
~~~~

如果 ncurses 和 libevent 安装在不同目录而不是全部在 `~/local` 中，
它们的 `lib/pkgconfig` 目录都需要在 `PKG_CONFIG_PATH` 中，例如：

~~~~
PKG_CONFIG_PATH=/opt/libevent/lib/pkgconfig:/opt/ncurses/lib/pkgconfig ./configure --prefix=$HOME/local
~~~~

新构建的 tmux 可以在 `~/local/bin/tmux` 中找到。

当 tmux 在 Linux 上本地安装时，运行时链接器可能需要通过 `LD_LIBRARY_PATH` 环境变量被告知在哪里找到库，例如：

~~~~
LD_LIBRARY_PATH=$HOME/local/lib $HOME/local/bin/tmux -V
~~~~

要查看手册页，必须设置 `MANPATH`：

~~~~
MANPATH=$HOME/local/share/man man tmux
~~~~

大多数用户希望在 shell 配置文件中配置这些，例如 ksh 的 `.profile` 或 bash 的 `.bash_profile`：

~~~~
export PATH=$HOME/local/bin:$PATH
export LD_LIBRARY_PATH=$HOME/local/lib:$LD_LIBRARY_PATH
export MANPATH=$HOME/local/share/man:$MANPATH
~~~~

### 从版本控制

从 Git 构建 tmux 的依赖项与从压缩包构建相同，另外还需要 autoconf 和 automake。构建与从压缩包构建相同，除了首先必须生成配置脚本。要安装到 `/usr/local`：

~~~~
git clone https://github.com/tmux/tmux.git
cd tmux
sh autogen.sh
./configure
make && sudo make install
~~~~

### 配置选项

tmux 提供了一些配置选项：

选项|描述
---|---
`--enable-debug`|使用调试符号构建
`--enable-static`|创建静态构建
`--enable-utempter`|如果安装了则使用 utempter 库
`--enable-utf8proc`|如果安装了则使用 utf8proc 库

请注意，`--enable-static` 可能需要安装静态库，例如在 RHEL 或 CentOS 上需要 `glibc-static` 包。

### 常见问题

#### configure 提示：`libevent not found` 或 `ncurses not found`

libevent 库或其头文件未安装。确保安装了适当的包（一些平台将库和头文件拆分为 `-dev` 或 `-devel` 包）。

#### configure 提示：`must give --enable-utf8proc or --disable-utf8proc`

macOS 内置的 UTF-8 支持非常差，所以最好尽可能使用
[utf8proc](https://juliastrings.github.io/utf8proc/) 库。安装后，
将 `--enable-utf8proc` 传递给 configure。

要强制 tmux 在没有 utf8proc 的情况下构建，请使用 `--disable-utf8proc`。

#### tmux 无法从 `~/local` 运行

在 Linux 上，确保设置了 `LD_LIBRARY_PATH`，或者尝试静态构建（给 configure 添加 `--enable-static`）。

#### `autogen.sh` 抱怨 `AM_BLAH` 或 `PKG_MODULES`

确保安装了 pkg-config。

#### configure 提示：`C compiler cannot create executables`

要么没有安装 C 编译器（gcc、clang），要么它不起作用 - 检查 `CFLAGS` 或 `CPPFLAGS` 中没有愚蠢的内容。

#### 构建失败并提示 "conflicting type for `forkpty`"

对于静态构建，请确保静态 libc 可用。在 RHEL 或 CentOS 上需要 `glibc-static` 包。

### AppImage 包

构建 tmux AppImage 包的说明和脚本可从
[Nelson Enzo 这里](https://github.com/nelsonenzo/tmux-appimage) 获得。预构建的
AppImage 包也可在 [这里](https://github.com/nelsonenzo/tmux-appimage/releases) 获得。

### Docker 脚本

[Docker](https://www.docker.com/) 安装脚本可在
[这里](https://github.com/ferryman0608/Dockerfile-tmux) 获得。

### Red Hat Enterprise Linux / CentOS RPM

从主仓库获得的 tmux 包通常相当过时，特别是对于长期支持发行版。更新的 tmux 版本的 RPM 可以从 [这里](http://galaxy4.net/repo/) 获得。

例如，在 RHEL8 上设置仓库并安装：

~~~~
sudo yum install http://galaxy4.net/repo/galaxy4-release-8-current.noarch.rpm
sudo yum install tmux
~~~~

或者在 RHEL6 上直接安装 RPM：

~~~~
sudo rpm -ivh http://galaxy4.net/repo/RHEL/6/x86_64/tmux-3.1b-2.el6.x86_64.rpm
~~~~

建议使用仓库方法自动接收未来的包更新。更多详细信息请参见 [此页面](https://anni.galaxy4.net/?page_id=39)。