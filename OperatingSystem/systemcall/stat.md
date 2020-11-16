# 文件属性

## stat 结构体

```cpp
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>

struct stat {
    dev_t     st_dev;     //  文件的设备编号
    ino_t     st_ino;     //  节点
    mode_t    st_mode;    //  文件的类型和存取的权限
    nlink_t   st_nlink;   //  连到该文件的硬连接数目，刚建立的文件值为 1
    uid_t     st_uid;     //  用户 ID
    gid_t     st_gid;     //  组 ID
    dev_t     st_rdev;    //  若此文件为设备文件，则为其设备编号
    off_t     st_size;    //  文件字节数
    blksize_t st_blksize; //  块大小
    blkcnt_t  st_blocks;  //  块数

    struct timespec st_atim;  // 最后一次访问时间
    struct timespec st_mtim;  // 最后一次修改时间
    struct timespec st_ctim;  // 属性最后一次改变时间

    #define st_atime st_atim.tv_sec      /* Backward compatibility */
    #define st_mtime st_mtim.tv_sec
    #define st_ctime st_ctim.tv_sec
};

// st_mode 用特征位来表示文件类型以及操作权限
// 定义了下面几种通过 st_mode 判断文件类型的宏
S_ISREG(m)  /* is it a regular file? -普通文件 */
S_ISDIR(m)  /* directory? -目录文件? */
S_ISCHR(m)  /* character device? -字符设备文件? */
S_ISBLK(m)  /* block device? -块设备文件? */
S_ISFIFO(m) /* FIFO (named pipe)? -管道文件? */
S_ISLNK(m)  /* symbolic link? (Not in POSIX.1-1996.) -符号链接? */
S_ISSOCK(m) /* socket? (Not in POSIX.1-1996.) -套接口? */
```

## stat

```cpp
int stat(const char *path, struct stat *buf);

// 通过文件名 filename 获取文件信息，并保存在 buf 所指的结构体 stat 中
// 执行成功则返回 0，失败返回 -1，错误代码存于 errno
```

## fstat、lstat

```cpp
int fstat(int filedes, struct stat *buf);
// fstat 接受的是一个文件描述符，需要使用 open 才能得到

int lstat(const char *path, struct stat *buf);
// 当文件是一个符号链接时，lstat 返回的是该符号链接本身的信息
// stat 返回的是该链接指向的文件的信息
```