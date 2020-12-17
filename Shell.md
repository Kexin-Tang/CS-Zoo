# 目录
- [简介](#简介)
  - [运行shell的方法](#运行shell的方法)
- [Shell变量](#shell变量)
  - [定义变量](#定义变量)
  - [使用变量](#使用变量)
  - [只读变量](#只读变量)
  - [删除变量](#删除变量)
  - [变量类型](#变量类型)
- [Shell字符串](#shell字符串)
  - [单引号](#单引号)
  - [双引号](#双引号)
  - [拼接字符串](#拼接字符串)
  - [获取字符串长度](#获取字符串长度)
  - [获取子字符串](#获取子字符串)
  - [查找子字符串](#查找子字符串)
- [Shell数组](#shell数组)
  - [读取数组](#读取数组)
  - [数组长度](#数组长度)
- [Shell注释](#shell注释)
  - [单行注释](#单行注释)
  - [多行注释](#多行注释)
- [Shell传参](#shell传递参数)
- [Shell基本运算符](#shell基本运算符)
  - [算术运算符](#算术运算符)
  - [关系运算符](#关系运算符)
  - [布尔运算符](#布尔运算符)
  - [字符串运算符](#字符串运算符)
  - [文件测试运算符](#文件测试运算符)
- [echo命令](#echo命令)
- [printf命令](#printf命令)
- [流程控制](#流程控制)
  - [if-else](#if-else)
  - [for循环](#for循环)
  - [while循环](#while循环)
  - [无限循环](#无限循环)
  - [until循环](#until循环)
  - [case](#case)
- [Shell函数](#shell函数)
- [Shell输入/输出重定向](#shell输入输出重定向)
  - [重定向命令列表](#重定向命令列表)
  - [深入重定向](#深入重定向)
  - [Here Document](#here-document)
  - [/dev/null文件](#devnull文件)
- [Shell文件包含](#shell文件包含)

# 简介

* 可以认为bash == shell

* 在shell文件的头部写上`#!/bin/bash`表明改文件为bash

* `echo "content"`用于向控制台输出内容

## 运行shell的方法
* 作为可执行程序
```bash
chmod +x ./test.sh 	# 使脚本有执行权限
./test.sh 			# 执行脚本(注意，此处一定不能省掉./)
```
* 作为解释器参数
```bash
sh test.sh
```

# Shell变量

## 定义变量
* 定义时不用加`$`。注意，变量名和等号之间**不能有空格**，同时要遵循以下的格式要求
	* 命名只能全英文，数字，下划线
	* 首个字符不能是数字
	* 不能使用标点符号
	* 不能使用bash关键词
```bash
name="shuaibi"
```

## 使用变量
* 使用一个定义过的变量只需要加美元`$`
```bash
name="shuaibi"
echo ${name}
```

* 花括号可选，但是强烈建议带上，以区分变量边界
```bash
# 如果不使用{}，那么就会输出4次"I am good at $skillScript"
# 而使用了{}后，程序知道在{}的为for的变量
for skill in Ada Coffe Action Java; do
	echo "I am good at ${skill}Script"
done
```

* 重新定义变量
```bash
name="shuaibi"
echo ${name}
name="dashuaibi"
echo ${name}
```

## 只读变量
* 使用`readonly`可以将变量定义为只读，不可更改
```bash
name="shuaibi"
readonly name
# name="dashuaibi",若执行则报错
```

## 删除变量
* 使用`unset`即可以删除变量*（unset不能删除只读变量）*

## 变量类型
1. **局部变量** 在脚本或者命令中定义，仅在当前shell中有效
2. **环境变量** 所有的程序，包括shell启动的程序，都能访问。可以通过shell来定义环境变量
3. **shell变量** shell变量是由shell设置的特殊变量。

# Shell字符串

## 单引号
```bash
str='this is a string'
```
单引号字符串的限制:
* 任何字符都按照原样输出，变量是无效的
* 单引号中不能出现单独的单引号（即使是转义形式也不可以），但是可以出现成对的单引号（但是要注意，成对单引号相当于把字符串进行了拆分）
```bash
str1='i am robot'		# ok
# str2='i am \' robot'	# 相当于'i am \'+ robot'(undefined)
# str3='i 'am' 'robot'	# 相当于'i'+am(undefined)+'robot'
```

## 双引号
```bash
name='noob'
str="hello \"${name}\"!"
echo ${str}	# 输出为 hello "noob"!
```
双引号字符串Vs单引号字符串
* 可以有变量
* 可以使用转义字符

## 拼接字符串
```bash
your_name="runoob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1
# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3
```

## 获取字符串长度
```bash
str='abcd'
echo ${#str}	# 4
```

## 获取子字符串
```bash
str='abcdefg'
echo ${str:1:3}	# bcd，注意字符串从0开始，且此处str:a:b是取[a, b]
```

## 查找子字符串
查找字符a或f的位置（哪个先出现就输出哪个）
```bash
str1="i am tangkexin ffff"
str2="ffff nixekgnat ma i"

echo `expr index "$str1" af`	# 3
echo `expr index "$str2" af`	# 1
```
**注意代码中不是`单引号'`**

# Shell数组
bash支持一维数组（不支持多维），并且没有大小限制，其下标从0开始

```bash
数组名=(值1 值2 ...)
```

可以使用不连续的下标，且下标没有范围限制

## 读取数组
```bash
value=${arr[idx]}
```

使用`@`可以获得所有元素

```bash
echo ${arr[@]}
```

## 数组长度
```bash
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```

# Shell注释
## 单行注释
使用`#`用于注释

## 多行注释
使用如下格式(EOF可以换成别的非关键字内容)
```bash
:<<EOF
content
EOF

:<<!
content
!
```

# Shell传递参数
在执行Shell脚本时可以向其中传入参数，格式为`$n`其中n为数字，代表命令的第n个参数
```bash
#!/bin/bash

echo "Shell的名字: $0"
echo "第一个参数  : $1"
```

```bash
sh test.sh 100
# 输出如下
# Shell的名字: test.sh
# 第一个参数  : 100 
```

其他特殊字符

参数 | 说明
:---: | :---:
$# | 传递到脚本的参数个数
$$ | 脚本运行的进程ID号
$! | 后台运行的最后一个进程的ID号
$- | 显示Shell使用的当前选项
$? | 显示最后命令的退出状态，0表示未出错
$* | 以一个单字符串显示所有传到shell的参数
$@ | 类似$*,但是输出每个参数，每个参数为一个独立的字符串


`$*` Vs `$@`

只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则`$*`等价于 "1 2 3"（传递了一个参数），而`$@`等价于 "1" "2" "3"（传递了三个参数)
```bash
#!/bin/bash
echo "Shell的参数个数   : $#"

echo "Shell的参数(使用*)"
for i in "$*"; do
	echo $i
done

echo "Shell的参数(使用@)"
for i in "$@"; do
	echo $i
done
```

得到的输出
```bash
sh test.sh 10 200
# Shell的参数个数   : 2
# Shell的参数(使用*)
# 10 200
# Shell的参数(使用@)
# 10
# 200
```

# Shell基本运算符
Shell支持多种运算符，包括：
* 算术
* 关系
* 布尔
* 字符串运算
* 文件测试

原生的bash并不支持简单的数学运算，但是可以通过其他命令来实现，例如`awk`和`expr`。

`expr`是表达式计算工具，能完成表达式求值，使用\`来实现
```bash
val=`expr 2 + 5`
echo $val 		# 7
```

**注意**：表达式和运算符之间要有空格，如`2+2`是错误的，要写成`2 + 2`

## 算术运算符
运算符 | 说明 | 举例
:---: | :---: | :---:
\+ | 加法 | `expr $a + $b`
\- | 减法 | `expr $a - $b`
\* | 乘法**(一定要使用\\来标注)** | `expr $a \* $b`
/ | 除法 | `expr $a / $b`
% | 取余 | `expr $a % $b`
= | 赋值 | a=$b
== | 判断相等 | [ $a == $b]
!= | 判断不等 | [ $a != $b]

## 关系运算符
关系运算符只支持数字，不支持字符串，除非字符串的值是数字

运算符 | 说明 | 举例
:---: | :---: | :---:
\-eq | 检测是否相等 | [$a \-eq $b]
\-ne | 检测是否不相等 | [$a \-ne $b]
\-gt | 检测左侧>右侧 | [$a \-gt $b]
\-lt | 检测左侧<右侧 | [$a \-lt $b]
\-ge | 检测左侧>=右侧 | [$a \-ge $b]
\-le | 检测左侧<=右侧 | [$a \-le $b]

## 布尔运算符
运算符 | 说明 | 举例
:---: | :---: | :---:
! | 非 | [! $a]
\-o | 或 | [$a -lt 20 \-o $b -ge 10]
\-a | 与 | [$a -lt 20 \-a $b -gt 10]

## 逻辑运算符
运算符 | 说明 | 举例
:---: | :---: | :---:
&& | 与 | [$a -lt 20 && $b -ge 10]
\|\| | 或 | [$a -lt 20 \|\| $b -ge 10]

## 字符串运算符
运算符 | 说明 | 举例
:---: | :---: | :---:
= | 字符串是否一样 | [$a = $b]
!= | 字符串是否不同 | [$a != $b]
\-z | 字符串是否长度为0 | [\-z $a]
\-n | 是否长度不为0,即是否存在 | [\-n $a]
$ | 字符串是否为空 | [$a]

## 文件测试运算符
运算符 | 说明 | 举例
:---: | :---: | :---:
\-b | 检测文件是否为块设备文件 | [\-b $file]
\-c | 检测文件是否为字符设备文件 | [\-c $file]
\-d | 检测文件是否为目录 | [\-d $file]
\-f | 检测文件是否为普通文件(非目录&&非设备文件) | [\-f $file]
\-r | 检测文件是否可读 | [\-r $file]
\-w | 检测文件是否可写 | [\-w $file]
\-x | 检测文件是否可执行 | [\-x $file]
\-s | 检测文件是否为空 | [\-s $file]
\-e | 检测文件是否存在 | [\-e $file]

# echo命令
* 显示普通字符串
```bash
echo "I am shuaibi"
echo I am shuaibi
```

* 显示转义字符
```bash
echo "I am \"shuaibi\""
```

* 显示变量——`read`从输入中读取一行
```bash
read name
echo "${name} is input"
```

* 换行——使用`-e`开启转义
```bash
echo -e "I am fine \n"
```

* 不换行——使用`\c`关闭换行，使下一句语句输出在该句后面而非下面
```bash
echo -e "I \c"
echo "am fine"
```

* 定向到文件
```bash
echo "I am fine" > test.txt
```

* 显示命令结果
```bash
echo `date`
```

# printf命令
与C/C++中printf基本一致，常见格式如下
```bash
printf "%d %f" {10} {10.00} 
```

# 流程控制

## if-else

* if的语法
```bash
if condition
then
	...
fi
# 也可以缩减到一行
if condition; then ...; fi
```

* if else
```bash
if condition
then
	...;
else
	...;
fi
```

* if elif else
```bash
if condition1
then
	...;
elif condition2
then
	...;
else
	...;
fi
```

## for循环
```bash
for var in item1 item2 ...
do
	...;
done
```

也可以缩写成一行
```bash
for var in item1 item2 ...; do ...; done;
```

还可以写成类似于C语言的形式
```bash
for((assignment; condition; next));do
	command;
done
```

**注意**：for(())内不需要使用$来标识变量，但是command里要使用$

## while循环
```bash
int=1
while((${int}<=5))
do
	echo $int
	let "int++"
done
```

## 无限循环
```bash
while :
do
	...;
done
```
```bash
while true
do
	...;
done
```
```bash
for((;;))
do
	...;
done
```

## until循环
```bash
until condition
do
	...;
done
```
与while相反，即condition为false时才继续循环

## case
```bash
case ${...} in
	mode1)
		...;
		;;
	mode2)
		...;
		;;
	*)
		...;
		;;
esac 
```

case工作方式如上所示。取值后面必须为单词in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

# Shell函数
函数定义的格式
```bash
[function] function_name [()]
{
	action;
	[return int;]
}
```

示例(`$?`必须紧跟`fun`调用后面，否则无法获得return的值)
```bash
fun()
{
	echo "第一个参数: ${1}"
	echo "第二个参数: ${2}"
	return `expr ${1} + ${2}`
}

fun 1 2
echo "两个数的和为: $?"
```

# Shell输入/输出重定向
## 重定向命令列表

命令 | 说明
:---: | :---: 
command > file | 输出重定向到file
command < file | 输入重定向到file
command >> file | 输出重定向（追加）
n > file | 将文件描述符为n的文件重定向到file
n >> file | 将文件描述符为n的文件重定向（追加）
n >& m | 将输出文件m和n进行合并
n <& m | 将输入文件m和n进行合并
<< tag | 将开始标记tag和结束标记tag之间的内容作为输入

**注意**：文件描述符——0(标准输入STDIN),1(标准输出STDOUT),2(标准错误输出STDERR)

## 深入重定向

* 当不写出文件描述符时，默认为标准输入/输出

* 将错误信息重定向到file
```bash
command 2>file
```

* 将标准输出和错误信息一同放入file
```bash
# >file表示将标准输出(1)定向到file，然后2>&1代表将标准错误(2)和标准输出(1)放到一起(即file)
command >flie 2>&1
```

* 对输入/输出都进行重定向
```bash
# 从file1中读取标准输入，然后将输出放到file2
command < file1 >file2
```

## Here Document
Here Document是一种特殊的重定向方式，用来将输入重定向到一个交互式Shell脚本

```bash
command << delimiter
	document
delimiter
```

它的作用是将两个delimiter间的内容(document)作为输入传递给command

**注意**：
1. 结尾的delimiter要顶格，前后都不能有字符，如空格和tab
2. 开始的delimiter前后的空格会被忽略

###### 示例
* 使用`wc -l`输出行数
```bash
wc -l << EOF
	"I am"
	"a document of"
	"3 rows"
EOF
```

## /dev/null文件
执行命令但是不在终端中显示输出，那么可以重定向到/dev/null中
```bash
# 不显示错误信息和输出信息
command > /dev/null 2>&1
```

# Shell文件包含
Shell可以包含外部脚本。这样可以封装共用代码作为独立文件。
```bash
source filename

. filename
```

###### 示例
有如下两个.sh
```bash
#!/bin/sh
# test1.sh

url="www.google.com"
```

```bash
#!/bin/sh
# test2.sh

. test1.sh
# 或者使用source test1.sh

echo "网址: ${url}"	# 网址: www.google.com
```
