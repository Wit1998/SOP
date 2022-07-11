# shell

操作linux运行的方式

- GUI
- CLI （comman line interface）

```bash
# 指的是告诉内核该用什么方式来解析文本中的内容
# 如果文件名以xx.sh的方式命名，会高亮。原因是系统会知道是脚本需要解析
#! /usr/bin/bash


# shell对缩进无要求
```

 

```bash
#执行脚本时
#绝对路径
/xx/xx/xxx.sh
#相对路径(当前路径下的xxx文件)
./xxx.sh
#可以在profire文件提前声明脚本位置，就可以直接使用该名称，而非路径名称

# 解析脚本的具体操作
bash -x xxx.sh
```

## 自定义变量

```bash
#定义的方式
``
$()

# 在grep时
使用^xxx，可以过滤以xxx开头的内容
使用xxx$，可以过滤以xxx结尾的内容
使用 .  可以正则过滤任意字符
```

## 特殊变量

```bash
#查看当前sh的进程id
echo $$

#返回上一个名字执行的值，正确是0，错误就报非0值
echo $?
如 if [ $? == 0]

# 都表示传给脚本的所有参数(如你传了三个数字或文件，)
echo $@ $*
如输入 xxxx.sh test1 test2 test3 ,$@和$*的结果为test1 test2 test3

# 传给脚本参数的数量(一般用作传变量数量的判断)
echo $#
如 if [[ $# != 2 ]]

# 表示脚本文件的执行路径，大多数情况会列出脚本的名字(要区分以绝对路径还是相对路径执行脚本)
echo $0
#传给脚本第一、第二个参数
echo $1 $2 
```

## 环境变量

```


PS1 表示终端颜色

```

## 条件判断if

```


使用 '\' 可以换行
-f 判断文件是不是存在
```

## 命令连接符

```bash
; 表示根据先后顺序执行完所有的操作，无论操作正确还是错误。ru'g
&& 与 前面必须正确，才会执行后面的
|| 或 前面的正确则不执行后面的，前面错误则执行后面的
```

条件分支

```
read xxx
case $xxx in
	1) echo 'you select 1'
	;;
	2) echo 'you select 2'
	;;
	3) echo 'you select 3'
	;;
	4) echo 'you select 4'
	;;
	*) echo 'you do not select a number between 1 to 4'
	;;
esac
```

## for循环

```bash
for i in {1..11}
do 
xx
done 
# shell脚本的并行，如ping ip的时,每次ping失败都会等待很久时间，这时使用&可以并行执行
ping 10.8.0.$i -c 1 &>> /dev/null && echo "$i up" || echo "$i down" &
```

## while循环

```bash
COUNTER=0
while [ $COUNTER -lt 5 ]
do 
	COUNTER = `expr $COUNTER+1`
	echo $COUNTER
done


# 输出颜色
#绿色
echo -e "\033[32m up \033[0m"
#红色
echo -e "\033[31m down \033[0m"

#eq(equal) 等于
#lt(less than) 小于
#le(less equal) 小于等于
#gt(greater than) 大于
#ge(greater equal) 大于等于
```

## until循环(交互式shell)

```bash
echo " 1) YES"
echo " 2) NO"
# 直到选择1或者2时，才输出
# 要么以1开头然后结尾，要么以2开头然后结尾
until [[ "$choice" =~ ^[1-2]$ ]]
do
#打印choice [1-2]: ，并抓取终端输出的结果并赋值给choice
	read -rp "choice [1-2]: " -e -i 1 choice
done


#与while相反
#while是条件语句满足条件时，进入循环。
#until时条件语句满足条件时，退出循环。


```

函数

```bash
#编写函数
vim fun.inc
function check()
{
	if [ "$?" == 0]
	then
		echo -e "\33[32m${1}=======>success\33[0m"
		echo -e "\33[31m${1}=======>fail\33[0m"
	fi
}

#调用函数文件
source ./fun.inc
check

```

