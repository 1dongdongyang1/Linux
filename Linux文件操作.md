# Linux文件操作

[toc]


## 1.C语言文件接口

+ C 语言中的文件操作函数提供了对文件进行读写操作的接口。
+ 常用的文件操作函数来自于标准库 `stdio.h`，以下是一些常用的 C 语言文件操作函数及其用途：

```c
//文件的开启
FILE *fopen(const char *filename, const char *mode);

//文件的关闭
int fclose(FILE *stream);

//文件的读取
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);

//文件的写入
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
```

**计算机系统结构图示：**

<img src=".\picture\计算机系统结构.png" alt="计算机系统结构" style="zoom:45%;" />

1. 文件是归操作系统管的，**只有操作系统可以操作文件**，但是用户想操作文件怎么办？
2. 操作系统留有面向上层的接口，用户使用这些接口就可以操作文件了，这些接口是上面C语言的接口吗？
3. 不是，但是C语言的文件接口在底层实现上，是调用了操作系统的文件接口。
4. 不管是C语言，C++，还是Java，相关的文件接口底层都是调用操作系统的文件接口，因为只有操作系统可以操作文件。
5. 调用操作系统的接口，本质上还是操作系统在操作。

## 2.Linux文件接口

```c
//文件的打开
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
//虽然open函数的两个版本看起来像C++的函数重载，但在底层实现上，它并不是通过重载实现，而是通过可变参数和条件判断来实现的。

//文件的关闭
int close(int fd);

//文件的读取
ssize_t read(int fd, void *buf, size_t count);

//文件的写入
ssize_t write(int fd, const void *buf, size_t count);
```

> 相关函数的使用介绍，自行通过man手册查看。

## 3.文件描述符

+ Linux 中的文件是以**文件描述符(file descriptor)**来进行标识和操作的。

+ 文件描述符是一个整数，标识打开的文件。操作系统通过文件描述符来执行实际的文件操作。

+ `open`的返回值，`close/read/write`参数中的`fd`都是**文件描述符**。

+ Linux进程默认情况下会自带3个文件描述符，0，1，2，分别代表标准输入，标准输出，标准错误。

  > 文件描述符为什么会和输入输出扯上关系？
  >
  > 因为Linux下一切皆文件，输入输出也被看作成为文件。

**文件描述符图示：**

<img src=".\picture\文件描述符.png" alt="文件描述符" style="zoom:40%;" />

1. 每当操作系统创建一个新的进程时，都会为该进程分配一个PCB。PCB是操作系统用于管理进程的一个数据结构，存储了与进程执行和调度相关的重要信息。而Linux操作系统中，实现PCB作用的是`task_struct`结构体。
2. 当文件被使用`open`打开，为了方便管理，会形成`file`对象，暂不讨论对象里存储了什么。
3. 在`task_struct`结构体中，存储了有关文件的信息，通过指针`files`，指向了专门记录文件信息的结构体`files_struct`，在该结构体里，用结构体指针数组`fd_array[]`记录着打开的文件的`file`对象的地址，而数组的下标就是**文件描述符**。

**文件描述符的分配规则**：在`fd_array[]`数组中，找到当前没有被使用的最小的一个下标，作为新的文件描述符。

## 4.重定向

如果我们现在关闭文件描述符`1`，再打开文件`myfile`，根据文件描述符的分配规则，文件`myfile`获得的文件描述符是`1`

```cpp
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>

int main()
{
    close(1);
    int fd = open("myfile", O_WRONLY|O_CREAT, 0644);

    printf("fd: %d\n", fd);
    fflush(stdout);
    
    close(fd);
    return 0;
}
```

运行发现，本应该输出到显示器的内容，输出到了文件`myfile`上，这种现象叫作**输出重定向**。

常见的重定向有：`>`，`>>`，`<`

**重定向图示：**

![重定向](.\picture\重定向.png)

## 5.模拟实现C语言文件接口

### 4.1FILE结构体

```c
#define SIZE 1024
typedef struct FILE_ {		
  int flags;		//标记位，区分缓冲区的缓存策略
  int fileno;		//文件分配符 fd
  int cap;			//缓冲区的最大容量
  int size;			//缓冲区当前的存储数量
  char buffer[SIZE];	//缓冲区
} FILE_;	
//为和C标准库中的FILE结构体区分，将我们写的命名后都加_
```

### 4.2fopen函数

```c
//C标准库fopen的参数
FILE * fopen ( const char * filename, const char * mode );
//man手册open的参数
int open (const char* pathname, int flags);
int open (const char* pathname, int flags, mode_t mode);
```

```c
//自己实现
FILE_* fopen_(const char* pathname, const char* mode) {
    int flags = 0;				//open的标记位	
    int defaultMode = 0666;		 

    if (strcmp(mode, "r") == 0) {
        flags |= O_RDONLY;
    } else if (strcmp(mode, "w") == 0) {
        flags |= (O_WRONLY | O_TRUNC | O_CREAT);
    } else if (strcmp(mode, "a") == 0) {
        flags |= (O_WRONLY | O_APPEND | O_CREAT);
    }

    int fd = 0;

    if (flags & O_RDONLY)
        fd = open(pathname, flags);
    else
        fd = open(pathname, flags, defaultMode);
    if (fd < 0) {
        const char* err = strerror(errno);
        write(2, err, strlen(err));
        return NULL;
    }
    FILE_* fp = (FILE_*)malloc(sizeof(FILE_));
    assert(fp);

    fp->flags = SYNC_LINE;      //默认出现\n输出
    fp->fileno = fd;
    fp->cap = SIZE;
    fp->size = 0;
    memset(fp->buffer, 0, SIZE);

    return fp;
}
```

### 4.3ffulsh函数

考虑到`fclose`函数会先强制刷新缓存区，在调用`close`函数，这里我们先实现`fflush`函数

```c
//C标准库fflush的参数
int fflush ( FILE * stream );
```

```c
//自己实现
void fflush_(FILE_* fp) {
    if (fp->size > 0) {
        write(fp->fileno, fp->buffer, fp->size);
        fp->size = 0;
    }
}
```

### 4.4fclose函数

```c
//C标准库fclose的参数
int fclose ( FILE * stream );
```

```c
//自己实现
void fclose_(FILE_* fp) {
    fflush_(fp);
    close(fp->fileno);
}
```

### 4.5fwrite函数

```c
//C标准库fwrite的参数
size_t fwrite ( const void * ptr, size_t size, size_t count, FILE * stream );
```

```c
//自己实现
void fwrite_(const void* ptr, int num, FILE_* fp) {		//默认输入字符串，省略size
    // 1.写入到缓存区
    memcpy(fp->buffer + fp->size, ptr, num);
    fp->size += num;
    // 2.判断是否刷新
    if (fp->flags & SYNC_NOW)   fflush_(fp);
    else if (fp->flags & SYNC_FULL) {
        if (fp->size == fp->cap)    fflush_(fp);
    } else if (fp->flags & SYNC_LINE) {
        if (fp->buffer[fp->size - 1] == '\n')   fflush_(fp);  //不考虑"abc\nef"
    }
}
```

