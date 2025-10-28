## 配置文件配方

此文件列出了一些有用的配置文件片段来控制 tmux
行为。

### 在相同工作目录中创建新窗格

这会更改默认键绑定以添加 `-c` 标志来指定
工作目录：

~~~~
bind '"' split-window -c "#{pane_current_path}"
bind % split-window -hc "#{pane_current_path}"
bind c new-window -c "#{pane_current_path}"
~~~~

### 防止窗格移动环绕

这会阻止窗格移动键在顶部、底部、左侧和
右侧环绕。

需要 tmux 2.6 或更高版本。

~~~~
bind -r Up if -F '#{pane_at_top}' '' 'selectp -U'
bind -r Down if -F '#{pane_at_bottom}' '' 'selectp -D'
bind -r Left if -F '#{pane_at_left}' '' 'selectp -L'
bind -r Right if -F '#{pane_at_right}' '' 'selectp -R'
~~~~

### 为鼠标滚轮发送 `Up` 和 `Down` 键

当应用程序未启用鼠标时，某些终端默认执行此操作。
这在 tmux 中执行相同操作（`mouse` 选项也必须为 `on`）：

~~~~
bind -n WheelUpPane if -Ft= "#{mouse_any_flag}" "send -M" "send Up"
bind -n WheelDownPane if -Ft= "#{mouse_any_flag}" "send -M" "send Down"
~~~~

另一种方法是仅在备用屏幕中发送按键：

~~~~
bind -n WheelUpPane {
	if -F '#{||:#{pane_in_mode},#{mouse_any_flag}}' {
		send -M
	} {
		if -F '#{alternate_on}' { send-keys -N 3 Up } { copy-mode -e }
	}
}
bind -n WheelDownPane {
	if -F '#{||:#{pane_in_mode},#{mouse_any_flag}}' {
		send -M
	} {
		if -F '#{alternate_on}' { send-keys -N 3 Down }
	}
}
~~~~

### 使 `C-b w` 绑定仅显示一个会话

这使得 `C-b w` 树模式绑定仅显示附加会话中的窗口。

~~~~
bind w run 'tmux choose-tree -Nwf"##{==:##{session_name},#{session_name}}"'
~~~~

### 创建新窗格进行复制

这会打开一个具有活动窗格历史记录的新窗格 - 用于从历史记录中复制
多个项目到 shell 提示符很有用。

需要 tmux 3.2 或更高版本。

~~~~
bind C {
	splitw -f -l30% ''
	set-hook -p pane-mode-changed 'if -F "#{!=:#{pane_mode},copy-mode}" "kill-pane"'
	copy-mode -s'{last}'
}
~~~~

### C-双击打开 *emacs(1)*

C-双击一个单词以在弹出窗口中打开 *emacs(1)*（处理 `file:line`）。

需要 tmux 3.2 或更高版本。

~~~~
set -g word-separators ""
bind -n C-DoubleClick1Pane if -F '#{m/r:^[^:]*:[0-9]+:,#{mouse_word}}' {
	run -C 'popup -w90% -h90% -E -d "#{pane_current_path}" "
		x=$(echo "#{mouse_word}"|awk -F: "{print \"+\"\$2,\$1}")
		emacs "$x"
	"'
} {
	run -C 'popup -w90% -h90% -E -d "#{pane_current_path}" "
		emacs "#{mouse_word}"
	"'
}
~~~~

### 更改树模式中的快捷键

这会按当前会话的窗口索引分配快捷键
而不是按行号，并使用 `a` 到 `z` 表示更高的数字而不是
`M-a` 到 `M-z`。请注意，这会替换 `a` 到 `z` 键的现有用途。

~~~~
bind w run -C { choose-tree -ZwK "##{?##{!=:#{session_name},##{session_name}},,##{?window_format,##{?##{e|<:##{window_index},10},##{window_index},##{?##{e|<:##{window_index},36},##{a:##{e|+:##{e|-:##{window_index},10},97}},}},}}" }
~~~~