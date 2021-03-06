## Linux文件系统

`EXT2`作为最传统的磁盘文件系统，所以要了解 Linux 的文件系统，就要先由 EXT2 开始，而文件系统是建立在磁盘上面的，因此我们得了解磁盘的物理组成才行

## 磁盘组成与分区的复习

### 磁盘组成

磁盘的组成主要有：

- 圆形的磁盘（主要记录数据的部分）
- 机械手臂，与机械手臂上的磁盘读取头（可擦写磁盘上的数据）
- 主轴马达，可以转动磁盘，让机械手臂的读取头在磁盘上读写数据

数据存储与读取的重点在于磁盘，那么磁盘上的物理组成则为（假设为单盘片）：

- 扇区（Sector）为最小的物理存储单位，且依据磁盘设计的不同，目前主要有512bytes与4k两种格式
- 将扇区组成一个圆，那就是磁柱（Cylinder）
- 早期的分区主要以磁柱为最小单位，现在的分区通常使用扇区为最小分区单位（每个扇区都有其号码）
- 磁盘分区表主要有两种格式：

  - MBR 分区表：限制较多
  - GPT 分区表：新的限制较少
- MBR 分区表中，第一个扇区很重要，里面主要有：

  1. 主要开机区（Master boot record ，MBR），占用 446bytes
  2. 分区表（partition table），占用 64bytes

- GPT 分区表除了分区数量扩充较多外，支持的磁盘容量也可以超过2TB

磁盘文件名部分，所有实体磁盘的文件名都已经被模拟成 `/dev/sd[a-p]` 的格式，
磁盘文件名为 /dev/sda。而分区槽则为 `/dev/sda[1-128]`（以第一颗磁盘为例）。

虚拟持平通常为  `/dev/vd[a-p]` 格式。如有使用到软件磁盘阵列的话，还有 `/dev/md[0-128]` 的磁盘文件名。使用的是 LVM 时，则为 `/dev/VGNAME/LVNAME` 等格式。

关于磁盘阵列与 LVM 后面会继续介绍，这里主要介绍以实体磁盘及虚拟磁盘为主。

- `/dev/sd/[a-p][1-128]`：为实体磁盘的磁盘文件名
- `/dev/vd/[a-d][1-128]`：为虚拟磁盘的磁盘文件名

### 磁盘分区

GPT 分区表支持大容量的磁盘，小磁盘默认会使用 MBR 的分区，可以使用配置强制使用 GPT 分区表

## 文件系统特性

为什么在分区完成之后需要格式化（format）才能够使用这个文件系统？因为每种操作系统所设定的文件属性、权限不同，为了存放这些文件所需的数据，因此就需要将分区槽进行格式化成操作系统能够利用的「文件系统格式」，windows 使用 FAT16，包括现在的 NTFS 文件系统，Linux 的正统文件系统则为 Ext2（Linux second extended file system，EXT2fs），在默认情况下，windows 操作系统不会认识 linux 的 ext2 的

传统的磁盘与文件系统之应用中，一个分区槽只能被格式化为一个文件系统，所以我们可以称之为说一个 filesystem 就是一个 partition。新技术的出现，如 LVM 与 软件磁盘阵列（software raid），这些技术可以将一个分区槽格式化为多个文件系统（如 LVM），也可以将多个分区槽合并成一个文件系统（LVM、RAID）。
所以目前在格式化时已经不再说成针对 partition 来格式化了，通常我们可以称呼一个可被挂载的数据为一个文件系统而不是一个分区槽。

那么文件系统是如何运行的呢？这个和实际的文件数据有关。较新的操作系统**文件数据除了文件实际内容以外，通常还有各种其他的属性，例如Linux操作系统的`文件权限（rwx）`和`文件属性（拥有者、群组、时间参数等）`**。并且在文件系统中，通常会将这两部分内容放到不同的区块中：

- inode
  - 权限与属性，一个文件占用一个inode，同时会记录此文件的数据所在的block号码

- data block
  * 用于记录真实的数据内容，若文件太大的时候， 会占用多个block

- super block

  - 存放整个文件系统的整体信息，包括inode和block的总量、使用量、剩余量，以及文件系统的格式与相关信息等

    

![inode-block的联系](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211029163342074.png)

同时以上也是**索引式文件系统**，与此同时还有另外一种的存储形式（FAT格式）

![FAT存储格式](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211029163929212.png)

**碎片整理：**

> 主要原因就是文件写入的block过于离散化，然后就会导致读取的性能较低，对于FAT文件系统经常需要碎片整理，对于Ext2/Ext3的文件系统来说，就不需要进行碎片整理了

## Linux 的 Ext2 文件系统（inode）

ext2 就是使用这种 inode 为基础的文件系统，并且文件系统一开始就将 inode 与 block 规划好了，除非重新格式化（或则利用 resize2fs 等指令变更文件系统大小），否则 inode 与 block 固定后就不再变动。当文件系统数据高达数百 GB 时，将所有的 inode 与 block 通通放置在一起很不理智，而且这么多数量的 inode 与 blcok ，不太统一管理。因此 ext2 文件系统在格式化的时候，基本上是分区为多个区块组（block group）的，每个区块群组都有独立的 inode、block、superblock 系统。这样分成一群一群的比较好管理，整个来说 ext2 格式化后有点像下图这样

![image-20211031004158244](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031004158244.png)

在整体规划中，**文件系统最前面有一个启动扇区（boot sector）**，这个启动扇区可以安装开机管理程序，这是个非常重要的设计，因为能将不同的开机管理程序安装到个别的文件系统最前端，而不用覆盖整颗磁盘唯一的 MBR，正因为这样才能够制作出多重引导环境每个区块群组（block group）的 6 个主要内容如下

### data block 资料区块

data block 是用来存放文件内容的地方，EXT2 中所支持的 block 大小有 1k、2k 及 3k 三种。在格式化时 block 的大小就固定了，并都有编号，方便 inode 的记录。由于 block 大小的差异，会导致该文件系统能够支持的最大磁盘容量与最大单一文件容量并不相同。限制如下：

| Block 大小         | 1 KB  | 2KB    | 4kb   |
| ------------------ | ----- | ------ | ----- |
| 最大单一文件限制   | 16 GB | 256 GB | 2TB   |
| 最大文件系统总容量 | 2 TB  | 8 TB   | 16 TB |

虽然 ext2 已经能够支持大于 2GB 以上的单一文件容量，有些应用程序依然使用旧的限制，无法读取超过 2GB 的文件

block 的基本限制如下：

- 原则上，block 的大小与数理在格式化完成就不能够再改变了（除非重新格式化）
- 每个 block 内最多只能放置一个文件的数据
- 如果文件大于 block 的大小，则一个文件会占用多个 block 数量
- 若文件小于 block ，则该 block 的剩余容量就不能够再被使用（磁盘空间会浪费）

原理如上，那么假设你的 ext2 文件系统使用 4k block，有 10000 个小文件（均为 50 bytes），此时硬盘浪费多少容量？

```
一个 block 只能存储一个文件，每个 block 会浪费：4096 - 50 = 4046 byte
所有文件总量为：50 bytes * 10000 = 488.3 kbytes
此时浪费容量为：4046 bytes * 10000 = 38.6 MBytes

总共不到 1 MB 的总文件容量却浪费近 40 MB 的容量
```

在什么场景下会出现以上所说的问题？比如在 BBS 网站中的数据，使用纯文本记录每篇留言，当留言内容都都很少时，就会产生很多的小文件（留言越多产生小文件越多）那么将 block 设置为 1k ，可能也不妥当，因为大型文件会占用数量更多的 block，而 inode 也需要记录更多的 block 号码，此时将可能导致文件系统不良的读写效能所以在进行文件系统的格式化时，需要按你的使用场景来预计使用情况，如基本上都是几百兆的文件，那么就选择 4k 的（目前硬盘容量都很大了，所以一般都会选择 4k，而不管场景了）

### inode table

inode 记录文件的属性和实际数据的 block 号码，基本上记录的文件信息至少有以下：

- 该文件存取模式（read、write、excute）
- 文件拥有者与群组（owner、group）
- 文件的容量
- 文件建立或状态改变实际（ctime）
- 最近一次的读取实际（atime）
- 最近修改的时间（mtime）
- 定义文件特性的旗标（flag），如 SetUID 等
- 该文件真正内容的指向（pointer）

inode 的数量与大小在格式化时以及固定，还有以下特点：

- 每个 inode 大小均固定为 128 bytes（新的 ext4 与 xfs 可设定到 256 bytes
- 每个文件仅会占用一个 inode
- 因此文件系统能建立的文件数量与 inode 的数量有关
- 系统读取文件时需要先找到 inode，并分析 inode 所记录的权限与用户是否符合，符合才会读取 block 的内容

下面简略分析 ext2 的 inode、block 与文件大小的关系。

> inode 记录的数据非常多，但是仅 128 bytes，记录一个 block 号码花掉 4 byte；
> 假设有一个文件有 400 MB 且米格 block 为 4k 时，至少需要 10 万笔 block 号码要记录，但是 inode 的 128 byte 怎么能够记录下这么多的号码？

系统将 inode 记录 block 号码的区域定义为 12 个直接、一个间接、一个双间接、一个三间接记录区，如下图所示

![image-20211031003941260](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211031003941260.png)

- 直接：该区域内直接存取 block 号码
- 间接：该区域内记录了一个 block 号码，该 block 才是记录文件内容的 block 号码
- 双间接：当文件太大时，在第二层中来记录 block 号码
- 三间接：当文件更大时使用，在第三层中记录数据内容的 block 号码

这样子的 inode 能够指定多少个 block 呢？以 1k block 来说明：

- 12 个直接指向：12 * 1k = 12k

  总共可以记录 12 笔记录，总额为 12k

- 间接：256 * 1k = 256k

  每个 block 号码需要 4 byte 来记录，因此 1k 的大小能够记录 256 个。

- 双间接：256 * 256 * 1k = 256 的 2 次方

  第一层 block 会指定 256 个第二层，每个第二层可以指定 256 个号码

- 三间接：256 * 256 * 256 * 1k = 256 的 3 次方

- 总额：直接 + 间接 + 双间接 + 三间接

  12 + 256 + 256 * 256 + 256 * 256 * 256 = 16 GB

在 ext2 中，当 block 格式化为 1k 大小时，能够容量的最大单文件为 16 GB,在前面的文件系统限制表总的说明大小一致！但是该方法不能用在 2k 以及 4k block 大小的计算中，因为大于 2k 的 block 将会受到 ext2 文件系本身的限制？？？？

所以新系统能使用 ext4 还是使用最新的文件系统，ext4 的 inode 容量扩大到 256 bytes 了，可以记录更多的文件系统信息，包括新的 acl 以及 SELinux 类型等，单一文件容量高达 16 TB 且单一文件系统总容量可达 1 EB

### Superblock 超级区块

superblock 是记录真个 filesystem 相关信息的地方，没有 superblock 就没有这个 filesystem 了，记录的主要信息有：

- block 与 inode 的总量
- 未使用与已使用的 inode 、block 数量
- block 与 inode （block 1、2、4k，inode 为 128 、256 bytes）
- filesystem 的挂载时间、最近一次写入时间的时间、最近一次检验磁盘（fsck）的时间等
- 一个 valid bit 数值，若此文件系统已被挂载，则 valid bit 为 0，否则为 1

superblock 的大小为 1024 bytes，它非常重要，这个文件系统的基本信息都写在这里，如果 superblock 挂掉，那么可能需要花费很多时间去挽救。后续使用 dumpe2fs 指令来观察此外，每个 block group 都可能含有 superblock ，一个文件系统只应该有一个 superblock，多出来的只是备份（这样才可以有救援的机会）；第一个 block group 内会含有 superblock 之外，
后续的 block group 不一定含有 superblock，如果含有则是作为第一个 block group 内
superblock 的备份

### Filesystem Description 文件系统描述说明

该区段可以描述每个 block group 的开始与结束的 block 号码，以及说明每个区段（superblock、bitmap、inodemap、data block）分别介于哪一个 block 号码之间。这部分也可以使用 dumpe2fs 指令来观察

### block bitmap 区块对照表

新增文件时会用到 block ，如何选择到一个空的 block 来记录文件数据，就是通过 block bitmap 来知道的。同样，删除文件时，原本占用的 block 号码需要释放，bitmap 中对应的标志就需要修改

### inode bitmap （inode 对照表）

与 block bitmap 类似，记录 inode 的使用情况

### dumpe2fs 查询 ext 家族 superblock 信息的指令

由于目前 centos7 使用了 xfs 为预设文件系统，所以本次学习无法进行试验，dumpe2fs 只支持 ext 家族信息查询。后续讲过格式化内容之后，就可以自己切除一个 ext4 的文件系统来实践这里的指令这里的 ext 文件系统为 1GB 容量，使用默认方式进行格式化，观察内容如下：

```bash
dumpe2fs [-bh] 装置文件名
```

选项参数：

- b：列出保留为坏轨的部分
- h：仅列出 superblock 的数据，不会列出其他区段的内容



