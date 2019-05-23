# Linux文件系统

## 文件系统运行原理

### Linux文件存储
1. inode：一个文件一个inode，记录存储数据的datablock序号
2. datablock:实际存数数据单元
3. superblock：记录文件系统的整体信息包括inode和datablock使用量等情况

***
### LinuxEXT2的blockgroup
1. datablock:存储数据
2. inodetable:
    + 该文件的存取模式(read/write/excute)
    + 该文件的拥有者与群组(owner/group)
    + 该文件的容量
    + 该文件创建或状态改变的时间(ctime)
    + 最近一次的读取时间(atime)
    + 最近修改的时间(mtime)
    + 定义文件特性的旗标(flag)，如SetUID...
    + 该文件真正内容的指向(pointer)
3. superblock:存储文件系统信息
    + block与inode的总量;
    + 未使用与已使用的inode/block数量;
    + block与inode的大小(block为1,2,4K，inode为128Bytes或256Bytes);
    + filesystem的挂载时间、最近一次写入数据的时间、最近一次检验磁盘(fsck)的时间等文件系统的相关信息;
    + 一个validbit数值，若此文件系统已被挂载，则validbit为0，若未被挂载，则validbit为1。
4. Filesystem Description:描述data block的起始编号
5. block bitmap:判断block 是否为空
6. inode bitmap:判断inode是否使用
7. dumpe2fs:查询Ext superblock信息的指令 `dumpe2fs /dev/vda5`

***
### Linux磁盘与目录
VFS(虚拟文件系统):用来管理不同文件系统的集合

1. df
    ```
    -a:列出所有的文件系统，包括系统特有的 /proc 等文件系统
    -k:以 KBytes 的容量显示各文件系统
    -m:以 MBytes 的容量显示各文件系统
    -h:以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示
    -H:以 M=1000K 取代 M=1024K 的进位方式
    -T:连同该 partition 的 filesystem 名称 （例如 xfs） 也列出
    -i:不用磁盘容量，而以 inode 的数量来显示
    ```
2. du
    ```
    -a:列出所有的文件与目录容量，因为默认仅统计目录下面的文件量而已
    -h:以人们较易读的容量格式 （G/M） 显示
    -s:列出总量而已，而不列出每个各别的目录占用容量
    -S:不包括子目录下的总计，与 -s 有点差别
    -k:以 KBytes 列出容量显示
    -m:以 MBytes 列出容量显示
    ```

### 链接

>文件名是仅仅与文件目录有关的

1. 软链接：类似快捷方式
2. 硬连接：通过inode进行操作，实际上一个文件

>硬连接要求:不能跨文件系统，不能link目录

### 磁盘操作
#### 磁盘信息查询
1. lsblk： 列出磁盘设备
2. blkid: 列出设备UUID等信息
3. parted : 磁盘分区信息
#### 磁盘分区
1. gdisk: GPT分区
2. fdisk: MBR分区
