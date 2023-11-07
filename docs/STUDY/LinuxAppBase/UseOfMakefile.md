## Makefile最基本的语法规则

    目标： 依赖1 依赖2 ...
    [TAB]命令

当“依赖”比“目标”新，执行它们下面的命令。我们要把上面三个命令写成makefile规则，如下：

```bash
test ：a.o b.o  //test是目标，它依赖于a.o b.o文件，一旦a.o或者b.o比test新的时候，
就需要执行下面的命令，重新生成test可执行程序。
gcc -o test a.o b.o
a.o : a.c  //a.o依赖于a.c，当a.c更加新的话，执行下面的命令来生成a.o
gcc -c -o a.o a.c
b.o : b.c  //b.o依赖于b.c,当b.c更加新的话，执行下面的命令，来生成b.o
gcc -c -o b.o b.c
```

## Makefile的语法
### a.通配符
假如一个目标文件所依赖的依赖文件很多，那样岂不是我们要写很多规则，这显然是不合乎常理的

我们可以使用通配符，来解决这些问题。

修改代码如下：

```bash
test: a.o b.o 
	gcc -o test $^
	
%.o : %.c
	gcc -c -o $@ $<
```

%.o：表示所用的.o文件

%.c：表示所有的.c文件

\$\@：表示目标

\$\<：表示第1个依赖文件

\$\^：表示所有依赖文件

### b.假想目标：.PHONY
1.我们想清除文件，我们在Makefile的结尾添加如下代码就可以了：

```bash
clean:
	rm *.o test
```

*1）执行 make ：生成第一个可执行文件。
*2）执行 make clean : 清除所有文件，即执行： rm \*.o test。

make后面可以带上目标名，也可以不带，如果不带目标名的话它就想生成第一个规则里面的第一个目标。

2.使用Makefile

执行：**make [目标]** 也可以不跟目标名，若无目标默认第一个目标。我们直接执行make的时候，会在makefile里面找到第一个目标然后执行下面的指令生成第一个目标。当我们执行 make clean 的时候，就会在 Makefile 里面找到 clean 这个目标，然后执行里面的命令，这个写法有些问题，原因是我们的目录里面没有 clean 这个文件，这个规则执行的条件成立，他就会执行下面的命令来删除文件。

如果：该目录下面有名为clean文件怎么办呢？

我们在该目录下创建一个名为 “clean” 的文件，然后重新执行：make然后make
clean，结果(会有下面的提示：)：

```bash
make: \`clean' is up to date.
```

它根本没有执行我们的删除操作，这是为什么呢？

我们之前说，一个规则能过执行的条件：

*1)目标文件不存在
*2)依赖文件比目标新

现在我们的目录里面有名为“clean”的文件，目标文件是有的，并且没有

依赖文件，没有办法判断依赖文件的时间。这种写法会导致：有同名的"clean"文件时，就没有办法执行make clean操作。解决办法：我们需要把目标定义为假象目标，用关键子PHONY

```bash
.PHONY: clean //把clean定义为假象目标。他就不会判断名为“clean”的文件是否存在，
```

然后在Makfile结尾添加.PHONY: clean语句，重新执行：make clean，就会执行删除操作。

### C. 变量

在makefile中有两种变量：

1), 简单变量(即使变量)：

A := xxx    # A的值即刻确定，在定义时即确定

对于即使变量使用 “:=” 表示，它的值在定义的时候已经被确定了

2）延时变量

B = xxx   # B的值使用到时才确定

对于延时变量使用“=”表示。它只有在使用到的时候才确定，在定义/等于时并没有

确定下来。

想使用变量的时候使用“\$”来引用，如果不想看到命令是，可以在命令的前面加上"\@"符号，就不会显示命令本身。当我们执行make命令的时候，make这个指令本身，会把整个Makefile读进去，进行全部分析，然后解析里面的变量。常用的变量的定义如下：

```bash
:= # 即时变量
= # 延时变量
?= # 延时变量, 如果是第1次定义才起效, 如果在前面该变量已定义则忽略这句
+= # 附加, 它是即时变量还是延时变量取决于前面的定义
?=: 如果这个变量在前面已经被定义了，这句话就会不会起效果，
```

实例：

```bash
A := $(C)
B = $(C)
C = abc

#D = 100ask
D ?= weidongshan

all:
	@echo A = $(A)
	@echo B = $(B)
	@echo D = $(D)

C += 123
```

执行：

```bash
make
```

结果：

```bash
A =
B = abc 123
D = weidongshan
```

分析：

1) A := \$(C)：

A为即使变量，在定义时即确定，由于刚开始C的值为空，所以A的值也为空。

2) B = \$(C)：
B为延时变量，只有使用到时它的值才确定，当执行make时，会解析Makefile里面的所用变量，所以先解析C= abc,然后解析C += 123，此时，C = abc 123，当执行：\@echo B = \$(B) B的值为 abc 123。

3) D ?= weidongshan：

D变量在前面没有定义，所以D的值为weidongshan，如果在前面添加D = 100ask，最后D的值为100ask。

我们还可以通过命令行存入变量的值 例如：

执行：make D=123456 里面的 D ?= weidongshan 这句话就不起作用了。

结果：

```bash
A =
B = abc 123
D = 123456
```


# Makefile函数

makefile里面可以包含很多函数，这些函数都是make本身实现的，下面我们来几个常用的函数。引用一个函数用“\$”。

## 函数foreach

函数foreach语法如下： 

```bash
$(foreach var,list,text) 
```

前两个参数，‘var’和‘list’，将首先扩展，注意最后一个参数 ‘text’ 此时不扩展；接着，对每一个 ‘list’ 扩展产生的字，将用来为 ‘var’ 扩展后命名的变量赋值；然后 ‘text’ 引用该变量扩展；因此它每次扩展都不相同。结果是由空格隔开的 ‘text’。在 ‘list’ 中多次扩展的字组成的新的 ‘list’。‘text’ 多次扩展的字串联起来，字与字之间由空格隔开，如此就产生了函数 foreach 的返回值。

实例：

```bash
A = a b c
B = $(foreach f, &(A), $(f).o)

all:
	@echo B = $(B)
```

结果：

```bash
B = a.o b.o c.o
```

## 函数filter/filter-out

函数filter/filter-out语法如下：

```bash
$(filter pattern...,text)     # 在text中取出符合patten格式的值
$(filter-out pattern...,text) # 在text中取出不符合patten格式的值
```

实例：

```bash
C = a b c d/

D = $(filter %/, $(C))
E = $(filter-out %/, $(C))

all:
        @echo D = $(D)
        @echo E = $(E)
```

结果：

```bash
D = d/
E = a b c
```

## Wildcard

函数Wildcard语法如下：

```bash
$(wildcard pattern) # pattern定义了文件名的格式, wildcard取出其中存在的文件。
```

这个函数 wildcard 会以 pattern 这个格式，去寻找存在的文件，返回存在文件的名字。

实例：

在该目录下创建三个文件：a.c b.c c.c

```bash
files = $(wildcard *.c)

all:
        @echo files = $(files)
```

结果：

```bash
files = a.c b.c c.c
```

我们也可以用wildcard函数来判断，真实存在的文件

实例：

```bash
files2 = a.c b.c c.c d.c e.c  abc
files3 = $(wildcard $(files2))

all:
        @echo files3 = $(files3)
```

结果：

```bash
files3 = a.c b.c c.c
```

## patsubst函数

函数 patsubst 语法如下：

```bash
$(patsubst pattern,replacement,\$(var))
```

patsubst 函数是从 var 变量里面取出每一个值，如果这个符合 pattern 格式，把它替换成 replacement 格式，

实例：

```bash

files2  = a.c b.c c.c d.c e.c abc

dep_files = $(patsubst %.c,%.d,$(files2))

all:
        @echo dep_files = $(dep_files)

```

结果：

```bash
dep_files = a.d b.d c.d d.d e.d abc
```



# Makefile实例 #
前面讲了那么多Makefile的知识，现在开始做一个实例。

之前编译的程序`002_syntax`，有个缺陷，将其复制出来，新建一个`003_example`文件夹，放在里面。
在`c.c`里面，包含一个头文件`c.h`，在`c.h`里面定义一个宏，把这个宏打印出来。
c.c:
```
#include <stdio.h>
#include <c.h>

void func_c()
{
	printf("This is C = %d\n", C);
}
```

c.h:
```
#define C 1
```

然后上传编译，执行`./test`,打印出：
```
This is B
This is C =1
```
测试没有问题，然后修改`c.h`：
```
#define C 2
```
重新编译，发现没有更新程序，运行，结果不变，说明现在的Makefile存在问题。

为什么会出现这个问题呢， 首先我们test依赖c.o，c.o依赖c.c，如果我们更新c.c，会重新更新整个程序。
但c.o也依赖c.h，我们更新了c.h，并没有在Makefile上体现出来，导致c.h的更新，Makefile无法检测到。
因此需要添加:
```
c.o : c.c c.h
```
现在每次修改c.h，Makefile都能识别到更新操作，从而更新最后输出文件。

这样又冒出了一个新的问题，我们怎么为每个.c文件添加.h文件呢？对于内核，有几万个文件，不可能为每个文件依次写出其头文件。
因此需要做出改进，让其自动生成头文件依赖，可以参考这篇文章：http://blog.csdn.net/qq1452008/article/details/50855810
```
gcc -M c.c // 打印出依赖

gcc -M -MF c.d c.c  // 把依赖写入文件c.d

gcc -c -o c.o c.c -MD -MF c.d  // 编译c.o, 把依赖写入文件c.d
```
修改Makefile如下：
```

objs = a.o b.o c.o

dep_files := $(patsubst %,.%.d, $(objs))
dep_files := $(wildcard $(dep_files))

test: $(objs)
	gcc -o test $^

ifneq ($(dep_files),)
include $(dep_files)
endif

%.o : %.c
	gcc -c -o $@ $< -MD -MF .$@.d

clean:
	rm *.o test

distclean:
	rm $(dep_files)
	
.PHONY: clean	
```
首先用obj变量将.o文件放在一块。
利用前面讲到的函数，把obj里所有文件都变为.%.d格式，并用变量dep_files表示。
利用前面介绍的wildcard函数，判断dep_files是否存在。
然后是目标文件test依赖所有的.o文件。
如果dep_files变量不为空，就将其包含进来。
然后就是所有的.o文件都依赖.c文件，且通过-MD -MF生成.d依赖文件。
清理所有的.o文件和目标文件
清理依赖.d文件。

现在我门修改了任何.h文件，最终都会影响最后生成的文件，也没任何手工添加.h、.c、.o文件，完成了支持头文件依赖。

下面再添加CFLAGS，即编译参数。比如加上编译参数-Werror，把所有的警告当成错误。
```
CFLAGS = -Werror -Iinclude

…………


%.o : %.c
	gcc $(CFLAGS) -c -o $@ $< -MD -MF .$@.d
```
现在重新make，发现以前的警告就变成了错误，必须要解决这些错误编译才能进行。在`a.c`里面声明一下函数：
```
void func_b();
void func_c();
```
重新make，错误就没有了。

除了编译参数-Werror，还可以加上-I参数，指定头文件路径，-Iinclude表示当前的inclue文件夹下。
此时就可以把c.c文件里的`#include ".h"`改为`#include <c.h>`，前者表示当前目录，后者表示编译器指定的路径和GCC路径。
