# Z-Shell 用户指南

## Peter Stephenson

## 2003/03/23

# 目录

## [第1章：简介](zshguide01.html#l1)

### [1.1: 其他shell和其他指南](zshguide01.html#l2)

### [1.2: zsh版本](zshguide01.html#l3)

### [1.3: 约定](zshguide01.html#l4)

### [1.4: 致谢](zshguide01.html#l5)

## [第2章：在启动文件中放置什么](zshguide02.html#l6)

### [2.1: shell类型：交互式和登录shell](zshguide02.html#l7)

[2.1.1: 什么是登录shell？简单测试](zshguide02.html#l8)

### [2.2: 所有启动文件](zshguide02.html#l9)

### [2.3: 选项](zshguide02.html#l10)

### [2.4: 参数](zshguide02.html#l11)

[2.4.1: 数组](zshguide02.html#l12)

### [2.5: 在启动文件中放置什么](zshguide02.html#l13)

[2.5.1: 兼容性选项：`SH_WORD_SPLIT` 和其他](zshguide02.html#l14)

[2.5.2: csh 用户的选项](zshguide02.html#l15)

[2.5.3: 历史机制：历史类型](zshguide02.html#l16)

[2.5.4: 设置历史](zshguide02.html#l17)

[2.5.5: 历史选项](zshguide02.html#l18)

[2.5.6: 提示符](zshguide02.html#l19)

[2.5.7: 命名目录](zshguide02.html#l20)

[2.5.8: 给高级用户的'加速'选项](zshguide02.html#l21)

[2.5.9: 别名](zshguide02.html#l22)

[2.5.10: 环境变量](zshguide02.html#l23)

[2.5.11: 路径](zshguide02.html#l24)

[2.5.12: 邮件](zshguide02.html#l25)

[2.5.13: 其他类似路径的东西](zshguide02.html#l26)

[2.5.14: 版本特定内容](zshguide02.html#l27)

[2.5.15: 其他所有内容](zshguide02.html#l28)

## [第3章：处理基本shell语法](zshguide03.html#l29)

### [3.1: 外部命令](zshguide03.html#l30)

### [3.2: 内置命令](zshguide03.html#l31)

[3.2.1: 打印用的内置命令](zshguide03.html#l32)

[3.2.2: 其他仅为速度而设的内置命令](zshguide03.html#l33)

[3.2.3: 更改shell状态的内置命令](zshguide03.html#l34)

[3.2.4: cd和相关命令](zshguide03.html#l35)

[3.2.5: 命令控制和信息命令](zshguide03.html#l36)

[3.2.6: 参数控制](zshguide03.html#l37)

[3.2.7: 历史控制命令](zshguide03.html#l38)

[3.2.8: 作业控制和进程控制](zshguide03.html#l39)

[3.2.9: 终端、用户等](zshguide03.html#l40)

[3.2.10: 语法怪异之处](zshguide03.html#l41)

[3.2.11: 更多前置命令修饰符：`exec`, `noglob`](zshguide03.html#l42)

[3.2.12: 测试事物](zshguide03.html#l43)

[3.2.13: 处理函数和脚本的选项](zshguide03.html#l44)

[3.2.14: 随机文件控制事项](zshguide03.html#l45)

[3.2.15: 不要看这个空间，看别的](zshguide03.html#l46)

[3.2.16: 以及](zshguide03.html#l47)

### [3.3: 函数](zshguide03.html#l48)

[3.3.1: 加载函数](zshguide03.html#l49)

[3.3.2: 函数参数](zshguide03.html#l50)

[3.3.3: 编译函数](zshguide03.html#l51)

### [3.4: 别名](zshguide03.html#l52)

### [3.5: 命令摘要](zshguide03.html#l53)

### [3.6: 扩展和引号](zshguide03.html#l54)

[3.6.1: 历史扩展](zshguide03.html#l55)

[3.6.2: 别名扩展](zshguide03.html#l56)

[3.6.3: 进程、参数、命令、算术和大括号扩展](zshguide03.html#l57)

[3.6.4: 文件名扩展](zshguide03.html#l58)

[3.6.5: 文件名生成](zshguide03.html#l59)

### [3.7: 重定向：大于号和小于号](zshguide03.html#l60)

[3.7.1: 覆盖](zshguide03.html#l61)

[3.7.2: 文件描述符](zshguide03.html#l62)

[3.7.3: 追加、此处文档、此处字符串、读写](zshguide03.html#l63)

[3.7.4: 巧妙技巧：exec和其他文件描述符](zshguide03.html#l64)

[3.7.5: 多重I/O](zshguide03.html#l65)

### [3.8: Shell语法：循环、(子)shell等](zshguide03.html#l66)

[3.8.1: 逻辑命令连接符](zshguide03.html#l67)

[3.8.2: 结构](zshguide03.html#l68)

[3.8.3: 子shell和当前shell构造](zshguide03.html#l69)

[3.8.4: 子shell和当前shell](zshguide03.html#l70)

### [3.9: 模拟和可移植性](zshguide03.html#l71)

[3.9.1: 细节差异](zshguide03.html#l72)

[3.9.2: 使你自己的脚本和函数可移植](zshguide03.html#l73)

### [3.10: 运行脚本](zshguide03.html#l74)

## [第4章：Z-Shell 行编辑器](zshguide04.html#l75)

### [4.1: 介绍zle](zshguide04.html#l76)

[4.1.1: 基本事实](zshguide04.html#l77)

[4.1.2: Vi模式](zshguide04.html#l78)

### [4.2: 基本编辑](zshguide04.html#l79)

[4.2.1: 移动](zshguide04.html#l80)

[4.2.2: 删除](zshguide04.html#l81)

[4.2.3: 更多删除](zshguide04.html#l82)

### [4.3: 高级编辑](zshguide04.html#l83)

[4.3.1: 控制zle的选项](zshguide04.html#l84)

[4.3.2: 小缓冲区和扩展命令](zshguide04.html#l85)

[4.3.3: 前缀（数字）参数](zshguide04.html#l86)

[4.3.4: 单词、区域和标记](zshguide04.html#l87)

[4.3.5: 区域和标记](zshguide04.html#l88)

### [4.4: 历史和搜索](zshguide04.html#l89)

[4.4.1: 在历史中移动](zshguide04.html#l90)

[4.4.2: 在历史中搜索](zshguide04.html#l91)

[4.4.3: 从历史中提取单词](zshguide04.html#l92)

### [4.5: 绑定键和处理键映射](zshguide04.html#l93)

[4.5.1: 简单键绑定](zshguide04.html#l94)

[4.5.2: 删除键绑定](zshguide04.html#l95)

[4.5.3: 功能键等](zshguide04.html#l96)

[4.5.4: 绑定字符串而不是命令](zshguide04.html#l97)

[4.5.5: 键映射](zshguide04.html#l98)

### [4.6: 高级编辑](zshguide04.html#l99)

[4.6.1: 多行编辑](zshguide04.html#l100)

[4.6.2: 内置vared和函数zed](zshguide04.html#l101)

[4.6.3: 缓冲区栈](zshguide04.html#l102)

### [4.7: 扩展zle](zshguide04.html#l103)

[4.7.1: 小部件](zshguide04.html#l104)

[4.7.2: 执行其他小部件](zshguide04.html#l105)

[4.7.3: 一些特殊的内置小部件及其用途](zshguide04.html#l106)

[4.7.4: 特殊参数：普通文本](zshguide04.html#l107)

[4.7.5: 其他特殊参数](zshguide04.html#l108)

[4.7.6: 读取键和使用小缓冲区](zshguide04.html#l109)

[4.7.7: 示例](zshguide04.html#l110)

## [第5章：替换](zshguide05.html#l111)

### [5.1: 引号](zshguide05.html#l112)

[5.1.1: 反斜杠](zshguide05.html#l113)

[5.1.2: 单引号](zshguide05.html#l114)

[5.1.3: POSIX引号](zshguide05.html#l115)

[5.1.4: 双引号](zshguide05.html#l116)

[5.1.5: 反引号](zshguide05.html#l117)

### [5.2: 修饰符及其修饰的内容](zshguide05.html#l118)

### [5.3: 进程替换](zshguide05.html#l119)

### [5.4: 参数替换](zshguide05.html#l120)

[5.4.1: 使用数组](zshguide05.html#l121)

[5.4.2: 使用关联数组](zshguide05.html#l122)

[5.4.3: 替换的替换、开头和结尾等](zshguide05.html#l123)

[5.4.4: 选项标志：分割和连接](zshguide05.html#l124)

[5.4.5: 选项标志：`GLOB_SUBST` 和 `RC_EXPAND_PARAM`](zshguide05.html#l125)

[5.4.6: 更多参数标志](zshguide05.html#l126)

[5.4.7: 几个参数替换技巧](zshguide05.html#l127)

[5.4.8: 嵌套参数替换](zshguide05.html#l128)

### [5.5: 再次替换](zshguide05.html#l129)

### [5.6: 算术扩展](zshguide05.html#l130)

[5.6.1: 输入和输出进制](zshguide05.html#l131)

[5.6.2: 参数类型](zshguide05.html#l132)

### [5.7: 大括号扩展和数组](zshguide05.html#l133)

### [5.8: 文件名扩展](zshguide05.html#l134)

### [5.9: 文件名生成和模式匹配](zshguide05.html#l135)

[5.9.1: 比较模式和正则表达式](zshguide05.html#l136)

[5.9.2: 标准特性](zshguide05.html#l137)

[5.9.3: 通常可用的扩展](zshguide05.html#l138)

[5.9.4: 需要 `EXTENDED_GLOB` 的扩展](zshguide05.html#l139)

[5.9.5: 递归通配](zshguide05.html#l140)

[5.9.6: 通配限定符](zshguide05.html#l141)

[5.9.7: 通配标志：改变匹配行为](zshguide05.html#l142)

[5.9.8: 函数 `zmv`](zshguide05.html#l143)

## [第6章：补全，新旧对比](zshguide06.html#l144)

### [6.1: 补全和扩展](zshguide06.html#l145)

### [6.2: 使用shell选项配置补全](zshguide06.html#l146)

[6.2.1: 模糊补全](zshguide06.html#l147)

[6.2.2: `ALWAYS_LAST_PROMPT`](zshguide06.html#l148)

[6.2.3: 菜单补全和菜单选择](zshguide06.html#l149)

[6.2.4: 其他改变补全行为的方法](zshguide06.html#l150)

[6.2.5: 改变补全显示方式](zshguide06.html#l151)

### [6.3: 开始使用新补全](zshguide06.html#l152)

### [6.4: shell如何找到正确的补全](zshguide06.html#l153)

[6.4.1: 上下文](zshguide06.html#l154)

[6.4.2: 标签](zshguide06.html#l155)

### [6.5: 使用样式配置补全](zshguide06.html#l156)

[6.5.1: 指定补全器及其选项](zshguide06.html#l157)

[6.5.2: 改变列表格式：组等](zshguide06.html#l158)

[6.5.3: 影响特定补全的样式](zshguide06.html#l159)

### [6.6: 命令小部件](zshguide06.html#l160)

[6.6.1: `_complete_help`](zshguide06.html#l161)

[6.6.2: `_correct_word`, `_correct_filename`, `_expand_word`](zshguide06.html#l162)

[6.6.3: `_history_complete_word`](zshguide06.html#l163)

[6.6.4: `_most_recent_file`](zshguide06.html#l164)

[6.6.5: `_next_tags`](zshguide06.html#l165)

[6.6.6: `_bash_completions`](zshguide06.html#l166)

[6.6.7: `_read_comp`](zshguide06.html#l167)

[6.6.8: `_generic`](zshguide06.html#l168)

[6.6.9: `predict-on`, `incremental-complete-word`](zshguide06.html#l169)

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

[6.9.3: 生成文件名等的函数](zshguide06.html#l185)

[6.9.4: `zsh/parameter` 模块](zshguide06.html#l186)

[6.9.5: 特殊补全参数和 `compset`](zshguide06.html#l187)

[6.9.6: 高级补全：使用标签和样式机制](zshguide06.html#l188)

[6.9.7: 让工作为你完成：处理参数等](zshguide06.html#l189)

[6.9.8: 更多补全实用函数](zshguide06.html#l190)

### [6.10: 最后](zshguide06.html#l191)

## [第7章：模块和其他零散内容 *未完成*](zshguide07.html#l192)

### [7.1: 模块控制：`zmodload`](zshguide07.html#l193)

[7.1.1: 定义参数的模块](zshguide07.html#l194)

[7.1.2: 底层系统交互](zshguide07.html#l195)

[7.1.3: ZFTP](zshguide07.html#l196)

### [7.2: 贡献内容](zshguide07.html#l197)

[7.2.1: 提示符主题](zshguide07.html#l198)

### [7.3: 4.1版的新功能](zshguide07.html#l199)

## [附录1：获取zsh和获取更多信息 *未完成*](zshguide08.html#l200)

------------------------------------------------------------------------