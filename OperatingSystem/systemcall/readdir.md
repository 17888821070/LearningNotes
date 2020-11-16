# 文件夹信息

## dirent

```cpp
#include <dirent.h>
#include <sys/types.h>

struct dirent {
    ino_t          d_ino;       /* 索引节点号*/
    off_t          d_off;       /* 在目录文件中的偏移*/
    unsigned short d_reclen;    /* 文件名长*/
    unsigned char  d_type;      /* 文件类型*/              
    char           d_name[256]; /* 文件名，最长 255 字符*/
};
```

## opendir

打开文件夹，获取指针

```cpp
DIR *opendir( const char *name);

DIR *fdopendir( int fd);
```

## readdir

```cpp
struct dirent *readdir(DIR *dirp);
// 使用 readdir 后，readdir 会读到下一个文件
// readdir 是依次读出目录中的所有文件，每次只能读一个

int readdir_r(DIR *dirp, struct dirent *entry, struct dirent **result);
```