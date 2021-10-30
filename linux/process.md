进程作为面试中最为常见的知识点，经常会被问道什么是进程、进程在内存中的分布、进程的各种API、如何排查一些CPU占用过高的进程等。

## 一些基础概念

**程序：**

> 通常为binary programm，放置在存储媒体中（如硬盘、光盘、软盘、磁带等），为实体文件的形态存在

**进程：**

> 程序的一次执行过程，如果我们将程序视为状态机，那么进程就可以理解为状态机上的路径。进程的管理就可以理解为状态机创建、重置、删除。同时操作系统代码借助中断等机制完成进程切换
>
> 经典的概述为：进程是具有独立功能的程序在一个数据集合上运行的过程

## 进程的状态管理

对于进程的状态管理，常见的系统调用为：`fork`、`execve`、`exit`，分别用于进程的`创建`、`改变`、`删除`

### fork

用于创建一个新的进程，新创建的进程返回0，执行fork的进程返回进程号

**经典操作之fork bomb**

```shell
:(){ :|:& };:
## 格式化
fork() {
	fork | fork &
};fork
```

 因为进程是通过复制进行创建的，所以我们总是能够找到**父子关系**

![image-20211031104146432](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031104146432.png)



### execve()

fork用于进程的创建，但是我们还需要执行创建的进程，这就是execve的功能。其概念为：

> 将当前运行的状态机重置成为另一个程序的初始状态

![execve](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031104605165.png)

![image-20211031104653265](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031104653265.png)



### exit()

有了fork、execve我们就可以只有执行任何的进程了，但是还缺少一个进程的销毁操作---exit()

> 立即销毁进程

**exit的几种写法：**

- exit(0) - stdlib.h中声明的libc函数
  - 会调用atexit
- _exit(0) - glibc的系统调用syscall wrapper
  - 执行”exit_group“的系统调用终止整个进程
  - 不会调用atexit
- syscall(SYS_exit, 0)
  - 执行”exit"系统调用终止当前线程
  - 不会调用atexit

![exit](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031105526526.png)



## 进程管理

我们通过静态的ps和动态的top以及pstree都能够查看系统中正在运行的进程

![image-20211031111217626](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031111217626.png)

### 仅查看自己的bash相关进程

![image-20211031125103545](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031125103545.png)

- F：进程旗标（process flags），说明这个进程的总结权限，常见的号码有：
  - 4：表示此进程的权限为 root
  - 1：则表示此子进程仅进行 **复制（fork）而没有实际执行（exec）**
- S：进程状态（STAT），主要状态有：
  - R（Running）：正在运行中
  - S（Sleep）：该程序目前正在睡眠状态（idle），但可以被唤醒（signal）
  - D：不可被唤醒的睡眠状态，通常该程序可能在等待 I/O 的情况
  - T：停止状态（stop），可能是在工作控制（背景暂停）或除错（traced）状态
  - Z（Zombie）：僵尸状态，进程已终止但却无法被移除至内存外
- UUID/PID/PPID：代表此进程被该 UID 所拥有、进程的 PID 、此进程的父进程 PID
- C：代表 CPU 使用率，单位为百分比
- PRI/NI：Priority/Nice 的缩写，代表此进程被 CPU 所执行的优先级，数值越小表示该进程越快被 CPU 执行。详细的 PRI 与 NI 将在下一小节讲解
- ADDR/SZ/WCHAN：都与内存有关
  - ADDR：kernel function，该进程在内存的哪个部分，如果是 running 的进程，一般会显示 `-`
  - SZ：该进程用掉多少内存
  - WCHAN 该进程是否运行中，若为 `-` 表示正在运行中
- TTY：登陆者的终端机位置，若为远程登录则使用动态终端接口（pts/n）
- TIME：使用掉的 CPU 时间。注意：是此进程实际花费 CPU 运行的时间
- CMD：command 的缩写，此进程的触发程序指令

### 列出类似进程树的进程显示

![image-20211031125253358](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031125253358.png)

### 动态观察进程的变化

ps 可以显示一个时间点的进程状态，而 top 则可以持续的侦测进程运行状态

```shell
top [-d 数字] | top [-bnp]

选项与参数：
	-d：后面可以接秒数，整个进程画面更新的秒数，预设是 5 秒更新一次
	-b：以批次的方式执行 top，还有更多的参数可以使用（莫名其妙啊，啥参数？），通常会搭配数据流重导向来将批次的结果输出为文件
	-n：与 -b 搭配，需要进行几次 top 的输出
	-p：指定某些 PID 来进行观察

在 top 执行过程中可以使用的按键指令：
	？：显示在 top 中可以输入的按键指令
	P：以 CPU 的使用资源排序显示
	M：以 Memory 的使用资源排序显示
	N：以 PID 排序
	T：由该进程使用 CPU 时间累积（TIME+）排序
	k：给予某个 PID 一个信号（signal）
	r：给予某个 PID 重新制定一个 nice 值
	q：离开 top 软件的按键
	E：切换单位显示，比如从 KB 切换为 G 显示
	c：切换 COMMAND 的信息，name/完成指令
```

![image-20211031125439328](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031125439328.png)

top 的信息基本上分为两个区域，上面 6 行，和下面的列表

- 第一行信息：top -

  - 目前开机时间：22:20:11 这个

  - 开机到目前为止所经过的时间：up 1:05 这个

  - 已经登录系统的用户人数：4 users

  - 系统在 1、5、15 分钟的平均工作负载

    在第 15 章谈到过 batch 工作方式负载小于 0.8 就是这里显示的值了。

    表示的是，系统平均要负责运行几个进程，这里是三个值，也就是对应平均 1/5/15 分钟

    越小达标系统越空闲，若高于 1 ，那么你的系统进程执行太频繁了

- 第二行：tasks

  显示的是目前进程的总量与各个状态（running、sleeping、stopped、zombie）的进程数量

  如果发现有 zombie 进程的话，就需要找下是哪个进程变成了僵尸进程了

- 第三行：`$Cpus`

  CPU 整体负载，每个项目可使用 ？ 查询。

  需要特别注意的是  wa 项，表示 I/O wait，通常系统变慢，都是 I/O 产生的问题比较大，需要特别注意该项占用的 CPU 资源，如果是多核 CPU，可以按下数字键「1」来切换成不同 CPU

- 第四行和第五行

  目前的物理内存与虚拟内存（Mem/Swap）的使用情况。要注意的是 swap 的使用量要尽量的少，如果 swap 被大量使用，表示系统的物理内存不足

- 第六行：当在 top 程序中输入指令时，显示状态的地方

### pstree

```shell
pstree [-AIU] [-up]

选项与参数：
	-A：各进程之间的连接以  ASCII 字符来连接
	-U：各进程之间的连接以万国码的字符来连接。在某些终端机接口下可能会有错误
	-p：并同时列出每个 process 的 PID
	-u：并同时列出每个 process 的所属账户名称
```

![image-20211031125647788](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031125647788.png)

### 系统资源的观察

#### free：观察内存使用情况

```shell
free [-b|-k|-m|-g|-h] [-t] [-s N -c N]

选项与参数：
	-b：单位参数；默认是用 k，其他单位对应 bytes、Mbytes、Kbytes、Gbytes
	-t: 输出的最终结果，显示物理内存与 swap 的总量
	-s：可以让系统每几秒输出一次，不间断输出；
	-c：与 -s 同时处理，让 free 列出几次
```

![free](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031130722918.png)

#### uname：查询系统与核心相关信息

```shell
uname [-asrmpi]

选项与参数：
	-a：所有系统相关的，都列出来
	-s：系统核心名称
	-r：核心的版本
	-m：本系统的硬件名称，例如 i686 或 x86_64
	-p：CPU 的类型，与 -m 类似
	-i：硬件的平台（ix86）
```

![uname](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031130646035.png)

#### uptime：观察系统启动时间与工作负载

![uptime](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031130557192.png)

### netstat：追踪网络或插槽文件

```shell
netstat -[atunlp]

选项与参数：
	-a：将目前系统上所有的联机、监听、Socket 数据都列出来
	-t：列出 tcp 网络封包的数据
	-u：列出 udp 网络封包的数据
	-n：不以进程的服务名称，以端口号来显示
	-l：列出目前正在网络监听的（listen）的服务
	-p：列出该网络服务的进程 PID
```

![netstat](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031130432355.png)

网络联机部分：

- Proto：网络封包协议，主要分为 TCP 与 UDP。
- Recv-Q：非由用户程序连接到此 socket 的复制和总 Bytes 数
- Send-Q：非由远程主机传送过来的 acknowledged 总 Bytes 数
- Local Address：本地端的 Ip:port
- Foreign Address：远程主机的 IP:port
- State：联机状态，主要有建立（ESTABLISED）、监听（LISTEN）

### dmesg：分析核心产生的信息

系统在开机的时候，核心会去侦测系统的硬件，那么硬件的检测信息由于开机过程中要么一闪而过，要么没有显示在屏幕上，可以使用 dmesg 来查看

从系统开机起，核心产生的信息都会记录到内存中，通过 dmesg 可以查询到，信息过多时可以通过 more 指令查看

![dmesg](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031130316621.png)



### vmstat：侦测系统资源变化

vmstat 可以侦测 CPU、内存、磁盘输入输出状态等信息。比如可以了解一台繁忙的系统到底是哪个环节最耗时间，可以使用 vmstat 分析看看，常见选项与参数如下：

```shell
vmstat [-a] [延迟 [总计侦测次数]]		# CPU/内存等信息
vmstat [-fs]										 # 内存相关
vmstat [-S 单位]									# 设置显示数据的单位
vmstat [-d]											 # 与磁盘有关
vmstat [-p 分区槽]								 # 与磁盘有关

选项与参数：
	-a：使用 inactive/active（是否活跃）取代 buffer/cache 的内存输出信息
	-f：开机到目前为止，系统复制（fork）的进程数
	-s：将一些事件（开机到目前为止）导致的内存变化情况列表说明
	-S：后面可以接单位，例如 k、M 等
	-d：列出磁盘的读写总量统计表
	-p：后面列出分区槽，可显示该分区槽的读写总量统计表
```

![vmstat](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031130224514.png)

## job control管理

### 直接将指令丢到后台运行

> 假设我们需要将整个/etc/进行备份为/tmp/etc.tar.gz，同时不想等待，可以这么做：

![后台进行](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031122938807.png)

### 将当前任务丢到后台运行并暂停

使用 `Ctrl + z`可以停止后台工作任务，并且通过`job -l`输出暂停的任务

![查看暂停任务](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031123310486.png)

### 查看当前在后台运行的任务状态

```shell
jobs [-lrs]

# 选项与参数：
	-l：# 除了列出 job number 与指令之外，同时列出 PID 的号码
	-r：# 仅列出正在背景 run 的工作
	-s：# 仅列出正在背景中暂停 stop 的工作
```

仔细看上面有减号和加号：

- `+`：表示最近被放到背景的工作；如果只输入 fg 指令，那么 `[3]` 会被拿到前景中来处理
- `-`：表示最近最后第二个被放置到背景中的工作。如果超过最后第三个以后的工作，就不会有 `-、+` 符号了

### 将后台运行的任务拿到前台处理

```shell
fg %jobnumber

$jobnumber: jobnumber 是工作号码（数字），哪个 % 是可有可无的
```

![image-20211031124151582](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031124151582.png)

### 将后台的任务站台变成运行中

![image-20211031124702654](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031124702654.png)

### 管理任务

通过 fg 拿到前景来，可以通过 kill 将该工作直接移除

```shell
kill -signal $jobnumber
kill -l

选项与参数：
	-l：L 的小写，列出目前 kill 能够使用的信号（signal）有哪些？
	signal：给予后续工作什么指示，用 man 7 signal 可知：
		-1：重新读取一次参数的配置文件（类似 reload）
		-2：代表与由键盘输入 ctrl+c 同样的动作
		-9：立刻强制删除一个工作
		-15：已正常的进程方式终止一项工作。与  -9 是不一样的
```



![image-20211031124804045](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031124804045.png)

### 脱机管理

当我们脱机之后，无论任务放置在后台还是前台都是无法继续运行的，因为我们整个任务的依然和当前的终端有关，一旦退出，所有除了系统服务，当前会话产生的所有任务都会停止。所以为了让任务能够在脱机后继续运行， 我们需要将对应的任务放置到系统的服务中。

**实例：**

编写一个sleep 50s的程序，并且脱机之后仍然能够正常运行

![image-20211031113651063](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031113651063.png)

![image-20211031113802508](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031113802508.png)

