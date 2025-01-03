# 关于skynet-mingw [![Build status](https://ci.appveyor.com/api/projects/status/9j45lldyxmfdau3r?svg=true)](https://ci.appveyor.com/project/dpull/skynet-mingw)

[skynet-mingw](https://github.com/dpull/skynet-mingw) 是[skynet](https://github.com/cloudwu/skynet)的windows平台的实现。其主要特点是：

1. skynet 以submodule链接，方便升级，**确保不改**。
1. 仅扩展了700行代码，方便维护。
1. 自动更新skynet，自动构建，自动化测试，确保质量。

## 编译
不想自行编译的朋友可访问 [自动构建平台获取最新的构建版本](https://ci.appveyor.com/project/dpull/skynet-mingw/build/artifacts)。

1. 安装 [MinGW](http://sourceforge.net/projects/mingw/files/)
1. 安装 `gcc g++`
1. 安装 `pthread (dev)`
1. 运行 `MinGW\msys\1.0\msys.bat`
1. 运行 `prepare.sh`
1. 运行 `make`

### 常见问题
1. 建议使用 `MinGW\msys\1.0\msys.bat` 进行编译
1. 错误: `gcc: Command not found`, 解决: 修改 `msys\1.0\etc\fstab` 中的 `/mingw` 路径
1. 当提示缺少类似`dlfcn.h`文件时，建议看看头文件搜索路径是否有问题，举个例子`perl(Strawberry Perl)`中有`gcc`程序，同时它注册了系统环境变量

## 测试

```bash
./skynet.exe examples/config    # Launch first skynet node  (Gate server) and a skynet-master (see config for standalone option)
./3rd/lua/lua examples/client.lua   # Launch a client, and try to input hello.
```

## 已知问题

1. console服务不可用(无法对stdin进行select)， 会提示如下出错信息，暂时没有解决方案。

```bash
stack traceback:
        [C]: in function 'assert'
        ./lualib/socket.lua:361: in function 'socket.lock'
        ./service/console.lua:15: in upvalue 'func'
        ./lualib/skynet.lua:452: in upvalue 'f'
        ./lualib/skynet.lua:105: in function <./lualib/skynet.lua:104>
```

2. 使用`skynet.abort`无法退出，看堆栈卡在了系统中，暂时没有解决方案。（替代方案`os.exit(true)`）

```bash
#0  0x77bd718c in ntdll!ZwWaitForMultipleObjects () from C:\WINDOWS\SYSTEM32\ntdll.dll
#1  0x74c0a4fa in WaitForMultipleObjectsEx () from C:\WINDOWS\SYSTEM32\KernelBase.dll
#2  0x74c0a3d8 in WaitForMultipleObjects () from C:\WINDOWS\SYSTEM32\KernelBase.dll
#3  0x6085be1c in pause () from D:\MinGW\msys\1.0\bin\msys-1.0.dll
#4  0x6085ccf1 in msys-1.0!cwait () from D:\MinGW\msys\1.0\bin\msys-1.0.dll
#5  0x6080dff4 in msys-1.0!cygwin_stackdump () from D:\MinGW\msys\1.0\bin\msys-1.0.dll
#6  0x00413fe5 in ?? ()
#7  0x00413e8f in ?? ()
#8  0x00412a1b in ?? ()
#9  0x0040f77b in ?? ()
#10 0x0040f151 in ?? ()
#11 0x00403869 in __mingw_opendir ()
#12 0x0000000a in ?? ()
#13 0x0069fe30 in ?? ()
#14 0x00000000 in ?? ()
```

## 相关文档
[开发笔记](https://blog.dpull.com/post/2015-11-08-skynet_mingw) 



## Make
`GNU Make`是一个控制从程序的源文件中生成程序的可执行文件和其他非源文件的工具。

`Make`可以从一个名为`Makefile`的文件中获得如何构建程序的信息，该文件列出了每个非源文件以及如何从其他文件计算它。当我们编写一个程序时，应该为它编写一个`Makefile`文件，这样就可以使用`Make`来编译和安装这个程序，**在执行make命令时，会根据目标文件依赖的文件最后修改时间来推断目标文件是否是最新版本，如果目标文件已是最新版本将不会执行对应的命令**。

一个简单的`makefile`将以下面的规则组成。`target`表示生成的文件名称，`prerequisites`是一个用于创建目标文件的输入文件，而`recipe`是一组需要执行的动作，一个动作可以有多个命令。
```make
target: prerequisites
    recipe
```

在`Make`官网中，它给了我们一个例子，该例子描述了一个`edit`的可执行文件，生成这个可执行文件需要用到8个`.o`类型的文件作为输入文件，而这8个`.o`类型的文件分别由8个`c`源文件和3个头文件生成。并且提供了一个`clean`功能，这个功能可以删除所有的`.o`文件。
```make
edit : main.o kbd.o command.o display.o insert.o search.o files.o utils.o
    gcc -o edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o

main.o : main.c defs.h
    gcc -c main.c

kbd.o : kbd.c defs.h command.h
    gcc -c kbd.c

command.o : command.c defs.h command.h
    gcc -c command.c

display.o : display.c defs.h buffer.h
    gcc -c display.c

insert.o : insert.c defs.h buffer.h
    gcc -c insert.c

search.o : search.c defs.h buffer.h
    gcc -c search.c

files.o : files.c defs.h buffer.h command.h
    gcc -c files.c

utils.o : utils.c defs.h
    gcc -c utils.c

clean :
    rm edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o
```

通过在命令行中输入`make`命令，就会按照makefile文件定义的规则开始编译，并生成edit可执行文件。
```bash
make
```

如果想要重新编译可以执行`make clean`命令来执行`clean`功能，删除掉已经编译好的文件。
```bash
make clean
```

在一个`Makefile`中一共包含五种内容，分别是`显式规则、隐式规则、定义变量、指令和注释`。

1、 显式规则说明了，如何生成一个或多的的目标文件。这是由`Makefile`的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。

2、 隐式规则,由于我们的`make`有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写`Makefile`，这是由`make`所支持的。

3、 变量的定义,在`Makefile`中我们要定义一系列的变量，变量一般都是字符串，这个类似于C语言中的宏，当`Makefile`被执行时，其中的变量都会被扩展到相应的引用位置上。

4、 指令包括了三个部分，一个是在一个`Makefile`中引用另一个`Makefile`，就像C语言中的`include`一样;另一个是指根据某些情况指定`Makefile`中的有效部分，就像C语言中的预编译`#if`一样;还有就是定义一个多行的命令。

5、 `Makefile`中只有行注释，和`UNIX`的`Shell`脚本一样，其注释是用`“#”`字符。如果你要在你的`Makefile`中使用`“#”`字符，可以用反斜框进行转义，如：`“/#”`。

## 伪目标
在上面的示例中，提供了`clean`功能，这种写法是存在问题的，当我们的项目路径下没有`clean`文件时，执行`make clean`命令是没有问题的，但是如果项目路径下有一个`clean`文件，那么这个命令将不会被执行，为了让`make`命令不将`clean`当做文件可以添加一个特殊的标识来避免。
```make
.PHONY: clean

clean :
    rm edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o
```

上面的操作通过`.PHONY: clean`将`clean`标识为一个伪目标，这样即时项目路径下有`clean`文件，clean`命令`依然可以正常运行。

## makfile文件名
默认情况下执行`make`命令时会在目录下查找`GNUmakefile、makefile、Makefile`。

通常我们最好使用`Makefile`作为文件名，因为该文件基本是出现在目录列表的前面，更容易被看到，虽然支持`GNUmakefile`文件名，但是最好不要使用这个文件名，这个文件名通常是当编写的程序是特地为GNU所提供的时候才使用。

如果执行`make`命令，在项目目录下并没有发现上面说到的三个文件名的文件，将不会执行任何操作，如果我们的`makefile`在其它目录下那么我们可以在执行`make`命令时指定`makefile`所在的路径。
```make
make -f /user/config/config /home/config/config
```

在使用`-f`或`--file`参数时可以指定一个或多个`makefile`文件，并且在使用这两个参数的时候并不会检查目标文件名是否是`makefile`文件名，而是直接运行指定的文件内容

## 包含其它makefile文件
在`makefile`文件中可以`include`指令让当前`makefile`文件暂停，先读取`include`指令指定的文件再继续向下执行。
```make
include tester.mk
```

在一个`include`指令中可以包含多个`makefile`文件，按照顺序进行执行，每个`makefile`之间需要用空格分隔。
```make
include tester.mk dev.mk
```

## make默认执行目标
默认情况下使用`make`命令将会执行`Makefile`文件中的第一个目标，对于下面的文件，执行`make`命令时将执行`main.o`目标，并不会执行`clean`目标，除非使用`make clean`来指定执行的目标。
```Makefile
main.o : main.c defs.h
    gcc -c main.c

clean :
    rm main.o
```

如果想要设置`make`命令的默认执行目标可以使用`.DEFAULT_GOAL`来指定执行的默认目标，在下面的`Makefile`文件中设置了默认执行`clean`目标。
```Makefile
.DEFAULT_GOAL=clean
main.o : main.c defs.h
    gcc -c main.c

clean :
    rm main.o
```

## 变量的使用
通过定义变量可以一处定义多处使用，在使用变量的地方可以使用`$(变量名)`的方式来引用变量。
```Makefile
obj = main.o

main: $(obj)
    gcc -o main $(obj)

$(obj) : main.c
    gcc -c main.c -o $(obj)

clean :
    rm main.o
```

使用变量时`make`还可以自动推断命令，不需要执行需要编译的C文件，`make`自动推到出来，对于上面的例子我们可以简化一下。
```Makefile
obj = main.o

# 命令会自动推断main生成时需要使用main.o文件
main:

# 省略掉了gcc编译的命令，make会自动使用cc命令来编译

$(obj) :

clean :
    rm main.o
```

## make的执行流程
GNU make是一行一行解析makefiles的，解析流程分为六步：
1、 读取完整的逻辑行，包含反斜杠的转义。

2、 去除注释内容。

3、 如果该行以命令前缀（tab）开头，并且处于规则上下文中，将该行添加到当前命令并阅读下一行。

4、 将立即展开的元素展开。

5、 扫描行中的分隔符（入“:”或“=”），确定该行时宏定义还是规则。

6、 内化生成的操作，阅读下一行。

在第四步中说到了立即展开，实际GNU make有两种不同的阶段完成对应的工作，分别为`立即展开`和`延时展开`，变量和函数的展开发生在第一阶段，称为立即展开，剩下的都为延迟展开。

## 通配符
通配符用来指定一组符合条件的文件名。`Makefile`的通配符与Bash一致，主要有`*`（匹配0个或多个字符）、`?`（匹配任意一个字符）和`[…]`（匹配指定的字符） 。比如 *.o 表示所有后缀名为.o的文件，例如通配符在之前的例子就已经使用到。
```Makefile
.PHONY: clean
clean:
    rm -rf *.o
```

通配符并不是在所有地方都可以使用，在定义变量中是不可以的，下面定义的`objects`变量被应用的规则中表示的是一个“`*.o`”的文件而不是匹配所有`.o`文件。
```Makefile
objects = *.o
foo : $(objects)
    cc -o foo $(CFLAGS) $(objects)
```

## 模式规则
`make`命令可以对文件名使用类似于正则表达式的方式进行匹配，主要用到的匹配符是百分号`%`。比如，假定当前目录下有`a.c`和`b.c`两个源码文件，需要将它们编译为对应的对象文件`a.o`和`b.o`，需要编写两条规则。
```Makefile
a.o: a.c
    gcc -c a.c -o a.o

b.o: b.c
    gcc -c b.c -o b.o
```

如果工程包含大量的C文件这么做是非常繁琐的，为了简化可以使用` %.o: %.c` 模式匹配，效果等同于如下写法。使用匹配符`%`，可以将大量同类型的文件，只使用一条`Makefile`规则就完成构建（**通过模式匹配就可以只实现一条规则将所有的`.c`文件编译为对应的`.o`文件**）。
```Makefile
a.o: a.c
b.o: b.c
```

“`%`”定义对文件名的匹配，表示任意长度的非空字符串。在依赖目标中同样可以使用“`%`”，只是依赖目标中“%”的取值，取决于其目标。

`make`为了方便模式规则的使用，还提供了一些自动化变量，这些变量可以规则的命令中使用
| 变量 | 说明 |
| - | - |
| `$@` | 表示规则中的所有目标文件的集合 |
| `$%` | 只有当目标是函数库文件时，表示规则中的目标成员名，如果目标不是函数库文件值为空（库文件UNIX下是`.a`，Windows下是`.lib`） |
| `$<` | 依赖目标中的第一个目标名字 |
| `$?` | 所有比目标新的依赖目标集合 |
| `$^` | 所有依赖目标的集合，如果有重复的依赖目标会去重 |
| `$+` | 所有依赖目标的集合，如果有重复的依赖目标不会去重 |
| `$*` | 目标模式中“`%`”及其之前的部分 |
| `$(@D)` | “`$@`”的目录部分，如果“`$@`”中没有斜杠，该属性值为“`.`”表示当前目录 |
| `$(@F)` | “`$@`”的文件部分 |
| `$(*D)` | “`$@`”文件的目录部分 |
| `$(*F)` | “`$@`”的文件部分，不包括文件后缀 |
| `$(%D)` | 函数包文件成员的目录部分 |
| `$(%F)` | 函数包文件成员的文件名部分 |
| `$(<D)` | 依赖目标中的第一个目标的目录部分 |
| `$(<F)` | 依赖目标中的第一个目标的文件名部分 |
| `$(^D)` | 所有依赖目标文件中目录部分，自动去重 |
| `$(^F)` | 所有依赖目标文件中的文件名部分，自动去重 |
| `$(+D)` | 所有依赖目标文件中的目录部分，不去重 |
| `$(+F)` | 所有依赖目标文件中的文件部分，不去重 |
| `$(?D)` | 所有被更新文件的目录部分 |
| `$(?F)` | 所有被更新文件的文件名部分 |

需要注意的是模式规则中“`%`”的展开和变量与函数的展开是有区别的，“`%`”的展开发生在变量和函数的展开之后。变量和函数的展开发生在`make`载入`Makefile`时，而“`%`”的展开则发生在运行时。

## 双冒号规则
对于普通冒号定义的规则，当规则中目标文件存在时，此规则的命令是不会被执行的，但是对于一个没有依赖只有命令行的双冒号规则，当引用此目标时，规则的命令会被无条件执行。
```Makefile
hello::
        gcc hello.c -o hello
clean:
        rm hello
```

上面的写法跟下面伪目标的效果是相同的。
```Makefile
.PHONY: hello
hello:
	gcc hello.c -o hello
```

## 目标文件搜索
一个工程文件中的源文件有很多，并且存放的位置可能不相同（工程中的文件会被放到不同的目录下），这个时候就需要用到目标文件搜索功能。

`make`为我们提供了两个搜索方式，分别是`VPATH（一般搜索）`和`vpath（选择搜索）`，`VPATH`是环境变量，使用时需要指定文件的路径；`vpath`是关键字，按照模式搜索，搜索的时候不仅需要加上文件的路径，还需要加上相应限制的条件。

在`Makefile`中，可以通过在`VPATH`变量中设置一系列目录路径来指定源文件的搜索路径。例如：
```Makefile
VPATH = src:../lib:../../include
```

上面的代码中，VPATH`变量`指定了三个目录，用冒号分隔。`Make`在查找源文件时，会先在当前目录下查找，如果找不到，就会依次在`VPATH`指定的目录中查找，直到找到为止。

在`Makefile`中，可以通过在`vpath`变量中设置文件模式和对应的目录路径来使用`vpath`。例如：
```Makefile
vpath %.c src
vpath %.h include
vpath %.o obj
```

上面的代码中，`vpath`指定了三种文件模式和对应的目录路径，`%`表示通配符，匹配对应模式的文件名。`Make` 在查找符合模式的文件时，会先在当前目录下查找，如果找不到，就会按照`vpath`中指定的目录顺序依次查找，直到找到为止。

通过`vpath`可以更加灵活地指定不同类型的文件的搜索路径，避免了`VPATH`在搜索时搜索所有的文件类型的缺陷。

在查找依赖文件时，Make会首先查找当前目录下是否存在所需的文件，如果不存在则会根据`vpath`变量指定的搜索路径查找文件，如果还是找不到则会根据`VPATH`变量指定的搜索路径查找文件。
```Makefile
CC := gcc

vpath %.h inc
vpath %.c src
VPATH := src1

hello : main.c main.h
	$(CC) -c -o $@ $< -I inc
```

`VPATH`的优点：
1、 可以指定`Make`在哪些目录中查找依赖文件，能够完全覆盖`Makefile`中的规则。

2、 可以将源文件和依赖文件分开存放，方便管理和维护。

3、 可以使用通配符匹配一类文件，并指定对应的目录路径。

`VPATH`的缺点：
1、 对于每一个`Make`规则，都需要手动添加`VPATH`变量，这会增加`Makefile`的复杂度。

2、 在`Make`进行依赖文件查找时，`VPATH`会覆盖当前目录，这可能会对意图不明确的`Makefile`产生影响。

`vpath`的优点：
1、 全局变量，避免了重复添加和修改变量的麻烦。

2、 可以根据文件类型进行匹配，并指定对应的搜索路径。

3、 使用简单，不用为每一个规则手动添加搜索路径变量。

`vpath`缺点：
1、 只能指定文件类型和目录路径，而不能针对特定的文件进行指定。

2、 如果存在同名文件，可能会出现查找到错误文件的问题。

## 内置目标
`.PHONY`，表示伪目标，位目标不会被作为文件看待，只是一个标签
`.SUFFIXES`，所有依赖指出一系列在后缀规则中需要检查的后缀名
`.DEFAULT`，如果目标的某个条件没有被定义，则该目标指定的规则将会被执行。
```Makefile
.DEFAULT:
	@echo default

# 这里依赖的force没有被定义，所以会执行.DEFAULT定义的目标
test: force
	@echo test

test2:
	@echo test2
```

`.PRECIOUS`，可以保留不想删除的中间文件。
```Makefile
# 在执行清除命令时不会删除main.o文件
.PRECIOUS:main.o
```

`.INTERMEDIATE`，通过把某文件指定为中间文件，则make每次执行完成都会删除该文件
```Makefile
# 执行完Make命令后会清除掉指定的中间文件
.INTERMEDIATE:main.o foo.o bar.o
```

`.NOTINTERMEDIATE`
`.SECONDARY`
`.SECONDEXPANSION`，辅助扩展作为一个make内置目标，其后定义的依赖会在makefile读取完成后，再展开一次。
`.DELETE_ON_ERROR`，我们知道，make在执行命令时会检查命令的返回码，如果返回码不等于0的话，make会默认停止并退出。但如果在出错之前，目标已经生成了，这样可能并不是我们想要的。假设我们想在目标生成后在做点什么，但出错了也就是我们的任务并没有完成，而此时目标已经生成了，这也就是意味着，可能我们的任务之后就没机会做了，因下次make检查目标时发现目标已存在就不会再执行命令。此时，我们可以使用.DELETE_ON_ERROR来告诉make如果执行某目标的命令时出错，请删除该目标。
```Makefile
main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o
    mv main bin/main

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o

.PHONY:cleanall cleanobj

cleanall:cleanobj
	-rm main

cleanobj:
	-rm *.o

.DELETE_ON_ERROR:
```

`.IGNORE`，让Make命令忽略错误
`.LOW_RESOLUTION_TIME`，低解析度时间，就是告诉make对于某目标进行依赖检查时，时间精度精确到秒级就可以了。
`.SILENT`，告诉make不要输出命令信息。
`.EXPORT_ALL_VARIABLES`，导出所有变量。
`.ONESHELL `，make在执行指令时，会为该指令单独开个线程，而且默认会使用互相独立的shell来执行各指令。而只要makefile中出现.ONESHELL那么make对于所有指令都会使用同一个shell。

## 变量
变量在声明时需要给予初始值，在使用时需要给在变量名前加上“`$`”符号，最好使用“`()`”或“`{}`”将变量包裹起来，如果需要使用到“`$`”字符，那么需要用“`$$`”来表示。

下面是使用变量的一个示例：
```Makefile
objects = program.o foo.o utils.o
program: $(objects)
  cc -o program $(objects)

$(objects): defs.h
```

变量会在使用它的地方展开，这就像是C中的宏一样，对于示例中的内容展开后可以得到下面的内容。
```Makefile
objects = program.o foo.o utils.o
program: program.o foo.o utils.o
  cc -o program program.o foo.o utils.o

program.o foo.o utils.o: defs.h
```

我们可以使用`?=`给未定义的变量附加默认值。
```Makefile
# 这里定义了aa变量所以会输出world，如果没有定义这个变量将会输出hello
aa := world   
aa ?= hello   
              
all:          
    echo $(aa)
```

## 嵌套变量
定义变量的值时，我们可以使用其它变量来构造变量的值。

对于下面的例子，在执行之后会在控制台中输出`Hub`。
```Makefile
foo = $(bar)

bar = $(ugh)

ugh = Hub?

all:
  echo $(foo)
```

嵌套变量可以将变量的真实值推到后面，但是如果出现互相依赖，这会让`make`陷入无限制的变量展开过程（对于这种情况会报错），还有就是在变量中使用函数时会导致`make`运行较慢，为了避免这种情况，我们可以使用“`:=`”操作符来定义变量。
```Makefile
x := foo

y := $(x)bar

x := later
```

对于上面的实例其展开后的结果如下：
```Makefile
y := foo bar
x := later
```

通过展开的结果我们可以看出，在使用`:=`定义变量时，前面的变量不能使用后面的变量，只能使用已经定义好的变量。
```Makefile
# 对于这种情况，展开后y = bar
y := $(x)bar
x := later
```

## 追加变量
我们可以使用“`+=`”操作符向变量后面追加新的值。

在下面的例子中，通过`+=`操作符向`objects`的末尾追加了`another.o`字符，展开后`objects`的值将为`main.o foo.o bar.o utils.o another.o`。
```Makefile
objects = main.o foo.o bar.o utils.o
objects += another.o
```

对于一个变量如果没有定义过，那么`+=`会自动变为`=`，如果前面已有变量被定义，那么`+=`会继承于前一次操作的赋值符，如果前一次使用的是`:=`定义的变量，那么`+=`会变为`:=`为其赋值。
```Makefile
variable := value

variable += more
```

下面的写法和上面的写法是等价的
```MakeFile
variable := value

variable := $(variable) more
```

## override指令
如果有变量时通过`make`命令参数设置的，那么`Makefile`会对这个变量的赋值忽略，如果想要在`Makefile`中设置这类参数的值可以使用“`override`”指令符来实现。

```Makefile
override variable = value

override variable += more test
```

如果一个变量的值需要设置多行，我们可以使用`define`和`endef`指令来定义。
```Makefile
define two-lines
  echo foo
  echo $(bar)
endef
```

我们同样可以使用`override`指令来标识多行变量
```Makefile
override define foo
  bar
endef
```

对于不需要的变量我们可以使用undefine指令来取消定义。
```Makefile
foo := foo
bar = bar

undefine foo
undefine bar

# 下面将会打印两个undefined
$(info $(origin foo))
$(info $(flavor bar))
```

## 环境变量
make运行时的系统环境变量可以被加载到`Makefile`文件中，但是如果`Makefile`中又定义了这个变量或者说这个变量由`make`指令带入，那么系统环境变量的值将被覆盖。

如果我们在环境变量中设置了`CFLAGS`变量，就可以在`Makefile`中使用这个变量。

## 目标变量
前面说到的自定义变量都是全局变量，如果想要定义某个变量的作用范围只在某条规则及其连带的规则中，而不会影响规则链以外的全局变量，这个时候就可以使用目标变量。

在下面的例子中，无论全局变量CFLAGS值为什么，在prog规则链中其值始终为-g
```Makefile
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
  $(CC) $(CFLAGS) prog.o foo.o bar.o

prog.o: prog.c
  $(CC) $(CFLAGS) prog.o
foo.o: foo.c
  $(CC) $(CFLAGS) foo.o
bar.o: bar.c
  $(CC) $(CFLAGS) bar.o
```

## 自动化变量
| 变量 | 说明 |
| - | - |
| $@ | 表示规则中的所有目标文件的集合 |
| $% | 只有当目标是函数库文件时，表示规则中的目标成员名，如果目标不是函数库文件值为空（库文件UNIX下是`.a`，Windows下是`.lib`） |
| $< | 依赖目标中的第一个目标名字 |
| $? | 所有比目标新的依赖目标集合 |
| $^ | 所有依赖目标的集合，如果有重复的依赖目标会去重 |
| $+ | 所有依赖目标的集合，如果有重复的依赖目标不会去重 |
| $* | 目标模式中“%”及其之前的部分 |
| $(@D) | “$@”的目录部分，如果“$@”中没有斜杠，该属性值为“.”表示当前目录 |
| $(@F) | “$@”的文件部分 |
| $(*D) | “$@”文件的目录部分 |
| $(*F) | “$@”的文件部分，不包括文件后缀 |
| $(%D) | 函数包文件成员的目录部分 |
| $(%F) | 函数包文件成员的文件名部分 |
| $(<D) | 依赖目标中的第一个目标的目录部分 |
| $(<F) | 依赖目标中的第一个目标的文件名部分 |
| $(^D) | 所有依赖目标文件中目录部分，自动去重 |
| $(^F) | 所有依赖目标文件中的文件名部分，自动去重 |
| $(+D) | 所有依赖目标文件中的目录部分，不去重 |
| $(+F) | 所有依赖目标文件中的文件部分，不去重 |
| $(?D) | 所有被更新文件的目录部分 |
| $(?F) | 所有被更新文件的文件名部分 |

## 内置变量
1、 `MAKEFILE_LIST`，该变量是一个只读变量，其包含了当前构建过程中所有被包含的Makefile文件的列表

2、 `.DEFAULT_GOAL`
`MAKE_RESTARTS`
`MAKE_TERMOUT`
`MAKE_TERMERR`
`.RECIPEPREFIX`
`.VARIABLES`，该变量记录了已经定义的所有变量名列表
`.FEATURES`
`.INCLUDE_DIRS`
`.EXTRA_PREREQS`

## 条件语法
条件语句可以根据一个变量的值来控制 `make`执行或者忽略`Makefile`的特定部分。 条件语句可以是两个不同变量、或者变量和常量值的比较，条件语句只能 用于控制`make`实际执行的`makefile`文件部分，它不能控制规则的`shell`命令执行过程。 `Makefile`中使用条件控制可以做到处理的灵活性和高效性。

条件语句一共有6个关键字分别为`ifeq`、`ifneq`、`ifdef`、`ifndef`、`else`、`endif`，其中`else`和`endif`关键字可以和其它关键字配合使用。

`ifeq`关键字判断参数是否不相等，如果相等则为`true`，否则为`false`。
```Makefile
ifeq ($(a), $(b))
	@echo "a = b"
else
	@echo "a != b"
endif
```

`ifneq`关键字判断参数是否不相等，不相等则为`true`，否则为`false`。
```Makefile
ifneq ($(a), $(b))
	@echo "a = b"
else
	@echo "a != b"
endif
```

`ifdef`关键字判断变量的值是否不为空，有值将为`true`，否则为`false`。
```Makefile
ifdef v
	@echo "a = b"
else
	@echo "a != b"
endif
```

`ifndef`关键字判断变量的值是否不为空，没有值为`true`，否则为`false`。
```Makefile
ifndef v
	@echo "a = b"
else
	@echo "a != b"
endif

```

## 隐式规则
在 make 中，有一些标准的重建目标文件的方式被频繁使用。例如，一种通常的制作目标文件的方式是使用 C 编译器 cc 编译 C 源文件。

隐式规则告诉 make 如何使用通常的技术，因此当您想使用它们时，您不必详细指定它们。例如，有一个用于 C 编译的隐式规则。文件名决定了运行哪些隐式规则。例如，C 编译通常会接受一个 .c 文件并生成一个 .o 文件。因此，当 make 发现这个文件名后缀组合时，它会应用 C 编译的隐式规则。

隐式规则的链可以依次应用；例如，make 将通过 .c 文件从 .y 文件重新制作一个 .o 文件。

内建的隐式规则在它们的配方中使用了几个变量，通过更改这些变量的值，您可以更改隐式规则的工作方式。例如，变量 CFLAGS 控制由 C 编译的隐式规则传递给 C 编译器的标志。

您还可以通过编写模式规则来定义自己的隐式规则。

后缀规则是定义隐式规则的一种更有限的方式。模式规则更通用和更清晰，但为了兼容性，保留了后缀规则。

## 隐式规则使用的变量
这是一些内建隐式规则中使用的常见程序名称变量以及它们的默认值：

AR： 归档维护程序，默认为 ‘ar’。

AS： 用于编译汇编文件的程序，默认为 ‘as’。

CC： 用于编译C程序的程序，默认为 ‘cc’。

CXX： 用于编译C++程序的程序，默认为 ‘g++’。

CPP： 运行C预处理器并将结果输出到标准输出的程序，默认为 ‘$(CC) -E’。

FC： 用于编译或预处理Fortran和Ratfor程序的程序，默认为 ‘f77’。

M2C： 用于编译Modula-2源代码的程序，默认为 ‘m2c’。

PC： 用于编译Pascal程序的程序，默认为 ‘pc’。

CO： 用于从RCS中提取文件的程序，默认为 ‘co’。

GET： 用于从SCCS中提取文件的程序，默认为 ‘get’。

LEX： 用于将Lex语法转换为源代码的程序，默认为 ‘lex’。

YACC： 用于将Yacc语法转换为源代码的程序，默认为 ‘yacc’。

LINT： 用于在源代码上运行lint的程序，默认为 ‘lint’。

MAKEINFO： 用于将Texinfo源文件转换为Info文件的程序，默认为 ‘makeinfo’。

TEX： 用于从TeX源文件生成TeX DVI文件的程序，默认为 ‘tex’。

TEXI2DVI： 用于从Texinfo源文件生成TeX DVI文件的程序，默认为 ‘texi2dvi’。

WEAVE： 用于将Web转换为TeX的程序，默认为 ‘weave’。

CWEAVE： 用于将C Web转换为TeX的程序，默认为 ‘cweave’。

TANGLE： 用于将Web转换为Pascal的程序，默认为 ‘tangle’。

CTANGLE： 用于将C Web转换为C的程序，默认为 ‘ctangle’。

RM： 用于删除文件的命令，默认为 ‘rm -f’。

以下是一些用于程序附加参数的变量：

ARFLAGS： 传递给归档维护程序的标志，默认为 ‘rv’。

ASFLAGS： 传递给汇编程序的额外标志（在显式调用 ‘.s’ 或 ‘.S’ 文件时）。

CFLAGS： 传递给C编译器的额外标志。

CXXFLAGS： 传递给C++编译器的额外标志。

COFLAGS： 传递给RCS co程序的额外标志。

CPPFLAGS： 传递给C预处理器和使用它的程序（如C和Fortran编译器）的额外标志。

FFLAGS： 传递给Fortran编译器的额外标志。

GFLAGS： 传递给SCCS get程序的额外标志。

LDFLAGS： 当编译器应该调用链接器（‘ld’）时传递给编译器的额外标志。库（‘-lfoo’）应添加到LDLIBS变量中。

LDLIBS： 传递给编译器在应该调用链接器（‘ld’）时的库标志或名称。LOADLIBES是LDLIBS的已弃用（但仍受支持）的替代品。非库链接器标志，如’-L’，应放在LDFLAGS变量中。

LFLAGS： 传递给Lex的额外标志。

YFLAGS： 传递给Yacc的额外标志。

PFLAGS： 传递给Pascal编译器的额外标志。

RFLAGS： 传递给Ratfor程序的Fortran编译器的额外标志。

LINTFLAGS： 传递给lint的额外标志。

通过更改这些变量的值，您可以自定义隐式规则的行为，而无需重新定义规则本身。这使得对默认规则进行微调变得非常方便。

## 隐式规则链
这段文本解释了关于中间文件（Intermediate files）的概念和它们在GNU Make中的处理方式。中间文件是由隐式规则生成的文件，它们被认为是临时的，可以被Make系统删除，以避免不必要的文件积累。以下是一些关键点：

链（Chain）： 有时文件可以通过一系列的隐式规则生成，例如通过运行Yacc和cc可以从n.y生成n.o。这样的一系列规则称为链。

中间文件： 如果n.c不存在但Make知道如何生成它，那么n.c被称为中间文件。中间文件是隐式规则生成的，会在Make数据库中以及在文件系统中存在。如果n.c被当作中间文件，Make将只有在必要的情况下才会生成它，而且在不再需要时会删除它。

标记中间文件： 您可以通过将文件列为.INTERMEDIATE特殊目标的先决条件来显式标记文件为中间文件。

.NOTINTERMEDIATE： 如果您希望防止文件被视为中间文件，可以将其列为.NOTINTERMEDIATE特殊目标的先决条件。

禁用中间文件： 您可以通过在Makefile中提供.NOTINTERMEDIATE作为一个没有先决条件的特殊目标来完全禁用中间文件。

次要文件： 如果您不希望Make因为文件不存在而自动创建文件，但又不希望自动删除该文件，可以将其列为.SECONDARY特殊目标的先决条件。

优化规则： 为了性能原因，Make系统提供了一些优化规则，可以优化某些情况，而不需要使用完整的规则链。

链中的规则不可重复： 在一个链中，单个隐式规则不会出现两次。这样可以防止Make尝试通过两次运行链接器来生成目标文件等荒谬的情况。

性能优化： Make系统为了性能原因，在搜索构建隐式规则的先决条件时，不会考虑非终端的匹配任意规则。

这些概念有助于理解Make系统如何处理生成文件以及如何优化构建过程。

## https://blog.csdn.net/NickDeCodes/article/details/133872925

## 文本处理函数

1、 字符串替换函数`subst`，将字符串中指定的字符替换掉，替换完将返回新的字符串。
```Makefile
$(subst FROM,TO,TEXT)

# 示例
$(subst ee, EE , test ee)    #test EE
```

2、 模式替换函数`patsubst`，按照指定的模式将匹配到的字符替换为新指定的模式。
```Makefile
# 把字串“x.c.c bar.c”中以.c 结尾的单词替换成以.o 结尾的字符。函数的返回结果是“x.c.o bar.o”
$(patsubst %.c,%.o,x.c.c bar.c)     #x.c.o bar.o
```

3、 去除空格函数`strp`，可以去除字符串开头和结尾处的空格字符，并将字符串内部的多个连续空格合并为一个。
```Makefile
STR =        a          b  c 
LOSTR = $(strip $(STR))
all:
	$(info $(STR))              #        a          b  c
	$(info $(LOSTR))            #a b c
```

4、 查找字符串函数`findstring`，在源字符串中查找指定的字符，如果源字符串中包含指定的字符将返回指定的字符，否则返回空。
```Makefile
# 第一个函数结果是字“a”；第二个值为空字符
$(info $(findstring a,a b c))
$(info $(findstring a,b c))
```

5、 过滤函数`filter`，过滤掉源字符串不匹配的字符，只保留符合指定模式的字符（**可以使用多个模式**）。
```Makefile
sources := foo.c bar.c baz.s ugh.h 

# 返回值为：foo.c bar.c baz.s
all :
	$(info $(filter %.c %.s,$(sources)))
```

6、 反过滤函数`filter-out`，过滤掉源字符串匹配的字符，只保留不符合指定模式的字符（**可以使用多个模式**）。
```Makefile
sources := foo.c bar.c baz.s ugh.h 

# 返回值为：ugh.h
all :
	$(info $(filter-out %.c %.s,$(sources)))
```

7、 排序函数`sort`，将给定的字符串列表以首字母为准进行升序，并去掉重复字符。
```Makefile
# 返回值为：bar foo lose
$(sort foo bar lose foo)
```

8、 取单词函数`word`，获取源字符串列表中指定位置的元素（**位置从1开始算**）。
```Makefile
# 返回值为：bar
$(word 2, foo bar baz)
```

9、 取范围内字符串`wordlist`，从源字符串列表中获取指定范围内的字符串。
```Makefile
# 返回值为：bar baz
$(wordlist 2, 3, foo bar baz)
```

10、 统计单词数目`words`，统计处源字符串列表中字符串的数量。
```Makefile
sources := foo.c bar.c baz.s ugh.h 

# 返回值为：4
all :
	$(info $(words $(sources)))
```

11、 取首个单词`firstword`，取出源字符串中的第一个单词。
```Makefile
# 返回值为：foo
$(firstword foo bar)
```

## 文件名处理函数
1、 取目录函数`dir`，从文件名中取出各个文件名的目录部分，并返回目录部分字符串（如果文件名中出现空格会截取最后一个`/`之前的字符，如果文件名中没有`/`则认为是`./`下的文件）。
```Makefile
# 返回值为：/path/to/
$(dir /path/to/file.txt)
```

2、 取文件名函数`notdir`，从文件名中取出非目录部分，非目录部分是指最后一个斜线`/`（不包括斜线）之后的部分，删除所有文件名中的目录部分，只保留非目录部分。
```Makefile
# 返回值为：/file.txt
$(dir /path/to/file.txt)
```

3、 取后缀函数`suffix`，从文件序列中取出文件名的后缀，后缀是文件名中最后一个以点`.`开始的（包含点号）部分，如果文件名中不包含一个点号，则为空。
```Makefile
# 返回值为：.c .c
$(suffix src/foo.c src-1.0/bar.c)
```

4、 取前缀函数`basename`，前缀部分指的是文件名中最后一个点号之前的部分。
```Makefile
# 返回值为：src/foo src-1.0/bar /home/jack/.font
$(basename src/foo.c src-1.0/bar.c /home/jack/.font.cache-1 hacks)
```

5、 加后缀函数`addsuffix`，为指定的所有文件添加后缀。
```Makefile
# 返回值为：foo.c bar.c
$(addsuffix .c,foo bar)
```

6、 加前缀函数`addprefix`，为指定的所有文件添加前缀。
```Makefile
# 返回值为：src/foo src/bar
$(addprefix src/,foo bar)
```

7、 单词连接函数`join`，将两个文件名列表中的对应位置值拼接在一起，返回一个新的文件名列表。
```Makefile
# LIST1与LIST2数量相等
# 返回值为：a.c b.o
$(join a b, .c .o)

# LIST1与LIST2数量不相等，多余部分连接在最后一并返回
# 返回值为：a.c b.o c
$(join a b c, .c .o)
```

8、 获取匹配模式文件名函数`wildcard`，列出当前目录下所有符合指定模式的文件名。
```Makefile
# 返回值foo.c bar.c；返回当前目录下所有.c文件列表
$(wildcard *.c)
```

9、 相对路径转绝对路径函数`abspath`，将传入的相对路径名转为绝对路径。
```Makefile
name = ../openwrt/COPYING

all:
	$(info $(abspath $(name)))      #COPYING文件的绝对路径
```


## 其他函数
1、 循环函数`foreach`，将字符串列表元素用空格分开取出幅值给第一个变量，然后执行传入的表达式。
```Makefile
dirs := a b c d 
files := $(foreach dir, $(dirs), $(dir)/)
# 返回值：a/  b/  c/  d/
```

2、 `shell`函数，可以执行对应的`shell`命令，并返回执行结果。
```Makefile
$(shell pwd)           #当前路径
```

3、 `if`函数，类似于Java中的三木运算符。
```Makefile
# 当SRC_DIR值不为空时返回 /home/src否则返回$(SRC_DIR)
SRC_DIR = a
SUBDIR = $(if $(SRC_DIR), $(SRC_DIR), /home/src)
```