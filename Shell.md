# 目录
- [简介](#简介)
  - [运行shell的方法](#运行shell的方法)
- [Shell变量](#shell变量)

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
