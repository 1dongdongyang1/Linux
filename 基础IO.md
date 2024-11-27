# 基础IO

[toc]


## 1.C语言文件接口

+ C 语言中的文件操作函数提供了对文件进行读写操作的接口。
+ 常用的文件操作函数来自于标准库 `stdio.h`，以下是一些常用的 C 语言文件操作函数及其用途：

### 1.1文件的开启与关闭

+ `fopen`：打开文件

  ```c
  FILE *fopen(const char *filename, const char *mode);
  ```

  + `filename`：文件名（路径）。
  + `mode`：文件打开模式，例如：
    + `"r"`：以只读模式打开文件。
    + `"w"`：以写模式打开文件（如果文件存在则清空文件内容；如果不存在则创建新文件）。
    + `"a"`：以追加模式打开文件，写操作会加在文件末尾。
    + `"r+"`：以读写模式打开文件。
    + `"w+"`：以读写模式打开文件，并清空文件内容。
    + `"a+"`：以读写模式打开文件，并将写操作加在文件末尾。

+ `fclose`：关闭文件

  ```c
  int fclose(FILE *stream);
  ```

  + `stream`：文件指针，指向要关闭的文件。

### 1.2文件读写

+ `fgetc`：从文件中读取一个字符

  ```c
  int fgetc(FILE *stream);
  ```

  + 读取成功返回读取到的字符（`int` 类型），到达文件末尾返回 `EOF`。

+ `fgets`：从文件中读取一行

  ```c
  char *fgets(char *str, int n, FILE *stream);
  ```

  + `str`：存储读取数据的字符数组。
  + `n`：最多读取 `n-1` 个字符。
  + `stream`：文件指针。
  + 返回 `str` 或在出错时返回 `NULL`。

+ `fputc`：向文件中写入一个字符

  ```c
  int fputc(int char, FILE *stream);
  ```

+ `fputs`：向文件中写入一个字符串

  ```c
  int fputs(const char *str, FILE *stream);
  ```

+ `fread`：从文件中读取一块数据

  ```c
  size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
  ```

  + `ptr`：存储读取内容的缓冲区。
  + `size`：每个元素的大小。
  + `nmemb`：要读取的元素个数。
  + 返回读取的元素个数。

+ `fwrite`：向文件中写入一块数据

  ```c
  size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
  ```

  + `ptr`：要写入的数据缓冲区。
  + `size`：每个元素的大小。
  + `nmemb`：要写入的元素个数。
  + 返回写入的元素个数。

### 1.3文件指针移动

+ `fseek`：移动文件指针

  ```c
  int fseek(FILE *stream, long offset, int whence);
  ```

  + `offset`：偏移量。
  + `whence`：
    + `SEEK_SET`：从文件开头移动。
    + `SEEK_CUR`：从当前文件指针位置移动。
    + `SEEK_END`：从文件末尾移动。

+ `ftell`：返回文件指针的位置

  ```c
  long ftell(FILE *stream);
  ```

  + 返回当前文件指针的位置。

+ `rewind`：重置文件指针到文件开始

  ```c
  void rewind(FILE *stream);
  ```

### 1.4文件状态检查

+ `feof`：检查文件是否到达末尾

  ```c
  int feof(FILE *stream);
  ```

  + 返回非零值表示文件结束，零表示文件未结束。

+ `ferror`：检查文件操作是否出错

  ```c
  int ferror(FILE *stream);
  ```

  + 返回非零值表示发生错误。



## 2.Linux系统调用IO接口

+ 在 Linux 操作系统中，I/O（输入/输出）操作是通过一组系统调用来完成的。
+ 这些系统调用提供了底层的接口，允许程序与文件系统、设备和其他硬件资源进行交互。
+ 常见的 I/O 系统调用包括文件操作、读写操作以及管理文件描述符的调用。

### 2.1文件操作系统调用

+ `open`：打开文件

  ```c
  int open(const char *pathname, int flags);
  int open(const char *pathname, int flags, mode_t mode);
  ```

  > 虽然 `open` 函数的两个版本看起来像 C++ 的函数重载，但在底层实现上，它并不是通过真正的重载机制，而是通过可变参数和条件判断来实现灵活性的。

  + `pathname`：要打开的文件路径。

  + `flags`：文件的打开模式，可以是以下值的组合：
    + `O_RDONLY`：只读模式。
    + `O_WRONLY`：只写模式。
    + `O_RDWR`：读写模式。
    + `O_CREAT`：如果文件不存在则创建它。
    + `O_TRUNC`：打开文件时将其截断为 0 大小（如果文件存在）。
    + `O_APPEND`：追加写入
  + `mode`：如果使用 `O_CREAT`，该参数指定文件的权限。

  **返回值**：成功时返回文件描述符（正数），失败时返回 `-1`。

+ `close`：关闭文件

  ```c
  int close(int fd);
  ```

  + `fd`：文件描述符，表示要关闭的文件。

### 2.2读写操作系统调用

+ `read`：读取文件

  ```c
  ssize_t read(int fd, void *buf, size_t count);
  ```

  + `fd`：文件描述符。
  + `buf`：缓冲区，用于存储读取的数据。
  + `count`：要读取的字节数。

  **返回值**：返回读取的字节数，遇到文件末尾返回 `0`，出错时返回 `-1`。

+ `write`：写入文件

  ```c
  ssize_t write(int fd, const void *buf, size_t count);
  ```

  + `fd`：文件描述符。
  + `buf`：缓冲区，包含要写入的数据。
  + `count`：要写入的字节数。

  **返回值**：返回写入的字节数，失败时返回 `-1`。

### 2.3文件定位系统调用

+ `lseek`：文件指针移动

  ```c
  off_t lseek(int fd, off_t offset, int whence);
  ```

  + `fd`：文件描述符。
  + `offset`：偏移量。
  + `whence`：决定从哪里开始移动偏移：
    + `SEEK_SET`：从文件开头移动。
    + `SEEK_CUR`：从当前文件位置移动。
    + `SEEK_END`：从文件末尾移动。

  **返回值**：返回新的文件偏移量，失败时返回 `-1`。

### 2.4关系小结

+ 系统调用接口是操作系统提供的接口，C语言IO接口底层实现是调用系统调用IO接口的。
+ 不只是C语言，其他语言也是一样，IO接口的实现在底层都调用了系统调用IO接口。

## 3.重定向

**重定向**是 Linux 和 Unix 系统中的一种功能，允许用户将命令的输入或输出从默认位置（如标准输入/标准输出）重定向到文件、设备或其他命令。这使得操作系统的 I/O 更加灵活，并允许用户轻松地将程序的输出保存到文件中或从文件读取输入。



重定向的基础是三个文件描述符，分别对应系统的输入、输出和错误流：

+ **标准输入（stdin）**：文件描述符 `0`，默认从键盘输入。
+ **标准输出（stdout）**：文件描述符 `1`，默认输出到终端（屏幕）。
+ **标准错误（stderr）**：文件描述符 `2`，默认输出错误信息到终端。



我们如何将我们自己写的shell加上重定向的功能呢？

> [原shell](https://github.com/1dongdongyang1/Myshell/commit/2faf32f63d1ba1fb83be12c63c61e7189049ef90)

+ 命令是子进程来执行的，真正重定向的工作是子进程来完成的

+ 父进程的作用就是给子进程提供重定向信息的

  **依据**：子进程和父进程共享内存，父进程的全局变量就可以给子进程提供信息

> [代码](https://github.com/1dongdongyang1/Myshell/commit/fc5d48d9cfb598822b40021fed7e394f3bd619cd#diff-1fc384789a7a6c7eed42594cd643b34b61b78507f79a09b72dcef426cdd559fcR138)



## 4.模拟实现C语言接口

> [代码](https://github.com/1dongdongyang1/Mylibc/commit/c75242d80da68605571674c27836fd6f37b9b160)

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

