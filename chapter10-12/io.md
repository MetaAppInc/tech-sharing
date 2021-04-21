# 系统级 I/O

---------

## 一、什么是 I/O

I/O（输入 / 输出）通常指数据在存储器或其他周边设备间的输入输出，是信息处理系统与外部世界之间的通信。

## 二、Unix I/O

Unix 系统中所有 I/O 设备都会被映射为文件， I/O 的输入输出都会被当作对相应文件的的读和写来执行。这使得所有的输入输出都能以一个统一的方式来执行。

* 打开文件。应用程序要求内核打开相应的文件，内核返回描述符。

* 每个进程开始都有三个打开的文件：标准输入（描述符 0）、标准输出（描述符 1）、标准错误（描述符 2）

* 改变当前文件的位置。内核保持一个相对于文件初始位置的偏移量，应用程序可以显式改变该值。

* 读写文件。读文件从文件复制字节到内存，写文件从内存复制数据至文件，并更新偏移量。

* 关闭文件。内核释放资源并回收描述符到可用描述符池。

## 三、文件

每个 linux 文件都有一个类型来表明它在系统中的角色。

* 普通文件。包含任意数据。分为文本文件和二进制文件。只包含 ASCII 和 Unicode 字符的普通文件是文本文件，其他的则为二进制文件。

* 目录。包含一组链接的文件。每个链接都将一个链接名映射到一个文件。每个目录都至少包含两个链接：到该目录自身的链接及到该目录的父级目录。

* 套接字。用于与另一个进程进行跨网络通信。

## 四、读写文件

操作系统自带一些函数以支持文件的 I/O 操作：

* `open` 打开已存在的文件或创建一个新文件，并返回相应的描述符。通过指定参数来指明进程该如何访问该文件，或给当前文件设置访问权限。

* `read` 从描述符所对应的文件中复制指定字节到内存中，返回 -1 代表错误，0 代表 EOF，否则为实际传输的字节数量。

* `write` 复制最多 n 个字节到描述符所对应的文件的当前位置。

* `lseek` 显式改变文件当前的偏移量。

* `stat` 获取文件的元数据。

* `fstat` 和 `stat` 一样，只不过 `fstat` 传入的是描述符而后者需要传入文件名。

除此之外，C 语言标准库也有一系列对应的 I/O 操作函数。标准 I/O 库将一个打开的文件模型化为一个流，流是对文件i描述符和流缓冲区的抽象。目的是使开销较高的 Linux I/O 系统调用的数量尽可能地减小。

##### 读取文件元数据

应用程序可以通过 `stat` 和 `fstat` 函数读取文件元数据，保存至 `stat` 数组中。

```c
           struct stat {
               dev_t     st_dev;         /* ID of device containing file */
               ino_t     st_ino;         /* Inode number */
               mode_t    st_mode;        /* File type and mode */
               nlink_t   st_nlink;       /* Number of hard links */
               uid_t     st_uid;         /* User ID of owner */
               gid_t     st_gid;         /* Group ID of owner */
               dev_t     st_rdev;        /* Device ID (if special file) */
               off_t     st_size;        /* Total size, in bytes */
               blksize_t st_blksize;     /* Block size for filesystem I/O */
               blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */

               /* Since Linux 2.6, the kernel supports nanosecond
                  precision for the following timestamp fields.
                  For the details before Linux 2.6, see NOTES. */

               struct timespec st_atim;  /* Time of last access */
               struct timespec st_mtim;  /* Time of last modification */
               struct timespec st_ctim;  /* Time of last status change */

           #define st_atime st_atim.tv_sec      /* Backward compatibility */
           #define st_mtime st_mtim.tv_sec
           #define st_ctime st_ctim.tv_sec
           };
```

## 五、共享文件

内核用三个相关的数据结构来表示打开的文件：

* 描述符表。每个进程有独立的描述符表，它的表项是由进程打开的文件描述符来索引的。每个打开的文件描述符表项指向*文件表*的一个表项。

* 文件表。打开文件的集合是由一张文件表来表示的。所有进程共享这张表。每个文件表表项由当前文件位置、引用计数以及一个指向 v-node 表中对应表项的指针等组成。关闭一个描述符会减少相应的文件表表项的引用计数。内核不会删除这个文件表表项，直到它的引用计数为 0。

* v-node 表。所有进程共享这张表。每个表项包含 `stat` 结构中的大多数信息，包括 `st_mode` 和 `st_size` 成员。
