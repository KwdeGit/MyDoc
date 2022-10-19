# 文件格式

1、【强制】编码为UTF-8无BOM格式

2、【强制】回车换行风格为UNIX风格

3、【强制】以缩进来区分代码块，缩进为4个空格，不得使用制表符

4、【强制】选择bash来执行脚步，即文件第一行为#！/bin/bash

# 命名规约

## 脚本名

1、【强制】脚本均以帕斯卡方式命名，并有.sh后缀

```
ey:WangYou.sh
```

## 脚步结构

1、【强制】为便于确认执行入口，脚步的执行入口函数必须为main

ey:

```
#!/bin/bash
function main()
{

}
main
```

2、【强制】除全局变量的定义和main函数的调用外，不得有其他代码位于函数体外

ey:

```
#!/bin/bash
function main()
{
	if [[ $1 == 'param' ]];then
		do thing
	fi
}
main
反例：
#!/bin/bash
if [[ $1 == 'param' ]];then
	do thing
fi
```

## 函数名

1、【强制】函数名以驼峰方式命名

ey:

```
findName()
{

}
```

2、【推荐】函数以function修饰，并在同一个脚步内风格一致

ey:

```
function findName()
{

}
```

## 变量名

1、【强制】全局变量以大写加下划线方式命名

```
ey:LIST_DATA
```

2、【推荐】全局变量采用模块相关前缀，防止脚步被包含时，全局变量被污染

3、【强制】局部变量以小写加下划线方式命名

```
ey:loop_count=10
```

4、【推荐】局部变量以local修饰，并在同一脚步内保持风格一致

## 注释规约

1、【强制】在脚本头部对脚本功能及使用方法的注释说明

ey:

```
# @brief
# @param $1
# @param $2
# @return 
```

2、【推荐】对功能复杂的函数增加注释，并详细介绍函数功能和入参

3、【强制】函数注释风格如下

ey:

```
# @brief
# @param $1
# @param $2
# @return 
function loop()
{
}
```

4、行注释在所需注释代码的上一行

正例：

ey:

```
#输出信息到屏幕
echo "hello world"
反例：
echo "hello world" #输出信息到屏幕
```

## 变量定义与使用

1、【推荐】全局变量定义在脚本功能注释之后，函数定义之前

2、【推荐】定义常量变量时以readonly修饰

```
ey:readonly PATH=/home
```

3、【强制】不得直接使用$1 $2操作，需要赋值给明确含义的命名变量

ey:

```
function log()
{
	local level="$1"
	local mod="$2"
	local mes="$3"
	echo "${level}" "${mod}" "${mes}"
}
反例：
function log()
{
	echo "$1" "$2" "$3"
}
```

4、【推荐】除shell特殊变量如$1 $2 $3 $@ $#外，其他变量取值时都加大括号，即增加可读性，又减少错误

5、【强制】local修饰的局部变量在执行命令结果时，将声明和赋值分开，如果放在同一行，$?表示赋值是否成功，并不是命令执行退出码

## 其他

1、【推荐】包含其他脚本时，使用source而非.。本质上市一样的，但是source更可读，减少因书写疏忽导致的错误

ey:

```
source ./Common.sh
反例：
#因书写疏忽少写了.，使得Common是在子shell中执行，someFunction无法被调用
./Common.sh
#调用Common中的function
someFunction
```

2、【推荐】参数为空或不存在时，初始化为默认值

ey：

```
function myFunctin()
{
	#参数$1不存在时，foo默认值为1
	#参数$2不存在时，bar默认值为2
	local foo={1:-1}
	local bar={2:-2}
	echo "foo:${foo},bar:${bar}"
}
myFunction
```

3、【推荐】文件路径尽量采用相对于脚步自身路径

4、【推荐】执行子命令时使用$(command)，反引号是老用法不推荐

ey:

```
# var是'\'
var=$(echo \\)
反例：
# var是'\'，需要3个'\'才能得到正确字符，与其他场景下使用习惯不同
var='echo \\\\'
```

## 经验总结

**ssh执行命令**

ssh命名最好增加双引号或单引号，防止命令在本地执行。

ey:

```
ssh root@192.168.1.0 "source /etc/profile;ls -l /home"
反例：
#ls命令实际在本地执行，并输出结果，并不能获得远程的信息
ssh root@192.168.1.0 source /etc/profile;ls -l /home
```

**ssh命令卡死**

sh执行脚本时，内部启动不会退出的子进程，导致ssh命令无法返回退出，一般加上超时时间

ey:

```
#shell进程nohup放后台，是输入输出重定向
ssh root@192.168.110.105 "nohup sh -c 'sleep 5' >/dev/null 2>&1 &"
反例：
#shell命令后台执行
ssh root@192.168.100.105 "sh -c 'sleep 100 &'"
#shell进程后台执行
ssh root@192.168.100.105 "sh -c 'sleep 100 '&"
#shell进程nohup放后台，但是输入输出未重定向
ssh root@192.168.110.105 "nohup sh -c 'sleep 5'&"
```

**单引号和双引号**

shell脚步中，双引号会对内容进行转义（比如变量进行解析），单引号不会。

ey:

```
ssh root@192.168.1.0 'grep shell.log | awk "{print $3}"'
反例：
#$3会被解析成脚本或者函数的参数，并不能作为awk分隔的第三个参数
ssh root@192.168.1.0 "grep shell.log | awk '{print $3}'"
```

**shell启动子进程**

1、shell中使用反引号（``)或者$()执行命令时，后台默认启动子进程执行，并且子进程名称与当前执行脚步名称一样，在过滤sh脚步时，需要特别注意

2、shell脚步进程名一般是sh，并不能通过pidof获取执行shell脚步的进程id，需要通过ps命令过滤脚步名称

ey:

```
#脚步文件：test.sh
function test()
{
	#可以看到有两个进程名称test.sh进程，但是进程id不同
	`ps axu | grep test.sh`
}
```

**grep全字匹配**

shell脚步内grep过滤信息是，尽量使用全字匹配，保证精确匹配，不会过滤多行

```
#文本文件，dn.stat
Status=Normal
diskStatus=1
#精确匹配一行
grep -w Status dn.stat
grep "^Status=" dn.stat
```

