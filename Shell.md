# 管道和重定向

## 1. 重定向输出

`>`     如果文件已存在，它的内容将被覆盖。

`>>`    输出会附加到文件的末尾。

`$ kill -HUP 1234 >kellout.txt 2>killerr.txt`把标准输出和标准错误输出分别重定向到不同的文件按中。

`$ kill -1 1234 >killouterr.txt 2>&1`把标准输出和标准错误输出都重定向到同一个文件中。

`$ kill -1 1234 >/dev/null 2>&1`用Linux的通用“回收站”/dev/null来丢弃所有的输出信息。

## 2. 重定向输入

`more < killout.txt`

## 3. 管道

使用sort命令对ps命令的输出进行排序：

- 如果不适用管道

`$ ps > psout.txt`

`$sort psout.txt > pssort.out`

- 使用管道

`$ ps | sort > pssort.out`

# shell编程

## 1. 执行shell脚本

1. `$ /bin/sh first`
2. `$ chmod +x first`            `$ ./first`

## 2. shell的语法

---

### 2.1变量

- 在默认情况下，所有变量都被看做**字符串**并以字符串来存储，即使它们被复制为数值时也是如此。
- 区分大小写。
- 可以通过在变量名前加一个**$** 符号来访问它的内容。
- 如果字符串里包含**空格**，就必须用引号把它们括起来。
- **等号两边不能有空格**。

`read`命令将用户的输入赋值给一个变量，需要一个参数作为变量名。

```
$ read salutation
hello
$ echo $salutation
hello
```

---

#### 引号的使用

- 在双引号中使用一个$变量表达式，程序执行到这一行时就会把变量替换为它的值；在单引号中不会替换。

- 在`$`字符前面加上一个`\`字符可以取消它的特殊含义。

- ---

#### 环境变量

| 环境变量 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| $HOME    | 当前用户的家目录                                             |
| $PATH    | 以冒号分隔的用来搜索命令的目录列表                           |
| $PS1     | 命令提示符，通常是$字符，但在bash中，你可以使用一些更复杂的值。例如，字符串[\u@\h \w]$就是一个流行的默认值，它给出用户名、机器名和当前目录名，当然也包括一个$提示符。 |
| $PS2     | 二级提示符，用来提示后续的输入，通常是>字符。                |
| $IFS     | 输入域分隔符。当shell读取输入时，它给出用来分隔单词的一组字符，他们通常是空格、制表符和换行符 |
| $0       | shell脚本的名字                                              |
| $#       | 传递给脚本的参数个数                                         |
| $$       | shell脚本的进程号，脚本程序通常会用它来生成一个唯一的临时文件，如/tmp/tmpfile_$$ |

---

#### 参数变量

| 参数变量  | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| $1,$2,... | 脚本程序的参数                                               |
| $*        | 在一个变量中列出所有的参数，各个参数之间用环境变量IFS中的第一个字符分隔开。如果IFS被修改了，那么$*将命令行分割为参数的方式就将随之改变。 |
| $@        | 是$*的一种变体，它不适用IFS环境变量，所以即使IFS为空，参数也不会挤在一起。 |

```bash
$ IFS=''
$ set foo bar bam
$ echo "$@"
foo bar bam
$ echo "$*"
foobarbam
$ unset IFS
$ echo "$*"
foo bar bam
```

### 2.2 条件

---

#### test或 [ 命令

- 布尔判断命令

```bash
if [ -f fred.c ]; then
...
fi
```

- `[ `    符号和被检查的条件之间留出空格。如果if和then在同一行，就必须使用；分隔。

| 字符串比较         | 结果                                   |
| ------------------ | -------------------------------------- |
| string1 = string2  | 如果两个字符串相等则结果为真           |
| string1 != string2 | 如果两个字符串不同则结果为真           |
| -n string          | 如果字符串不为空则结果为真             |
| -z string          | 如果字符串为null（一个空串）则记过为真 |

| 算数比较      | 结果                               |
| ------------- | ---------------------------------- |
| exp1 -eq exp2 | 如果两个表达式相等则结果为真       |
| exp1 -ne exp2 | 如果两个表达式不等则结果为真       |
| exp1 -gt exp2 | 如果exp1大于exp2则结果为真         |
| exp1 -ge exp2 | 如果exp1大于等于exp2则结果为真     |
| exp1 -lt exp2 | 如果exp1小于exp2则结果为真         |
| exp1 -le exp2 | 如果exp1小于等于exp2则结果为真     |
| ! exp         | 如果表达式为假则结果为真，反之亦然 |

| 文件条件测试 | 结果                                       |
| ------------ | ------------------------------------------ |
| -d file      | 如果文件是一个目录则结果为真               |
| -b file      | 判断该文件是否存在，并且是否为块设备文件   |
| -c file      | 判断该文件是否存在，并且是否为字符设备文件 |
| -L file      | 判断该文件是否存在，并且是否为符号链接文件 |
| -p file      | 判断该文件是否存在，并且是否为管道文件     |
| -S file      | 判断该文件是否存在，并且是否为套接字文件   |
| -f file      | 如果文件是一个普通文件则结果为真           |
|              |                                            |
| -e file      | 如果文件存在则结果为真，通常使用-f选项     |
| -g file      | 如果文件的set-group-id位被设置则结果为真   |
| -r file      | 如果文件可读则结果为真                     |
| -s file      | 如果文件的大小不为0则结果为真              |
| -u file      | 如果文件的set-user-id位被设置则结果为真    |
| -w file      | 如果文件按可写则结果为真                   |
| -x file      | 如果文件可执行则结果为真                   |

- set-group-id和set-user-id（也叫做set-gid和set-uid），set-uid位授予了程序其拥有者的访问权限而不是其使用者的访问权限，set-gid位授予了程序其所在组的访问权限。

### 2.3 控制结构

---

#### if 语句

```bash
if condition1
then
	statements
elif condition2
	statements
else
	statements
fi
exit 0
```

---

#### for 语句

```bash
for file in $(ls f*.sh); do
	lpr $file
done
exit 0
```

---

#### while 语句

```bash
while condition do
	statements
done
```

do 和while之间的语句将反复执行，直到条件不再为真

---

#### until语句

```bash
until condition
do
	statements
done
```

循环将反复执行直到条件为真，而不是在条件为真时反复执行。

---

#### case语句

```bash
case cariable in
	pattern [ | pattern] ...) statements;;
	pattern [ | pattern] ...) statements;;
	...
esac
```

```bash
echo "Is it morning? Please answer yes or no"
read timeofday
case "$timeofday" in
	[yY] | [Yy][Ee][Ss] )
		echo "Good Morning"
		echo "Up bright and early this morning"
		;;
	[nN]* )
		echo "Good Afternoon"
		;;
	* )
		echo "Sorry,answer not recognized"
		echo "Please answer yes or no"
		exit 1
		;;
esac
exit 0
```

---

#### 语句块

如果想在某些只允许使用单个语句的地方（比如在AND或OR列表中）使用多条语句，可以把它们括在花括号{ }中来构造一个语句块。

```bash
get_confirm && {
	frep -v "$cdcatnum" $tracks_file > $temp_file
	cat $temp_file > $tracks_file
	echo
	add_record_tracks
}
```

---

### 2.4 函数

```bash
function_name(){
	statements
}
```

- 必须在调用一个函数之前先对它进行定义。

```bash
#!/bin/sh

yes_or_no() {
	echo "Is your name $* ?"
	while true 
	do
		echo -n "Enter yes or no: "
		read x
		case "$x" in
			y|yes) return 0;;
			n|no) return 1;;
			*) echo "Ansewr yes or no"
		esac
	done
}

echo "Original parameters are $*"
if yes_or_no "$1"
then
	echo "Hi $1,nice name"
else
	echo "Never mind"
fi
exit 0
```

---

### 2.5 命令

#### break

跳出for、while或until循环

---

#### continue

跳过本次循环

---

####　：命令

是一个空命令，相当于true的一个别名。

---

#### eval 命令

```bash
foo=10
x=foo
y='$'$x
echo $y
```

输出$foo

```bash
foo=10
x=foo
eval y='$'$x
echo $y
```

输出10

---

#### exec 命令

- 将当前shell替换为一个不同的程序。

  `exec wall "Thanks for all the fish"`

  脚本中的这个命令会用wall命令替换当前的shell。脚本程序中exec命令后面的代码都不会执行，因为执行这个脚本的shell已经不存在了。

- 修改当前文件描述符

  `exec 3 < afile`

  文件描述符3被打开以便从文件afile中读取数据。

---

#### export 命令

将作为它参数的变量导出到子shell中，并使之在子shell中有效。

export2

```bash
#!/bin/sh
echo "$foo"
echo "$bar"
```

export1

```bash
#!/bin/sh
foo="The firsh meta-syntactic variable"
export bar="The second meta-syntactic variable"

export2
```

执行

```
$ ./export1
The second meta-syntactic variable
```

可以看出bar的值对新脚本仍然有效

---

#### expr 命令

expr命令将它的参数当作一个表达式来求值。

```
x=`expr $x + 1`
```

反引号（``）字符使x取值为命令expr $x + 1的执行结果。也可以使用$( )替换反引号

```
x=$(expr $x + 1)
```

| 表达式求值     | 说明                                          |
| -------------- | --------------------------------------------- |
| expr1 \| expr2 | 如果expr1非零，则等于expr1，否则等于expr2     |
| expr1 & expr2  | 只要有一个表达式为零，则等于零，否则等于expr1 |
| expr1 = expr2  | 等于                                          |
| expr1 > expr2  | 大于                                          |
| expr1 >= expr2 | 大于等于                                      |
| expr1 < expr2  | 小于                                          |
| expr1 <= expr2 | 小于等于                                      |
| expr1!=  expr2 | 不等于                                        |
| expr1 + expr2  | 加法                                          |
| expr1 - expr2  | 减法                                          |
| expr1 * expr2  | 乘法                                          |
| expr1 / expr2  | 除法                                          |
| expr1 % expr2  | 取余                                          |

---

#### set 命令

set命令的作用是为shell设置参数变量。

```bash
#!/bin/sh
echo the date is $(date)
set $(date)
echo The month is $2

exit 0
```

把date命令的输出设置为参数列表，然后通过位置参数$2获得月份

---

#### unset 命令

unset命令的作用是从环境中删除变量或函数。这个命令不能删除shell本身定义的只读变量（如IFS）。

---

#### trap 命令

`trap command signal`

第一个参数是接收到指定信号时将要采取的行动，第二个参数时要处理的信号名。

脚本程序通常是以从上到下的顺序解释执行的，所以必须在你想要保护的那部分代码之前指定trap命令。

如果要重置某个信号的处理方式到其默认值，只需要将command设置为 - 。如果要忽略某个信号，就把command设置为空字符串。一个不带参数的trap命令将列出当前设置的信号及其行动清单。

| 信号     | 说明                                 |
| -------- | ------------------------------------ |
| HUP(1)   | 挂起，通常因终端掉线或用户推出而引发 |
| INT(2)   | 中断，通常因按下Ctrl+C组合键而引发   |
| QUIT(3)  | 退出，通常因按下Ctrl+\组合键而引发   |
| ABRT(6)  | 中止，通常因某些严重的执行错误而引发 |
| ALRM(14) | 报警，通常用来处理超时               |
| TERM(15) | 终止，通常在系统关机时发送           |

```bash
#!/bin/bash

trap 'rm -f /tmp/my_tmp_file_$$' INT
echo creating file /tmp/my_tmp_file_$$
date > /tmp/my_tmp_file_$$

echo "press interrupt (Ctrl+C) to interrupt ..."
while [ -f /tmp/my_tmp_file_$$ ]; do
	echo File exits
	sleep 1
done
echo The file no longer exists

trap INT
echo creating file /tmp/my_tmp_file_$$
date > /tmp/my_tmp_file_$$

echo "press interrupt (Ctrl+C) to interrupt ..."
while [ -f /tmp/my_tmp_file_$$ ]; do
	echo File extsts
	sleep 1
done

echo we never get here
exit 0
```

```
运行这个脚本，在每次循环时按下Ctrl+C组合键，将得到如下所示的输出：
creating file /tmp/my_tmp_file_141
press interrupt (Ctrl+C) to interrupt ...
File extsts
File extsts
File extsts
File extsts
The file no longer exists
creating file /tmp/my_tmp_file_141
press interrupt (Ctrl+C) to interrupt ...
File extsts
File extsts
File extsts
File extsts
```

先用trap命令安排它出现一个INT(中断)信号时执行`rm -f /tmp/my_tmp_file_$$`命令删除临时文件。脚本程序然后进入一个while循环，只要临时文件存在，循环就一直持续下去。当用户按下Ctrl+C组合键时，脚本程序就会执行`rm -f /tmp/my_tmp_file_$$`语句，然后继续下一个循环。因为临时文件现在已经被删除了，所以第一个while循环将正常退出。

接下来，脚本程序再次调用trap命令，这次是指定当一个INT信号出现时不执行任何命令。脚本程序然后重新创建临时文件并进入第二个while循环。这次当用户按下Ctrl+C组合键时，没有语句被指定执行，所以采取默认处理方式，即立即终止脚本程序。因为脚本程序被立即终止了，所以最后的echo和exit语句永远都不会执行。

---

#### find 命令

`find [path] [option] [tests] [actions]`

`# find / -mount -name test -print`从根目录开始查找名为test的文件，并且输出该文件的完整路径，-mount表示不要搜索挂载的其他文件系统的目录。

| 选项            | 含义                               |
| --------------- | ---------------------------------- |
| -depth          | 在查看目录本身之前先搜索目录的内容 |
| -follow         | 跟随符号链接                       |
| -maxdepths N    | 最多搜索N层目录                    |
| -mount(或-xdev) | 不搜索其他文件系统中的目录         |

| 测试             | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| -atime N         | 文件在N天之前被最后访问过                                    |
| -mtime N         | 文件在N天之前被最后修改过                                    |
| -name pattern    | 文件名（不包括路径名）匹配提供的模式pattern                  |
| -newer otherfile | 文件比otherfile文件要新                                      |
| -type c          | 文件的类型为c，c时一个特殊类型。最常见的是d（目录）和f（普通文件）。 |
| -user username   | 文件的拥有者是指定的用户username                             |

| 操作符，短格式 | 操作符，长格式 | 含义                   |
| -------------- | -------------- | ---------------------- |
| ！             | -not           | 测试取反               |
| -a             | -and           | 两个测试都必须为真     |
| -o             | -or            | 两个测试有一个必须为真 |

---

#### grep 命令

**通用正则表达式解析器（General Regular Expression Parser）**

`grep [option] PATTERN [FILES]`

一个选项，一个要匹配的模式和要搜索的文件。

如果没有提供文件名，则grep命令将搜索标准输入。

| 选项 | 含义                                             |
| ---- | ------------------------------------------------ |
| -c   | 输出匹配行的数目，而不是输出匹配的行             |
| -E   | 启用扩展表达式                                   |
| -h   | 取消每个输出行的普通前缀，即匹配查询模式的文件名 |
| -i   | 忽略大小写                                       |
| -l   | 只列出包含匹配行的文件名，而不输出真正的匹配行   |
| -v   | 对匹配模式取反，即搜索不匹配行而不是匹配行       |

例子：

```bash
$ grep in word.txt
When shall we three meet again. In thunder,lightning,or in rain?
I come,Graymalkin!
```

```bash
$ grep -c in words.txt words2.txt
words.txt:2
words2.txt:14
```

```bash
$ grep -c -v in words.txt words2.txt
words.txt:9
words2.txt:16
```

第一个例子未使用选项，只是在文件words.txt中搜索字符产in，然后输出匹配的行。文件名未输出是因为只在一个文件中进行搜索。

第二个例子在两个不同的文件中计算匹配行的数目。文件名被输出。

第三个例子使用-v选项对搜索取反，在两个文件中计算不匹配行的数目。

---

#### 正则表达式

| 字符 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| ^    | 指向一行的开头                                               |
| $    | 指向一行的结尾                                               |
| .    | 任意单个字符                                                 |
| [ ]  | 方括号内包含一个字符范围，其中任何一个字符都可以被匹配，例如字符范围a~e，或在字符范围前面加上^符号表示使用反向字符范围，即不匹配指定范围内的字符 |

如果想要将上述字符用作普通字符，就需要在它们前面加上\字符。\\$

在方括号中还可以使用一些有用的特殊匹配模式：

| 匹配模式   | 含义                     |
| ---------- | ------------------------ |
| [:alnum:]  | 字母与数字字符           |
| [:alpha:]  | 字母                     |
| [:ascii:]  | ASCII字符                |
| [:blank:]  | 空格或制表符             |
| [:cntrl:]  | ASCII控制字符            |
| [:digit:]  | 数字                     |
| [:graph:]  | 非控制、非空格字符       |
| [:lower:]  | 小写字母                 |
| [:print:]  | 可打印字符               |
| [:punct:]  | 标点符号字符             |
| [:space:]  | 空白字符，包括垂直制表符 |
| [:upper:]  | 大写字母                 |
| [:xdigit:] | 十六进制数字             |

如果指定了用于扩展匹配的-E选项，哪些用于控制匹配完成的其他字符可能会遵循正则表达式的规则：

| 选项  | 含义             |
| ----- | ---------------- |
| ?     | 0次或1次         |
| *     | 0次或多次        |
| +     | 1次或多次        |
| {n}   | n次              |
| {n,}  | n次或n次以上     |
| {n,m} | n到m次，包括n和m |

例子：

`$ grep e$ words2.txt`找以e结尾的行

`$ grep a[[:blank:]] words2.txt`查找以字母a结尾的单词。

`$ grep Th.[[:space:]] words2.txt`查找以Th开头的由3个字母组成的单词。

`$ grep -E [a-z]\{10\} words2.txt`用扩展模式来搜索只有10个字符长的全部由小写字母组成的单词。



### 2.6 命令的执行

1. $(command)
2. \`command\`    反引号

#### 算数扩展

expr命令执行起来相当慢，因为它需要调用一个新的shell来出来expr命令。一种更新更好的办法是使用$((...))扩展

- 两对圆括号用于算数替换，而一对圆括号用于命令的执行和获取输出。

#### 参数扩展

```bash
#!/bin/sh

for i in 1 2
do
	my_secret_process ${i}_tmp
done
```

用于在变量后附加额外的字符。

| 参数扩展          | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| ${param:-default} | 如果parm为空，就把它设置为default的值                        |
| ${param:=bar}     | 检查变量foo是否存在且不为空，如果它不为空，就返回它的值，否则就把变量foo赋值为bar并返回这个值 |
| ${#param}         | 给出param的长度                                              |
| ${param%word}     | 从parm的尾部开始删除与word匹配的最小部分，然后返回剩余部分   |
| ${param%%word}    | 从parm的尾部开始删除与word匹配的最长部分，然后返回剩余部分   |
| ${param#word}     | 从parm的头部开始删除与word匹配的最小部分，然后返回剩余部分   |
| ${param##word}    | 从parm的头部开始删除与word匹配的最长部分，然后返回剩余部分   |

```
#!/bin/sh

unset foo
echo ${foo:-bar}    # bar

foo=fud
echo ${foo:-bar}   # fud

foo=/usr/bin/X11/startx
echo ${foo##*/}    # usr/bin/X11/startx
echo ${foo##*/}    # startx

bar=/usr/local/etc/local/networks
echo ${bar%local*}   # /sur/local/etc 
echo ${bar%%local*}   # /usr

exit 0
```

### 2.7 调试脚本程序

| 命令行选项      | set选项                    | 说明                                   |
| --------------- | -------------------------- | -------------------------------------- |
| sh -n \<script> | set -o noexec<br />set -n  | 只检查语法错误，不执行命令             |
| sh -v \<script> | set -o verbose<br />set -v | 在执行命令之前回显它们                 |
| sh -x \<script> | set -o xtrace<br />set -x  | 在处理完命令之后回显它们               |
| sh -u \<script> | set -o nounset<br />set -u | 如果使用了未定义的变量，就给出出错消息 |

用-o选项启用set命令的选项标志，用+o选项取消设置。