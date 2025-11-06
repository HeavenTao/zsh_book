::: page-top
[]{#top_of_page}
:::

::: nav-bar
  ------------------------------------------------------------------------ -------------------------------------------------------------------------------------
  [man7.org](../../../index.html) \> Linux \> [man-pages](../index.html)   [Linux/UNIX system programming training](http://man7.org/training/){.training-link}
  ------------------------------------------------------------------------ -------------------------------------------------------------------------------------
:::

------------------------------------------------------------------------

# termios(3) --- Linux 手册页面

+-----------------------------------+-----------------------------------+
| [名称](#NAME) \|                  |                                   |
| [库](#LIBRARY) \|                 |                                   |
| [概要](#SYNOPSIS) \|              |                                   |
| [描述](#DESCRIPTION) \|           |                                   |
| [返回值](#RETURN_VALUE) \|        |                                   |
| [属性](#ATTRIBUTES) \|            |                                   |
| [标准](#STANDARDS) \|             |                                   |
| [历史](#HISTORY) \|               |                                   |
| [注意事项](#NOTES) \| [错误](#BUGS)|                                   |
| \| [参见](#SEE_ALSO) \|           |                                   |
| [版权信息](#COLOPHON)             |                                   |
+-----------------------------------+-----------------------------------+
| ::: man-search-box                |                                   |
| :::                               |                                   |
+-----------------------------------+-----------------------------------+

    termios(3)               库函数手册              termios(3)

## [](#NAME){#NAME}名称         [[top]{.top-link}](#top_of_page)

           termios, tcgetattr, tcsetattr, tcsendbreak, tcdrain, tcflush,
           tcflow, cfmakeraw, cfgetospeed, cfgetispeed, cfsetispeed,
           cfsetospeed, cfsetspeed - 获取和设置终端属性，线路
           控制，获取和设置波特率

## [](#LIBRARY){#LIBRARY}库         [[top]{.top-link}](#top_of_page)

           标准C库 (libc, -lc)

## [](#SYNOPSIS){#SYNOPSIS}概要         [[top]{.top-link}](#top_of_page)

           #include <termios.h>
           #include <unistd.h>

           int tcgetattr(int fd, struct termios *termios_p);
           int tcsetattr(int fd, int optional_actions,
                         const struct termios *termios_p);

           int tcsendbreak(int fd, int duration);
           int tcdrain(int fd);
           int tcflush(int fd, int queue_selector);
           int tcflow(int fd, int action);

           void cfmakeraw(struct termios *termios_p);

           speed_t cfgetispeed(const struct termios *termios_p);
           speed_t cfgetospeed(const struct termios *termios_p);

           int cfsetispeed(struct termios *termios_p, speed_t speed);
           int cfsetospeed(struct termios *termios_p, speed_t speed);
           int cfsetspeed(struct termios *termios_p, speed_t speed);

       glibc的特性测试宏要求 (参见
       feature_test_macros(7)):

           cfsetspeed(), cfmakeraw():
               自glibc 2.19起:
                   _DEFAULT_SOURCE
               glibc 2.19及更早版本:
                   _BSD_SOURCE

## [](#DESCRIPTION){#DESCRIPTION}描述         [[top]{.top-link}](#top_of_page)

           termios函数描述了一个通用的终端接口，
           用于控制异步通信端口。

       termios结构
           这里描述的许多函数都有一个termios_p参数，
           它是指向termios结构的指针。该结构包含
           至少以下成员:

               tcflag_t c_iflag;      /* 输入模式 */
               tcflag_t c_oflag;      /* 输出模式 */
               tcflag_t c_cflag;      /* 控制模式 */
               tcflag_t c_lflag;      /* 本地模式 */
               cc_t     c_cc[NCCS];   /* 特殊字符 */

           可以分配给这些字段的值在下面描述。
           对于前四个位掩码字段，只有当定义了特定的特性测试宏时(参见
           feature_test_macros(7))，才会公开一些可设置的相关标志的定义，
           如方括号("[]")中所注明的那样。

           在下面的描述中，"not in POSIX"表示该值未在
           POSIX.1-2001中指定，而"XSI"表示该值在
           POSIX.1-2001中作为XSI扩展的一部分指定。

           c_iflag标志常量:

           IGNBRK 忽略输入上的BREAK条件。

           BRKINT 如果设置了IGNBRK，则忽略BREAK。
                  如果未设置IGNBRK但设置了BRKINT，则BREAK导致
                  输入和输出队列被刷新，如果终端是
                  前台进程组的控制终端，则会
                  向此前台进程组发送SIGINT。
                  当既未设置IGNBRK也未设置BRKINT时，BREAK读取为
                  空字节('\0')，除非设置了PARMRK，在这种情况下
                  它读取为序列\377 \0 \0。

           IGNPAR 忽略帧错误和奇偶校验错误。

           PARMRK 如果设置了此位，在传递到程序时，
                  具有奇偶校验或帧错误的输入字节会被标记。
                  此位仅在设置了INPCK且未设置IGNPAR时有意义。
                  错误字节的标记方式是前面有两个字节，
                  \377和\0。因此，程序实际上读取三个字节
                  以接收终端发送的一个错误字节。
                  如果有效字节的值为\377，且未设置ISTRIP(见下文)，
                  程序可能会将其与标记奇偶校验错误的前缀混淆。
                  因此，在这种情况下，有效字节\377
                  作为两个字节\377 \377传递给程序。

                  如果既未设置IGNPAR也未设置PARMRK，则将具有
                  奇偶校验错误或帧错误的字符读取为\0。

           INPCK  启用输入奇偶校验检查。

           ISTRIP 去掉第八位。

           INLCR  输入时将NL转换为CR。

           IGNCR  输入时忽略回车。

           ICRNL  输入时将回车转换为换行(除非
                  设置了IGNCR)。

           IUCLC  (不在POSIX中) 输入时将大写字符映射为小写。

           IXON   在输出上启用XON/XOFF流控制。

           IXANY  (XSI) 输入任何字符将重新启动停止的输出。
                  (默认只允许START字符重新启动输出。)

           IXOFF  在输入上启用XON/XOFF流控制。

           IMAXBEL
                  (不在POSIX中) 当输入队列满时响铃。
                  Linux不实现此位，且行为如同始终
                  设置一样。

           IUTF8 (自Linux 2.6.4起)
                  (不在POSIX中) 输入为UTF8；这允许字符-擦除
                  在规范模式下正确执行。

           c_oflag标志常量:

           OPOST  启用实现定义的输出处理。

           OLCUC  (不在POSIX中) 输出时将小写字符映射为大写。

           ONLCR  (XSI) 输出时将NL映射为CR-NL。

           OCRNL  输出时将CR映射为NL。

           ONOCR  在第0列不输出CR。

           ONLRET NL字符被假定执行回车功能；
                  内核对当前列的概念在NL和CR之后
                  设置为0。

           OFILL  发送填充字符以延迟，而不是使用定时
                  延迟。

           OFDEL  填充字符为ASCII DEL (0177)。如果未设置，填充
                  字符为ASCII NUL ('\0')。(在Linux上未实现。)

           NLDLY  换行延迟掩码。值为NL0和NL1。[需要
                  _BSD_SOURCE或_SVID_SOURCE或_XOPEN_SOURCE]

           CRDLY  回车延迟掩码。值为CR0、CR1、CR2或
                  CR3。[需要_BSD_SOURCE或_SVID_SOURCE或
                  _XOPEN_SOURCE]

           TABDLY 水平制表符延迟掩码。值为TAB0、TAB1、TAB2、
                  TAB3(或XTABS，但请参见BUGS部分)。
                  TAB3，即XTABS，将制表符扩展为空格(制表符
                  每八列停止一次)。[需要_BSD_SOURCE或
                  _SVID_SOURCE或_XOPEN_SOURCE]

           BSDLY  退格延迟掩码。值为BS0或BS1。(从未
                  实现过。)[需要_BSD_SOURCE或_SVID_SOURCE
                  或_XOPEN_SOURCE]

           VTDLY  垂直制表符延迟掩码。值为VT0或VT1。

           FFDLY  换页延迟掩码。值为FF0或FF1。[需要
                  _BSD_SOURCE或_SVID_SOURCE或_XOPEN_SOURCE]

           c_cflag标志常量:

           CBAUD  (不在POSIX中) 波特率掩码(4+1位)。[需要
                  _BSD_SOURCE或_SVID_SOURCE]

           CBAUDEX
                  (不在POSIX中) 额外波特率掩码(1位)，包含在
                  CBAUD中。[需要_BSD_SOURCE或_SVID_SOURCE]

                  (POSIX规定波特率存储在termios
                  结构中，但未具体指定位置，并提供
                  cfgetispeed()和cfsetispeed()来获取它。
                  一些系统在c_cflag中使用CBAUD选择的位，
                  其他系统使用单独的字段，例如sg_ispeed和
                  sg_ospeed。)

           CSIZE  字符大小掩码。值为CS5、CS6、CS7或CS8。

           CSTOPB 设置两个停止位，而不是一个。

           CREAD  启用接收器。

           PARENB 启用输出上的奇偶校验生成和输入上的奇偶校验检查。

           PARODD 如果设置，则输入和输出的奇偶校验为奇；否则
                  使用偶校验。

           HUPCL  最后一个进程关闭设备后降低调制解调器控制线
                  (挂断)。

           CLOCAL 忽略调制解调器控制线。

           LOBLK  (不在POSIX中) 阻止来自非当前shell层的输出。
                  供shl(shell层)使用。(在Linux上未实现。)

           CIBAUD (不在POSIX中) 输入速度掩码。CIBAUD位的值
                  与CBAUD位的值相同，
                  左移IBSHIFT位。[需要_BSD_SOURCE或
                  _SVID_SOURCE](在glibc中未实现，在Linux上
                  通过TCGET*和TCSET* ioctl支持；参见ioctl_tty(2))

           CMSPAR (不在POSIX中) 使用"粘性"(标记/空格)奇偶校验(在
                  某些串行设备上支持)：如果设置了PARODD，
                  奇偶校验位始终为1；如果未设置PARODD，
                  则奇偶校验位始终为0。[需要_BSD_SOURCE或_SVID_SOURCE]

           CRTSCTS
                  (不在POSIX中) 启用RTS/CTS(硬件)流控制。
                  [需要_BSD_SOURCE或_SVID_SOURCE]

           c_lflag标志常量:

           ISIG   当接收到INTR、QUIT、SUSP或DSUSP字符中的
                  任何字符时，生成相应的信号。

           ICANON 启用规范模式(如下所述)。

           XCASE  (不在POSIX中；Linux下不支持) 如果也
                  设置了ICANON，则终端仅显示大写。
                  输入被转换为小写，除了以\开头的字符。
                  在输出时，大写字符前面加上\，小写
                  字符被转换为大写。[需要
                  _BSD_SOURCE或_SVID_SOURCE或_XOPEN_SOURCE]

           ECHO   回显输入字符。

           ECHOE  如果也设置了ICANON，则ERASE字符擦除
                  前一个输入字符，WERASE擦除前一个
                  单词。

           ECHOK  如果也设置了ICANON，则KILL字符擦除
                  当前行。

           ECHONL 如果也设置了ICANON，则即使ECHO
                  未设置，也回显NL字符。

           ECHOCTL
                  (不在POSIX中) 如果也设置了ECHO，则
                  除TAB、NL、START和STOP之外的终端特殊
                  字符将回显为^X，其中X是ASCII码比
                  特殊字符大0x40的字符。例如，字符
                  0x08 (BS)回显为^H。[需要_BSD_SOURCE或
                  _SVID_SOURCE]

           ECHOPRT
                  (不在POSIX中) 如果也设置了ICANON和ECHO，则字符
                  在被擦除时打印。[需要
                  _BSD_SOURCE或_SVID_SOURCE]

           ECHOKE (不在POSIX中) 如果也设置了ICANON，则按照ECHOE
                  和ECHOPRT指定的方式，通过擦除行上的每个字符来
                  回显KILL。[需要_BSD_SOURCE或_SVID_SOURCE]

           DEFECHO
                  (不在POSIX中) 仅当进程正在读取时回显。(在Linux上
                  未实现。)

           FLUSHO (不在POSIX中；Linux下不支持) 正在
                  刷新输出。此标志通过输入DISCARD
                  字符切换。[需要_BSD_SOURCE或_SVID_SOURCE]

           NOFLSH 在为INT、QUIT和SUSP字符
                  生成信号时禁用刷新输入和输出队列。

           TOSTOP 向试图写入其控制终端的
                  后台进程的进程组发送SIGTTOU信号。

           PENDIN (不在POSIX中；Linux下不支持) 当读取下一个字符时，
                  输入队列中的所有字符都会重新打印。
                  (bash(1)以这种方式处理预输入。)[需要
                  _BSD_SOURCE或_SVID_SOURCE]

           IEXTEN 启用实现定义的输入处理。此标志，
                  以及ICANON必须启用才能解释特殊
                  字符EOL2、LNEXT、REPRINT、WERASE，
                  并使IUCLC标志生效。

           c_cc数组定义终端特殊字符。符号
           索引(初始值)和含义如下:

           VDISCARD
                  (不在POSIX中；Linux下不支持；017，SI，Ctrl-O)
                  切换：开始/停止丢弃待处理输出。当
                  设置了IEXTEN时识别，然后不作为输入传递。

           VDSUSP (不在POSIX中；Linux下不支持；031，EM，Ctrl-Y)
                  延迟挂起字符(DSUSP)：当用户程序
                  读取字符时发送SIGTSTP信号。当
                  设置了IEXTEN和ISIG，且系统支持作业
                  控制时识别，然后不作为输入传递。

           VEOF   (004，EOT，Ctrl-D) 文件结束字符(EOF)。更
                  精确地说：此字符导致待处理的tty缓冲区
                  发送到等待的用户程序，无需等待
                  行结束。如果它是行的第一个字符，则
                  用户程序中的read(2)返回0，表示文件结束。
                  当设置了ICANON时识别，然后不
                  作为输入传递。

           VEOL   (0，NUL) 额外的行结束字符(EOL)。
                  当设置了ICANON时识别。

           VEOL2  (不在POSIX中；0，NUL) 另一个行结束字符
                  (EOL2)。当设置了ICANON时识别。

           VERASE (0177，DEL，rubout，或010，BS，Ctrl-H，或#) 擦除
                  字符(ERASE)。这会擦除前一个尚未擦除的
                  字符，但不会擦除EOF或行开始之后的
                  内容。当设置了ICANON时识别，然后不作为输入
                  传递。

           VINTR  (003，ETX，Ctrl-C，或0177，DEL，rubout) 中断
                  字符(INTR)。发送SIGINT信号。当
                  设置了ISIG时识别，然后不作为输入
                  传递。

           VKILL  (025，NAK，Ctrl-U，或Ctrl-X，或@) 删除字符
                  (KILL)。这会擦除从上次EOF或
                  行开始以来的输入。当设置了ICANON时识别，
                  然后不作为输入传递。

           VLNEXT (不在POSIX中；026，SYN，Ctrl-V) 字面意思下一个(LNEXT)。
                  引用下一个输入字符，剥夺其可能的
                  特殊含义。当设置了IEXTEN时识别，然后
                  不作为输入传递。

           VMIN   非规范读取的最小字符数(MIN)。

           VQUIT  (034，FS，Ctrl-\) 退出字符(QUIT)。发送SIGQUIT
                  信号。当设置了ISIG时识别，然后不作为输入
                  传递。

           VREPRINT
                  (不在POSIX中；022，DC2，Ctrl-R) 重印未读字符
                  (REPRINT)。当设置了ICANON和IEXTEN时识别，然后
                  不作为输入传递。

           VSTART (021，DC1，Ctrl-Q) 开始字符(START)。重新启动
                  被停止字符停止的输出。当设置了IXON时识别，
                  然后不作为输入传递。

           VSTATUS
                  (不在POSIX中；Linux下不支持；状态请求：
                  024，DC4，Ctrl-T)。状态字符(STATUS)。在终端显示
                  状态信息，包括前台进程的状态和
                  消耗的CPU时间量。
                  还向前台进程组发送SIGINFO信号(Linux上不支持)。

           VSTOP  (023，DC3，Ctrl-S) 停止字符(STOP)。停止输出
                  直到输入开始字符。当设置了IXON时识别，
                  然后不作为输入传递。

           VSUSP  (032，SUB，Ctrl-Z) 挂起字符(SUSP)。发送SIGTSTP
                  信号。当设置了ISIG时识别，然后不作为输入
                  传递。

           VSWTCH (不在POSIX中；Linux下不支持；0，NUL) 切换
                  字符(SWTCH)。在System V中用于在
                  shell层之间切换，shell作业控制的前身。

           VTIME  非规范读取的十分之一秒超时(TIME)。

           VWERASE
                  (不在POSIX中；027，ETB，Ctrl-W) 单词擦除(WERASE)。
                  当设置了ICANON和IEXTEN时识别，然后不
                  作为输入传递。

           可通过将相应c_cc元素的值设置为
           _POSIX_VDISABLE来禁用单个终端特殊字符。

           上述符号下标值都不同，除了
           VTIME、VMIN可能分别与VEOL、VEOF具有相同的值。
           在非规范模式下，特殊字符含义被
           超时含义替换。有关VMIN和VTIME的说明，请参见
           下面对非规范模式的描述。

       检索和更改终端设置
           tcgetattr()获取与fd引用的对象关联的参数
           并将其存储在termios_p引用的termios结构中。
           此函数可以从后台
           进程调用；但是，终端属性可能会被
           前台进程随后
           更改。

           tcsetattr()设置与终端关联的参数
           (除非需要底层硬件的
           支持而不可用)从termios_p引用的
           termios结构。optional_actions指定更改何时
           生效:

           TCSANOW
                  更改立即发生。

           TCSADRAIN
                  在写入fd的所有输出传输完成后
                  发生更改。更改
                  影响输出的参数时应使用此选项。

           TCSAFLUSH
                  在写入fd引用的对象的所有输出传输完成后
                  发生更改，并且在更改
                  之前丢弃所有已接收但未读取的输入。

       规范和非规范模式
           c_lflag中ICANON规范标志的设置决定
           终端是运行在规范模式(ICANON设置)还是
           非规范模式(ICANON未设置)。默认情况下，设置ICANON。

           在规范模式下:

           •  输入按行提供。输入行在
              键入行分隔符之一时可用(NL、EOL、
              EOL2；或行开头的EOF)。除了EOF情况，
              行分隔符包含在read(2)返回的
              缓冲区中。

           •  行编辑已启用(ERASE、KILL；以及如果IEXTEN标志已
              设置：WERASE、REPRINT、LNEXT)。read(2)最多返回一行
              输入；如果read(2)请求的字节数少于
              当前行输入中可用的字节数，则只读取
              请求的字节数，剩余字符将
              在后续read(2)中可用。

           •  最大行长度为4096个字符(包括
              终止换行字符)；超过4096个字符的行
              被截断。4095个字符后，输入处理(例如，
              ISIG和ECHO*处理)继续，但4095个字符之后
              直到(但不包括)任何终止
              换行的任何输入数据都被丢弃。这确保终端可以
              始终接收更多输入，直到至少可以读取一行。

           在非规范模式下，输入立即可用(无需
           用户键入行分隔符字符)，不执行输入
           处理，行编辑被禁用。read
           缓冲区只能接受4095个字符；如果输入模式切换到
           规范模式，这提供了换行字符所需的
           空间。MIN (c_cc[VMIN])和TIME
           (c_cc[VTIME])的设置确定read(2)
           完成的情况；有四种不同情况:

           MIN == 0, TIME == 0 (轮询读取)
                  如果有数据可用，read(2)立即返回，返回
                  可用字节数和请求字节数中的较小值。
                  如果没有数据可用，read(2)返回
                  0。

           MIN > 0, TIME == 0 (阻塞读取)
                  read(2)阻塞直到MIN字节可用，并返回
                  最多请求的字节数。

           MIN == 0, TIME > 0 (带超时读取)
                  TIME指定计时器的限制，以十分之一秒为单位。
                  调用read(2)时启动计时器。read(2)
                  在至少一个字节的数据可用或
                  计时器到期时返回。如果计时器在
                  任何输入变得可用之前到期，read(2)返回0。如果数据
                  在调用read(2)时已可用，
                  调用行为如同数据在调用后
                  立即接收。

           MIN > 0, TIME > 0 (带字节间超时读取)
                  TIME指定计时器的限制，以十分之一秒为单位。
                  一旦初始输入字节可用，计时器
                  在接收到每个后续字节后重新启动。read(2)
                  在满足以下任一条件时返回:

                  •  已接收MIN字节。

                  •  字节间计时器到期。

                  •  已接收read(2)请求的字节数。
                     (POSIX未指定此终止
                     条件，在某些其他实现中read(2)
                     在这种情况下不返回。)

                  由于计时器仅在初始字节
                  可用后启动，至少会读取一个字节。如果数据
                  在调用read(2)时已可用，
                  调用行为如同数据在调用后
                  立即接收。

           POSIX未指定O_NONBLOCK文件
           状态标志的设置是否优先于MIN和TIME设置。如果
           设置了O_NONBLOCK，在非规范模式下的read(2)可能会返回
           立即，无论MIN或TIME的设置如何。
           此外，如果没有数据可用，POSIX允许在
           非规范模式下read(2)返回0，或-1且errno设置为
           EAGAIN。

       原始模式
           cfmakeraw()将终端设置为类似于
           旧版本7终端驱动程序的"原始"模式：输入按字符
           提供，回显被禁用，所有特殊处理的
           终端输入和输出字符被禁用。终端
           属性设置如下:

               termios_p->c_iflag &= ~(IGNBRK | BRKINT | PARMRK | ISTRIP
                               | INLCR | IGNCR | ICRNL | IXON);
               termios_p->c_oflag &= ~OPOST;
               termios_p->c_lflag &= ~(ECHO | ECHONL | ICANON | ISIG | IEXTEN);
               termios_p->c_cflag &= ~(CSIZE | PARENB);
               termios_p->c_cflag |= CS8;

       行控制
           tcsendbreak()在特定持续时间内传输连续的零值位流，
           如果终端使用异步
           串行数据传输。如果duration为零，则传输零值位
           至少0.25秒，不超过0.5秒。
           如果duration非零，则发送零值位
           一段时间，由实现定义。

           如果终端不使用异步串行数据
           传输，tcsendbreak()不采取任何操作直接返回。

           tcdrain()等待直到写入fd引用对象的所有输出
           已传输。

           tcflush()丢弃写入fd引用对象但未传输的数据，
           或已接收但未读取的数据，取决于
           queue_selector的值:

           TCIFLUSH
                  刷新已接收但未读取的数据。

           TCOFLUSH
                  刷新已写入但未传输的数据。

           TCIOFLUSH
                  刷新已接收但未读取的数据和已写入
                  但未传输的数据。

           tcflow()根据action的值暂停或恢复fd引用对象上的数据
           传输或接收:

           TCOOFF 暂停输出。

           TCOON  重新启动暂停的输出。

           TCIOFF 传输STOP字符，停止终端设备
                  向系统传输数据。

           TCION  传输START字符，启动终端设备
                  向系统传输数据。

           打开终端文件的默认设置是输入
           和输出都不暂停。

       行速度
           提供波特率函数用于获取和设置termios结构中
           输入和输出波特率的
           值。新值直到成功调用tcsetattr()才
           生效。

           将速度设置为B0指示调制解调器"挂断"。
           与B38400对应的
           实际比特率可以通过setserial(8)改变。

           输入和输出波特率存储在termios
           结构中。

           cfgetospeed()返回存储在termios_p指向的termios
           结构中的输出波特率。

           cfsetospeed()将termios_p指向的termios结构中存储的
           输出波特率设置为speed，speed必须是
           以下常量之一:

                  B0
                  B50
                  B75
                  B110
                  B134
                  B150
                  B200
                  B300
                  B600
                  B1200
                  B1800
                  B2400
                  B4800
                  B9600
                  B19200
                  B38400
                  B57600
                  B115200
                  B230400
                  B460800
                  B500000
                  B576000
                  B921600
                  B1000000
                  B1152000
                  B1500000
                  B2000000

           SPARC架构额外支持这些常量:

                  B76800
                  B153600
                  B307200
                  B614400

           非SPARC架构额外支持这些常量:

                  B2500000
                  B3000000
                  B3500000
                  B4000000

           由于架构之间的差异，可移植应用程序
           在使用特定Bnnn常量之前应检查是否已定义。

           零波特率B0用于终止连接。如果
           指定了B0，则不再
           断言调制解调器控制线。通常，这会断开线路。CBAUDEX是
           超出POSIX.1定义的速度掩码(57600及以上)。
           因此，B57600 & CBAUDEX非零。

           可以通过TCSETS2 ioctl将波特率设置为Bnnn常量定义以外的值；
           参见ioctl_tty(2)。

           cfgetispeed()返回存储在termios结构中的
           输入波特率。

           cfsetispeed()将termios结构中存储的输入波特率
           设置为speed，speed必须指定为上述cfsetospeed()列出的Bnnn
           常量之一。如果输入波特率
           设置为文字常量0(不是符号常量B0)，
           输入波特率将等于输出波特率。

           cfsetspeed()是4.4BSD扩展。它采用与cfsetispeed()相同的参数，
           并设置输入和输出速度。

## [](#RETURN_VALUE){#RETURN_VALUE}返回值         [[top]{.top-link}](#top_of_page)

           cfgetispeed()返回存储在termios结构中的
           输入波特率。

           cfgetospeed()返回存储在termios结构中的
           输出波特率。

           所有其他函数返回:

           0      成功。

           -1     失败并设置errno以指示错误。

           注意，如果任何请求的
           更改可以成功执行，tcsetattr()返回成功。
           因此，在进行
           多项更改时，可能需要在此调用后进一步调用tcgetattr()以检查所有更改是否已
           成功执行。

## [](#ATTRIBUTES){#ATTRIBUTES}属性         [[top]{.top-link}](#top_of_page)

           有关本节使用的术语的说明，参见
           attributes(7)。
           ┌──────────────────────────────────────┬───────────────┬─────────┐
           │ 接口                                 │ 属性          │ 值      │
           ├──────────────────────────────────────┼───────────────┼─────────┤
           │ tcgetattr(), tcsetattr(), tcdrain(), │ 线程安全      │ MT-Safe │
           │ tcflush(), tcflow(), tcsendbreak(),  │               │         │
           │ cfmakeraw(), cfgetispeed(),          │               │         │
           │ cfgetospeed(), cfsetispeed(),        │               │         │
           │ cfsetospeed(), cfsetspeed()          │               │         │
           └──────────────────────────────────────┴───────────────┴─────────┘

## [](#STANDARDS){#STANDARDS}标准         [[top]{.top-link}](#top_of_page)

           tcgetattr()
           tcsetattr()
           tcsendbreak()
           tcdrain()
           tcflush()
           tcflow()
           cfgetispeed()
           cfgetospeed()
           cfsetispeed()
           cfsetospeed()
                  POSIX.1-2008。

           cfmakeraw()
           cfsetspeed()
                  BSD。

## [](#HISTORY){#HISTORY}历史         [[top]{.top-link}](#top_of_page)

           tcgetattr()
           tcsetattr()
           tcsendbreak()
           tcdrain()
           tcflush()
           tcflow()
           cfgetispeed()
           cfgetospeed()
           cfsetispeed()
           cfsetospeed()
                  POSIX.1-2001。

           cfmakeraw()
           cfsetspeed()
                  BSD。

## [](#NOTES){#NOTES}注意事项         [[top]{.top-link}](#top_of_page)

           UNIX V7和几个后续系统有一个波特率列表，
           在B0到B9600的值之后，可以找到两个常量
           EXTA、EXTB("外部A"和"外部B")。许多系统扩展
           列表为更高的波特率。

           tcsendbreak()中非零持续时间的效果各不相同。SunOS
           指定持续时间为N秒的中断，其中N至少为
           0.25，不超过0.5。Linux、AIX、DU、Tru64发送持续时间
           毫秒的中断。FreeBSD、NetBSD、HP-UX和MacOS
           忽略持续时间的值。在Solaris和UnixWare下，
           tcsendbreak()的非零持续时间行为类似于tcdrain()。

## [](#BUGS){#BUGS}错误         [[top]{.top-link}](#top_of_page)

           在Linux 4.16之前(和glibc 2.28之前)的Alpha架构上，
           XTABS值与TAB3不同，因此被终端驱动程序的
           N_TTY线路规程代码忽略
           (因为它不是TABDLY掩码的一部分)。

## [](#SEE_ALSO){#SEE_ALSO}参见         [[top]{.top-link}](#top_of_page)

           reset(1), setterm(1), stty(1), tput(1), tset(1), tty(1),
           ioctl_console(2), ioctl_tty(2), cc_t(3type), speed_t(3type),
           tcflag_t(3type), setserial(8)

## [](#COLOPHON){#COLOPHON}版权信息         [[top]{.top-link}](#top_of_page)

           本页面是man-pages(Linux内核和C库
           用户空间接口文档)项目的一部分。有关
           项目的详细信息可以在
           ⟨https://www.kernel.org/doc/man-pages/⟩找到。如果您有
           此手册页的错误报告，请参见
           ⟨https://git.kernel.org/pub/scm/docs/man-pages/man-pages.git/tree/CONTRIBUTING⟩。
           本页面来自从
           ⟨https://mirrors.edge.kernel.org/pub/linux/docs/man-pages/⟩获取的
           tarball man-pages-6.15.tar.gz，获取日期为
           2025-08-11。如果您在此页面的HTML版本中发现任何
           渲染问题，或者您认为有更好的或更新的
           页面来源，或者您对此版权信息中的信息有更正或
           改进(不属于原始手册页的一部分)，请发送邮件至
           man-pages@man7.org

    Linux man-pages 6.15            2025-05-17                     termios(3)

------------------------------------------------------------------------

Pages that refer to this page: [\_exit(2)](../man2/_exit.2.html), 
[FIONREAD(2const)](../man2/FIONREAD.2const.html), 
[ioctl_console(2)](../man2/ioctl_console.2.html), 
[ioctl_tty(2)](../man2/ioctl_tty.2.html), 
[setpgid(2)](../man2/setpgid.2.html), 
[TCSETS(2const)](../man2/TCSETS.2const.html), 
[TCXONC(2const)](../man2/TCXONC.2const.html), 
[cc_t(3type)](../man3/cc_t.3type.html), 
[curs_inopts(3x)](../man3/curs_inopts.3x.html), 
[getpass(3)](../man3/getpass.3.html), 
[stdin(3)](../man3/stdin.3.html),  [tty(4)](../man4/tty.4.html), 
[attributes(7)](../man7/attributes.7.html), 
[credentials(7)](../man7/credentials.7.html), 
[pty(7)](../man7/pty.7.html), 
[signal-safety(7)](../man7/signal-safety.7.html), 
[termio(7)](../man7/termio.7.html),  [agetty(8)](../man8/agetty.8.html)

------------------------------------------------------------------------

------------------------------------------------------------------------

::: footer
+-----------------------+-----------------------+-----------------------+
| HTML rendering        |                       | [![Cover of           |
| created 2025-09-06 by |                       | TLP                   |
| [Michael              |                       | I](https://man7.org/t |
| Kerrisk](https://man7 |                       | lpi/cover/TLPI-front- |
| .org/mtk/index.html), |                       | cover-vsmall.png)](ht |
| author of [*The Linux |                       | tps://man7.org/tlpi/) |
| Programming           |                       |                       |
| Interface*](htt       |                       |                       |
| ps://man7.org/tlpi/). |                       |                       |
|                       |                       |                       |
| For details of        |                       |                       |
| in-depth **Linux/UNIX |                       |                       |
| system programming    |                       |                       |
| training courses**    |                       |                       |
| that I teach, look    |                       |                       |
| [here](https:/        |                       |                       |
| /man7.org/training/). |                       |                       |
|                       |                       |                       |
| Hosting by [jambit    |                       |                       |
| Gm                    |                       |                       |
| bH](https://www.jambi |                       |                       |
| t.com/index_en.html). |                       |                       |
+-----------------------+-----------------------+-----------------------+
:::

------------------------------------------------------------------------

::: statcounter
[![Web Analytics Made Easy -
StatCounter](https://c.statcounter.com/7422636/0/9b6714ff/1/){.statcounter}](https://statcounter.com/ "Web Analytics
Made Easy - StatCounter"){target="_blank"}
:::
