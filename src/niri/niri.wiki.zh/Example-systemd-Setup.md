当从显示管理器（如GDM）启动niri，或通过`niri-session`二进制文件启动时，它作为systemd服务运行。
这提供了运行与图形会话绑定的程序（如`mako`）和服务（如`xdg-desktop-portal`）所需的systemd集成。

以下是如何将[`mako`](https://github.com/emersion/mako)、[`waybar`](https://github.com/Alexays/Waybar)、[`swaybg`](https://github.com/swaywm/swaybg)和[`swayidle`](https://github.com/swaywm/swayidle)设置为systemd服务与niri一起运行的示例。
与[`spawn-at-startup`](./Configuration:-Miscellaneous#spawn-at-startup)不同，这使您可以轻松监控它们的状态和输出，并重启或重新加载它们。

1.  安装它们，即`sudo dnf install mako waybar swaybg swayidle`

2.  `mako`和`waybar`开箱即用地提供systemd单元，所以您可以简单地将它们添加到niri会话中：

        systemctl --user add-wants niri.service mako.service
        systemctl --user add-wants niri.service waybar.service

    这将在`~/.config/systemd/user/niri.service.wants/`中创建链接，这是需要与`niri.service`一起启动的服务的特殊systemd文件夹。

3.  `swaybg`不提供systemd单元，因为您需要将背景图像作为命令行参数传递。
    所以我们将创建自己的。
    创建`~/.config/systemd/user/swaybg.service`，内容如下：

    ```systemd
    [Unit]
    PartOf=graphical-session.target
    After=graphical-session.target
    Requisite=graphical-session.target

    [Service]
    ExecStart=/usr/bin/swaybg -m fill -i "%h/Pictures/LakeSide.png"
    Restart=on-failure
    ```

    将图像路径替换为您想要的路径。
    `%h`扩展为您的主目录。

    编辑`swaybg.service`后，运行`systemctl --user daemon-reload`以便systemd获取文件中的更改。

    现在，将其添加到niri会话中：

        systemctl --user add-wants niri.service swaybg.service

4.  `swayidle`同样不提供服务，所以我们也将创建自己的。
    创建`~/.config/systemd/user/swayidle.service`，内容如下：

    ```systemd
    [Unit]
    PartOf=graphical-session.target
    After=graphical-session.target
    Requisite=graphical-session.target

    [Service]
    ExecStart=/usr/bin/swayidle -w timeout 601 'niri msg action power-off-monitors' timeout 600 'swaylock -f' before-sleep 'swaylock -f'
    Restart=on-failure
    ```

    然后，运行`systemctl --user daemon-reload`并将其添加到niri会话中：

        systemctl --user add-wants niri.service swayidle.service

就是这样！
现在这三个实用程序将与niri会话一起启动，并在退出时停止。
您也可以使用命令如`systemctl --user restart waybar.service`重启它们，例如在编辑配置文件后。

要从niri启动中移除服务，请从`~/.config/systemd/user/niri.service.wants/`中移除其符号链接。
然后，运行`systemctl --user daemon-reload`。

### 跨登出运行程序

当以会话方式运行niri时，退出它（登出）将终止您在其中启动的所有程序。但是，有时您希望一个程序，如`tmux`、`dtach`或类似程序，在这种情况下持续存在。为此，请在临时systemd范围内运行它：

    systemd-run --user --scope tmux new-session