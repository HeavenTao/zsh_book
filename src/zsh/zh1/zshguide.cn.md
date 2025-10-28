------------------------------------------------------------------------

# Z-Shell 用户指南

## Peter Stephenson

## 2003/03/23

# 目录

## [第一章：简短介绍](zshguide01.html#l1)

### [1.1: 其他 shell 和其他指南](zshguide01.html#l2)

### [1.2: zsh 的版本](zshguide01.html#l3)

### [1.3: 约定](zshguide01.html#l4)

### [1.4: 致谢](zshguide01.html#l5)

## [第二章：启动文件中要放些什么](zshguide02.html#l6)

### [2.1: Shell 类型：交互式和登录 shell](zshguide02.html#l7)

[2.1.1: 什么是登录 shell？简单测试](zshguide02.html#l8)

### [2.2: 所有的启动文件](zshguide02.html#l9)

### [2.3: 选项](zshguide02.html#l10)

### [2.4: 参数](zshguide02.html#l11)

[2.4.1: 数组](zshguide02.html#l12)

### [2.5: 启动文件中要放些什么](zshguide02.html#l13)

[2.5.1: 兼容性选项：`SH_WORD_SPLIT` 及其他](zshguide02.html#l14)

[2.5.2: 为 csh 爱好者准备的选项](zshguide02.html#l15)

[2.5.3: 历史记录机制：历史记录的类型](zshguide02.html#l16)

[2.5.4: 设置历史记录](zshguide02.html#l17)

[2.5.5: 历史记录选项](zshguide02.html#l18)

[2.5.6: 提示符](zshguide02.html#l19)

[2.5.7: 命名目录](zshguide02.html#l20)

[2.5.8: 为高级用户准备的“加速”选项](zshguide02.html#l21)

[2.5.9: 别名](zshguide02.html#l22)

[2.5.10: 环境变量](zshguide02.html#l23)

[2.5.11: 路径](zshguide02.html#l24)

[2.5.12: 邮件](zshguide02.html#l25)

[2.5.13: 其他类似路径的东西](zshguide02.html#l26)

[2.5.14: 特定版本的东西](zshguide02.html#l27)

[2.5.15: 其他所有东西](zshguide02.html#l28)

## [第三章：处理基本 shell 语法](zshguide03.html#l29)

### [3.1: 外部命令](zshguide03.html#l30)

### [3.2: 内建命令](zshguide03.html#l31)

[3.2.1: 用于打印的内建命令](zshguide03.html#l32)

[3.2.2: 其他仅为速度而设的内建命令](zshguide03.html#l33)

[3.2.3: 改变 shell 状态的内建命令](zshguide03.html#l34)

[3.2.4: cd 和它的朋友们](zshguide03.html#l35)

[3.2.5: 命令控制和信息命令](zshguide03.html#l36)

[3.2.6: 参数控制](zshguide03.html#l37)

[3.2.7: 历史记录控制命令](zshguide03.html#l38)

[3.2.8: 作业控制和进程控制](zshguide03.html#l39)

[3.2.9: 终端、用户等](zshguide03.html#l40)

[3.2.10: 语法零碎](zshguide03.html#l41)

[3.2.11: 更多的前置命令修饰符：`exec`、`noglob`](zshguide03.html#l42)

[3.2.12: 测试事物](zshguide03.html#l43)

[3.2.13: 处理函数和脚本的选项](zshguide03.html#l44)

[3.2.14: 随机文件控制相关](zshguide03.html#l45)

[3.2.15: 不要看这里，看别处](zshguide03.html#l46)

[3.2.16: 以及其他](zshguide03.html#l47)

### [3.3: 函数](zshguide03.html#l48)

[3.3.1: 加载函数](zshguide03.html#l49)

[3.3.2: 函数参数](zshguide03.html#l50)

[3.3.3: 编译函数](zshguide03.html#l51)

### [3.4: 别名](zshguide03.html#l52)

### [3.5: 命令摘要](zshguide03.html#l53)

### [3.6: 展开和引用](zshguide03.html#l54)

[3.6.1: 历史展开](zshguide03.html#l55)

[3.6.2: 别名展开](zshguide03.html#l56)

[3.6.3: 进程、参数、命令、算术和花括号展开](zshguide03.html#l57)

[3.6.4: 文件名展开](zshguide03.html#l58)

[3.6.5: 文件名生成](zshguide03.html#l59)

### [3.7: 重定向：大于号和小于号](zshguide03.html#l60)

[3.7.1: Clobber（覆盖）](zshguide03.html#l61)

[3.7.2: 文件描述符](zshguide03.html#l62)

[3.7.3: 追加、here-document、here-string、读写](zshguide03.html#l63)

[3.7.4: 巧妙的技巧：exec 和其他文件描述符](zshguide03.html#l64)

[3.7.5: Multios（多路输入输出）](zshguide03.html#l65)

### [3.8: Shell 语法：循环、(子)shell 等](zshguide03.html#l66)

[3.8.1: 逻辑命令连接符](zshguide03.html#l67)

[3.8.2: 结构](zshguide03.html#l68)

[3.8.3: 子 shell 和当前 shell 结构](zshguide03.html#l69)

[3.8.4: 子 shell 和当前 shell](zshguide03.html#l70)

### [3.9: 仿真和可移植性](zshguide03.html#l71)

[3.9.1: 细节差异](zshguide03.html#l72)

[3.9.2: 使你自己的脚本和函数可移植](zshguide03.html#l73)

### [3.10: 运行脚本](zshguide03.html#l74)

## [第四章：Z-Shell 行编辑器](zshguide04.html#l75)

### [4.1: zle 介绍](zshguide04.html#l76)

[4.1.1: 简单事实](zshguide04.html#l77)

[4.1.2: Vi 模式](zshguide04.html#l78)

### [4.2: 基本编辑](zshguide04.html#l79)

[4.2.1: 移动](zshguide04.html#l80)

[4.2.2: 删除](zshguide04.html#l81)

[4.2.3: 更多删除操作](zshguide04.html#l82)

### [4.3: 更高级的编辑](zshguide04.html#l83)

[4.3.1: 控制 zle 的选项](zshguide04.html#l84)

[4.3.2: minibuffer 和扩展命令](zshguide04.html#l85)

[4.3.3: 前缀（数字）参数](zshguide04.html#l86)

[4.3.4: 单词、区域和标记](zshguide04.html#l87)

[4.3.5: 区域和标记](zshguide04.html#l88)

### [4.4: 历史记录和搜索](zshguide04.html#l89)

[4.4.1: 在历史记录中移动](zshguide04.html#l90)

[4.4.2: 搜索历史记录](zshguide04.html#l91)

[4.4.3: 从历史记录中提取单词](zshguide04.html#l92)

### [4.5: 绑定按键和处理键位映射](zshguide04.html#l93)

[4.5.1: 简单按键绑定](zshguide04.html#l94)

[4.5.2: 移除按键绑定](zshguide04.html#l95)

[4.5.3: 功能键等](zshguide04.html#l96)

[4.5.4: 绑定字符串而非命令](zshguide04.html#l97)

[4.5.5: 键位映射](zshguide04.html#l98)

### [4.6: 高级编辑](zshguide04.html#l99)

[4.6.1: 多行编辑](zshguide04.html#l100)

[4.6.2: 内建命令 vared 和函数 zed](zshguide04.html#l101)

[4.6.3: 缓冲区堆栈](zshguide04.html#l102)

### [4.7: 扩展 zle](zshguide04.html#l103)

[4.7.1: 小部件（Widgets）](zshguide04.html#l104)

[4.7.2: 执行其他小部件](zshguide04.html#l105)

[4.7.3: 一些特殊的内建小部件及其用途](zshguide04.html#l106)

[4.7.4: 特殊参数：普通文本](zshguide04.html#l107)

[4.7.5: 其他特殊参数](zshguide04.html#l108)

[4.7.6: 读取按键和使用 minibuffer](zshguide04.html#l109)

[4.7.7: 示例](zshguide04.html#l110)

## [第五章：替换](zshguide05.html#l111)

### [5.1: 引用](zshguide05.html#l112)

[5.1.1: 反斜杠](zshguide05.html#l113)

[5.1.2: 单引号](zshguide05.html#l114)

[5.1.3: POSIX 引号](zshguide05.html#l115)

[5.1.4: 双引号](zshguide05.html#l116)

[5.1.5: 反引号](zshguide05.html#l117)

### [5.2: 修饰符及其修饰对象](zshguide05.html#l118)

### [5.3: 进程替换](zshguide05.html#l119)

### [5.4: 参数替换](zshguide05.html#l120)

[5.4.1: 使用数组](zshguide05.html#l121)

[5.4.2: 使用关联数组](zshguide05.html#l122)

[5.4.3: 替换的替换、首尾处理等](zshguide05.html#l123)

[5.4.4: 选项标志：分割和连接](zshguide05.html#l124)

[5.4.5: 选项标志：`GLOB_SUBST` 和 `RC_EXPAND_PARAM`](zshguide05.html#l125)

[5.4.6: 更多参数标志](zshguide05.html#l126)

[5.4.7: 一些参数替换技巧](zshguide05.html#l127)

[5.4.8: 嵌套参数替换](zshguide05.html#l128)

### [5.5: 再次讨论替换](zshguide05.html#l129)

### [5.6: 算术展开](zshguide05.html#l130)

[5.6.1: 输入和输出基数](zshguide05.html#l131)

[5.6.2: 参数类型](zshguide05.html#l132)

### [5.7: 花括号展开和数组](zshguide05.html#l133)

### [5.8: 文件名展开](zshguide05.html#l134)

### [5.9: 文件名生成和模式匹配](zshguide05.html#l135)

[5.9.1: 比较模式和正则表达式](zshguide05.html#l136)

[5.9.2: 标准特性](zshguide05.html#l137)

[5.9.3: 通常可用的扩展](zshguide05.html#l138)

[5.9.4: 需要 `EXTENDED_GLOB` 的扩展](zshguide05.html#l139)

[5.9.5: 递归 globbing](zshguide05.html#l140)

[5.9.6: Glob 限定符](zshguide05.html#l141)

[5.9.7: Globbing 标志：改变匹配行为](zshguide05.html#l142)

[5.9.8: 函数 `zmv`](zshguide05.html#l143)

## [第六章：补全，旧与新](zshguide06.html#l144)

### [6.1: 补全和展开](zshguide06.html#l145)

### [6.2: 使用 shell 选项配置补全](zshguide06.html#l146)

[6.2.1: 模糊补全](zshguide06.html#l147)

[6.2.2: `ALWAYS_LAST_PROMPT`](zshguide06.html#l148)

[6.2.3: 菜单补全和菜单选择](zshguide06.html#l149)

[6.2.4: 改变补全行为的其他方式](zshguide06.html#l150)

[6.2.5: 改变补全的显示方式](zshguide06.html#l151)

### [6.3: 新补全入门](zshguide06.html#l152)

### [6.4: shell 如何找到正确的补全](zshguide06.html#l153)

[6.4.1: 上下文（Contexts）](zshguide06.html#l154)

[6.4.2: 标签（Tags）](zshguide06.html#l155)

### [6.5: 使用样式配置补全](zshguide06.html#l156)

[6.5.1: 指定补全器及其选项](zshguide06.html#l157)

[6.5.2: 改变列表格式：分组等](zshguide06.html#l158)

[6.5.3: 影响特定补全的样式](zshguide06.html#l159)

### [6.6: 命令小部件](zshguide06.html#l160)

[6.6.1: `_complete_help`](zshguide06.html#l161)

[6.6.2: `_correct_word`、`_correct_filename`、`_expand_word`](zshguide06.html#l162)

[6.6.3: `_history_complete_word`](zshguide06.html#l163)

[6.6.4: `_most_recent_file`](zshguide06.html#l164)

[6.6.5: `_next_tags`](zshguide06.html#l165)

[6.6.6: `_bash_completions`](zshguide06.html#l166)

[6.6.7: `_read_comp`](zshguide06.html#l167)

[6.6.8: `_generic`](zshguide06.html#l168)

[6.6.9: `predict-on`、`incremental-complete-word`](zshguide06.html#l169)

### [6.7: 匹配控制和控制插入位置](zshguide06.html#l170)

[6.7.1: 不区分大小写匹配](zshguide06.html#l171)

[6.7.2: 匹配选项名称](zshguide06.html#l172)

[6.7.3: 部分单词补全](zshguide06.html#l173)

[6.7.4: 子字符串补全](zshguide06.html#l174)

[6.7.5: 带大写字母的部分单词](zshguide06.html#l175)

[6.7.6: 最后说明](zshguide06.html#l176)

### [6.8: 教程](zshguide06.html#l177)

[6.8.1: 调度器](zshguide06.html#l178)

[6.8.2: 子命令补全：`_arguments`](zshguide06.html#l179)

[6.8.3: 补全特定参数类型](zshguide06.html#l180)

[6.8.4: 其余部分](zshguide06.html#l181)

### [6.9: 编写新的补全函数和小部件](zshguide06.html#l182)

[6.9.1: 加载补全函数：`compdef`](zshguide06.html#l183)

[6.9.2: 添加一组补全：`compadd`](zshguide06.html#l184)

[6.9.3: 用于生成文件名等的函数](zshguide06.html#l185)

[6.9.4: `zsh/parameter` 模块](zshguide06.html#l186)

[6.9.5: 特殊补全参数和 `compset`](zshguide06.html#l187)

[6.9.6: 更高级的补全：使用标签和样式机制](zshguide06.html#l188)

[6.9.7: 让他人为你完成工作：处理参数等](zshguide06.html#l189)

[6.9.8: 更多补全实用函数](zshguide06.html#l190)

### [6.10: 最后](zshguide06.html#l191)

## [第七章：模块及其他零碎 *未完成*](zshguide07.html#l192)

### [7.1: 模块控制：`zmodload`](zshguide07.html#l193)

[7.1.1: 定义参数的模块](zshguide07.html#l194)

[7.1.2: 底层系统交互](zshguide07.html#l195)

[7.1.3: ZFTP](zshguide07.html#l196)

### [7.2: 贡献部分](zshguide07.html#l197)

[7.2.1: 提示符主题](zshguide07.html#l198)

### [7.3: 4.1 版本的新功能](zshguide07.html#l199)

## [附录 1：获取 zsh 和更多信息 *未完成*](zshguide08.html#l200)

------------------------------------------------------------------------
