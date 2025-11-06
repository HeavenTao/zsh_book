### 高VRAM使用量修复

目前，NVIDIA驱动程序中存在一个特性，影响niri的VRAM使用量（驱动程序无法正确将VRAM释放回池中）。Niri*应该*使用约100 MiB的VRAM（如在[nvtop](https://github.com/Syllo/nvtop)中检查）；如果您看到接近1 GiB的VRAM使用量，您可能遇到了此问题（堆未将释放的缓冲区返回给驱动程序）。

幸运的是，您可以通过配置NVIDIA驱动程序的每进程应用程序配置文件来缓解此问题，如下所示：

*   `sudo mkdir -p /etc/nvidia/nvidia-application-profiles-rc.d`以创建配置目录（如果您正在阅读此内容，它很可能不存在）
*   将以下JSON块写入文件`/etc/nvidia/nvidia-application-profiles-rc.d/50-limit-free-buffer-pool-in-wayland-compositors.json`，为`niri`进程设置`GLVidHeapReuseRatio`配置值：

    ```json
    {
        "rules": [
            {
                "pattern": {
                    "feature": "procname",
                    "matches": "niri"
                },
                "profile": "Limit Free Buffer Pool On Wayland Compositors"
            }
        ],
        "profiles": [
            {
                "name": "Limit Free Buffer Pool On Wayland Compositors",
                "settings": [
                    {
                        "key": "GLVidHeapReuseRatio",
                        "value": 0
                    }
                ]
            }
        ]
    }
    ```

    （`/etc/nvidia/nvidia-application-profiles-rc.d/`中的文件可以命名为任何名称，实际上不需要扩展名）。

写入配置文件后重新启动niri以应用更改。

此解决方案的上游问题在[这里](https://github.com/NVIDIA/egl-wayland/issues/126#issuecomment-2379945259)。NVIDIA更新其内置应用程序配置文件以自动应用此设置的可能性很小；底层启发式方法得到适当修复的可能性不大。

在撰写本文时发布的驱动程序中的修复使用值0，而大约一年前Nvidia工程师发布的初始配置使用值1。

### 屏幕录制闪烁修复

<sup>直到：25.08</sup>

如果您在NVIDIA上有屏幕录制故障或闪烁，请在niri配置中设置以下内容：

```kdl,must-fail
debug {
    wait-for-frame-completion-in-pipewire
}
```

此后已移除此调试标志，因为问题已在niri中得到正确修复。