# make的使用

**目录**：

1. [什么是make](#1.0)
2. [主要功能](#2.0)
3. [基本组成](#3.0)
4. [基本使用](#4.0)

---

## 1.什么是make<p id="1.0"></p>

**概念**：

+ `make` 是一个在 Linux 和 Unix 系统上广泛使用的构建自动化工具，主要用于编译和构建大型项目。
+ `make` 使用一个名为 `Makefile/makefile` 的文件，根据文件中的规则和指令来决定如何编译和链接程序。

## 2.主要功能<p id="2.0"></p>

1. **自动化编译**：通过设置文件依赖和编译规则，可以自动化整个编译过程，减少手动输入命令的繁琐。
2. **增量构建**：`make` 只会重新编译那些自上次构建以来发生变化的文件，从而加快构建速度。
3. **项目管理**：可以管理项目中的多个目标，例如可执行文件、库文件、清理临时文件等。

## 3.基本组成<p id="3.0"></p>

一个典型的`Makefile`由以下几个部分组成：

1. **变量**：可以在 `Makefile` 中定义变量，以便在整个文件中复用。例如：

   ```makefile
   CC = gcc
   CFLAGS = -Wall -g		
   ```

   类似于C语言中的宏定义。

2. **目标（Target）**：`make` 命令最终生成的文件或输出。

3. **依赖项（Dependency）**：目标所依赖的文件或其他目标。

4. **规则（Rule）**：当依赖项发生变化时，用于生成目标的命令。

**基本格式**：

```makefile
target: dependencies
	command				#需要注意：command前面是Tab缩进
```

**示例**：

假设我们有一个包含两个源文件 `main.c` 和 `helper.c` 的项目，并且希望生成一个可执行文件 `my_program`。可以创建如下的 `Makefile`：

```makefile
# 定义变量
CC = gcc
CFLAGS = -Wall -g

# 定义目标
my_program: main.o helper.o
    $(CC) $(CFLAGS) -o my_program main.o helper.o
#可看作 gcc -Wall -g -o my_program main.o helper.o

# 定义依赖和规则
main.o: main.c
    $(CC) $(CFLAGS) -c main.c

helper.o: helper.c
    $(CC) $(CFLAGS) -c helper.c

# 定义清理规则
.PHONY:clean			#.PHONY修饰后，clean可以强制执行，不管是否变化
clean:
    rm -f my_program *.o
```

在此 `Makefile` 中：

+ `my_program` 是最终生成的目标，依赖于 `main.o` 和 `helper.o`。
+ `main.o` 和 `helper.o` 是中间目标，分别依赖于 `main.c` 和 `helper.c`。
+ `clean` 是一个清理目标，用于删除生成的文件。可以使用 `make clean` 来执行它。

> 如果对gcc编译的过程不理解，可以查阅小羊的系列文章。[点我](https://blog.csdn.net/2301_80030944/article/details/142521837)

在正常使用gcc编译多个源文件时，都是将每个源文件都汇编后，再链接，但是示例的写法却是先链接在编译。这样的结构像**栈**。

```makefile
my_program: main.o helper.o							#目标是my_program，找依赖项main.o helper.o	
    gcc -Wall -g -o my_program main.o helper.o		#发现依赖项没有实现，去找依赖项，链接的这条命令就被压栈了
    
main.o: main.c										#将依赖项main.o看作目标，找main.c，有main.c
    gcc -Wall -g -c main.c							#先执行编译命令

helper.o: helper.c									#同main.o
    gcc -Wall -g -c helper.c
```

当然，我们使用gcc编译多个文件时，直接就放在一起编译了，`makefile`同样可以

```makefile
my_program: main.c helper.c
	gcc -o my_program main.c helper.c
```

## 4.基本使用<p id="4.0"></p>

1. **编译**：在包含 `Makefile` 的目录中执行以下命令来编译项目：

   ```bash
   make
   ```

   `make` 默认执行第一个目标（通常是生成可执行文件）。如果文件没有变化，则 `make` 不会重新编译。

2. **指定目标**：可以指定要构建的目标。例如：

   ```bash
   make clean
   ```

3. **查看过程**：使用 `make -n` 可以查看构建过程中将要执行的命令，而不实际执行它们。

**常用的`make`选项**：

+ `make -f 文件名`：指定使用的 `Makefile`，默认是当前目录中的 `Makefile`。
+ `make -j [n]`：并行构建，`n` 表示要并行执行的任务数量。
+ `make -k`：遇到错误时继续构建其他目标。
+ `make -B`：强制重新构建所有目标，无论是否有变化。

**常用的`makefile`里的代替符号**：

1. **`$@`**

   + 表示规则的目标（target），即当前规则要生成的文件。

   - 例如，在规则 `my_program: main.o helper.o` 中，`$@` 会替代为 `my_program`。

2. **`$^`**

   - 表示规则的**所有**依赖项（dependencies），包含多个依赖文件时，依赖文件会以空格分隔。
   - 例如，在规则 `my_program: main.o helper.o` 中，`$^` 会替代为 `main.o helper.o`。

3. **`$<`**

   - 表示第一个依赖项（通常是第一个源文件）。
   - 例如，在规则 `main.o: main.c` 中，`$<` 会替代为 `main.c`。

​	

