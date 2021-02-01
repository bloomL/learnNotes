#### 入门

##### 什么是Bash

是许多 Linux 平台默认使用的 **shell**

**shell** 是一个命令解释器，是介于操作系统内核与用户之间的一个绝缘层。准确地说，它也是能力很强的计算机语言，被称为解释性语言或脚本语言

批量删除sh脚本

```shell
ls | grep sh | xargs rm
```



###### 编写Bash脚本

```shell
vim hello.sh

#!/bin/bash
# This is a comment
echo Hello World
# 使用重定向 ,运行脚本后会生成text文件
echo "Hello World !" > test.txt
```

**#!** 是说明 hello 这个文件的类型。Linux 系统根据 #! 及该字符串后面的信息确定该文件的类型。

第一行的 **#! 及后面的 /bin/bash** 就表明该文件是一个 **BASH 程序**，需要由 /bin 目录下的 bash 程序来解释执行。

第二行的 # This is a ... 就是 BASH 程序的**注释**，在 BASH 程序中从 **# 号（注意：后面紧接着是 ! 号的除外）开始到行尾的部分均被看作是程序的注释**。

第三行的 echo 语句的功能是把 **echo** 后面的字符串**输出**到标准输出中去。需要注意的是 BASH 中的绝大多数语句结**尾处都没有分号**。

###### 运行Bash脚本

```shell
# 使用shell来执行
sh hello.sh

# 使用bash来执行
bash hello.sh

# 使用.来执行
. ./hello.sh

# 使用source来执行
source hello.sh

# 还可以赋予脚本所有者执行权限，允许该用户执行该脚本
chmod u+rx hello.sh
./hello.sh
```

脚本清除 /var/log 下的 log 文件

**/dev/null** 可以理解为一个黑洞，里面是空的

```shell
vim cleanlogs.sh
#!/bin/bash
# 初始化变量
LOG_DIR=/var/log
cd $LOG_DIR
cat /dev/null > xxx.log
echo "Logs cleaned up."
exit

# 执行脚本
sudo sh cleanlogs.sh
# 授予权限，再执行
sudo chmod +x cleanlogs.sh
sh cleanup.sh
```

练习

```shell
# 新建一个 test.sh 输出 Hello Shiyanlou!
#!/bin/bash
echo "Hello Shiyanlou!"
# 复制 test.sh 为 test2.sh
cp test.sh test2.sh
cp /home/shiyanlou/test{,2}.sh     # ????
# 修改 test2.sh 实现将 Hello Shiyanlou 保存为 my.txt 文本
#!/bin/bash
echo "Hello Shiyanlou!" > my.txt
# 新建一个 cleantest.sh 脚本运行实现清空 test.sh 里的内容
#!/bin/bash
cat /dev/null > test.sh
```

#### 特殊字符

##### 注释（#）

行首以**#** 开头(**除#!之外**)的是注释。**#! **是用于指定当前脚本的解释器，我们这里为 bash，且应该指明完整路径，所以为 /bin/bash。

##### 分号（;）

使用分号**；** 可以在同一行上写两个或两个以上的命令

```shell
#!/bin/bash
echo hello; echo there
filename=ttt.sh
# 注意: "if"和"then"需要分隔，-e用于判断文件是否存在
if [ -e "$filename" ]; then
    echo "File $filename exists."; cp $filename $filename.bak
else
    echo "File $filename not found."; touch $filename
fi; echo "File test complete."
```

终止 case 选项（双分号）

```shell
#!/bin/bash

var=b

case "$varname" in
    [a-z]) echo "abc";;
    [0-9]) echo "123";;
esac
```

**分支**

**if  then**

```shell
if [ 条件判断式一 ]; then
    当条件判断式一成立时，执行的指令
elif [ 条件判断式二 ]; then
    当条件判断式二成立时，执行的指令
else
    当条件判断式一和二都不成立时，执行的指令
fi
```

**case ... esac**

```shell
case $变量名称 in
    "第一个变量内容")
        指令
        ;;
    "第二个变量内容")
        指令
        ;;
    *)
        不包含第一个变量内容和第二个变量内容时执行的指令
        exit 1
        ;;
esac
```

##### 点号（.）

等价于 **source** 命令

bash 中的 source 命令用于在当前 bash 环境下读取并执行 FileName.sh 中的命令。

```shell
source test.sh

. test.sh
```

##### 引号

**双引号（")**
"STRING" 将会阻止（解释）STRING 中**大部分**特殊的字符

**单引号（'）**
'STRING' 将会阻止 STRING 中**所有**特殊字符的解释

![image-20210129102414322](C:\Users\l'g\AppData\Roaming\Typora\typora-user-images\image-20210129102414322.png)

##### 斜线和反斜线

**斜线（/）**
文件名**路径分隔符**。分隔文件名不同的部分（如 /home/bozo/projects/Makefile）。也可以用来作为**除法**算术操作符。注意在 linux 中表示路径的时候，许多个 / 跟一个 / 是一样的。/home/shiyanlou 等同于 ////home///shiyanlou。

**反斜线（\）**
一种对单字符的引用机制。\X 将会“转义”字符 X。这等价于"X"，也等价于'X'。\ 通常用来**转义双引号（"）和单引号（'）**，这样双引号和单引号就**不会被解释成特殊含义**

符号 说明
\n 	表示新的一行
\r 	表示回车
\t 	表示水平制表符
\v 	表示垂直制表符
\b 	表示后退符
\a 	表示"alert"(蜂鸣或者闪烁)
\0xx 	转换为八进制的 ASCII 码, 等价于 0xx
\\"  	表示引号字面的意思
转义符也提供续行功能，也就是**编写多行命令**的功能。每一个单独行都包含一个不同的命令，但是每行结尾的转义符都会转义换行符，这样下一行会与上一行一起形成一个命令序列。

##### 反引号（`）

反引号中的命令会**优先执行**

```shell
# 先创建 back 目录，然后复制 test.sh 到 back 目录
cp `mkdir back` test.sh back
```

##### 冒号（:）

**空命令**
等价于“NOP”（no op，一个什么也不干的命令）。也可以被认为与 shell 的**内建命令 true** 作用相同。“:”命令是一个 bash 的内建命令，它的退出码（exit status）是（0）。

```shell
#!/bin/bash
while :
do
    echo "endless loop"
done
等价于
#!/bin/bash
while true
do
    echo "endless loop"
done
```

**变量扩展/子串替换**
在与 > 重定向操作符结合使用时，将会把一个文件**清空**，但是并不会修改这个文件的权限。如果之前这个文件并不存在，那么就创建这个文件。

```shell
: > test.sh   # 文件“test.sh”现在被清空了
# 与 cat /dev/null > test.sh 的作用相同。然而并不会产生一个新的进程, 因为“:”是一个内建命令

# 与 >> 重定向操作符结合使用时，将不会对预先存在的目标文件产生任何影响。如果这个文件之前并不存在，那么就创建它。
: >> target_file
```

##### 问号（?）

测试操作符
在一个双括号结构中，? 就是 C 语言的三元操作符

```shell
vim test.sh

#!/bin/bash
a=10
((t=a<50?8:9))
echo $t
```

##### 美元符号（$）

变量替换

##### 小括号（( )）

**命令组**
在括号中的命令列表，将会作为一个**子 shell** 来运行。

在括号中的变量，由于是在子 shell 中，所以对于脚本剩下的部分是不可用的。父进程，也就是脚本本身，将不能够读取在子进程中创建的变量，也就是在子 shell 中创建的变量。像是一个**局部变量**。

```shell
vim test.sh

#!/bin/bash
a=123
	(a=321;)
echo "$a"

```

**初始化数组**

```shell
vim test.sh

#!/bin/bash

arr=(1 2 6 8 9)
echo ${arr[3]}

# 输出8

```

##### 大括号（{ }）

**文件名扩展**,大括号中，不允许有空白，除非这个空白被引用或转义。

复制 t.txt 的内容到 t.back 中

```shell
vim test.sh

#!/bin/bash
if [ ! -w 't.txt' ];
then
    touch t.txt
fi
echo 'test text' >> t.txt
cp t.{txt,back}
```

**代码块**

代码块，又被称为内部组，这个结构事实上创建了一个匿名函数（一个没有名字的函数）。然而，与“标准”函数不同的是，在其中声明的变量，对于脚本其他部分的代码来说还是可见的。

```shell
vim test.sh

#!/bin/bash

a=123
{ a=321; }
echo "a = $a"
```

##### 中括号（[ ]）

**条件测试**
条件测试表达式放在 **[ ]** 中

```shell
vim test24.sh

#!/bin/bash

a=5
if [ $a -lt 10 ]
then
    echo "a: $a"
else
    echo 'a>=10'
fi
```

**数组元素**
在一个 array 结构的上下文中，中括号用来引用数组中每个元素的编号

```powershell
#!/bin/bash

arr=(12 22 32)
arr[0]=10
echo ${arr[0]}
```

##### 尖括号（< 和 >）

**重定向**
test.sh > filename：重定向 test.sh 的输出到文件 filename 中。如果 filename 存在的话，那么将会被覆盖。

test.sh &> filename：重定向 test.sh 的 stdout（标准输出）和 stderr（标准错误）到 filename 中。

test.sh >&2：重定向 test.sh 的 stdout 到 stderr 中。

test.sh >> filename：把 test.sh 的输出追加到文件 filename 中。如果 filename 不存在的话，将会被创建。

##### 竖线（|）

**管道**
分析前边命令的输出，并将输出作为后边命令的输入。这是一种产生**命令链**的好方法

```shell
#!/bin/bash
tr 'a-z' 'A-Z'
exit 0

ls -l | bash test26.sh
```

输出的内容均变为了**大写**字母

##### 破折号（-）

**选项，前缀**
在所有的命令内如果想使用选项参数的话,前边都要加上“-”。

**用于重定向 stdin 或 stdout**

##### 波浪号（~）

**目录**
~ 表示 home 目录。

#### 变量和参数

##### 变量定义

变量的名字就是变量保存值的地方。引用变量的值就叫做变量替换。

如果 variable 是一个变量的名字，那么 $variable 就是引用这个变量的值，即这变量所包含的数据。

**$variable 事实上只是 ${variable} 的简写形式**。在某些上下文中 $variable 可能会引起错误，这时候你就需要用 ${variable} 了。

定义变量

注意

变量名和等号之间不能有空格。同时，变量名的命名须遵循如下规则：

首个字符必须为字母（a-z，A-Z）。
中间不能有空格，可以使用下划线（_）。
不能使用标点符号。
不能使用 bash 里的关键字（可用 help 命令查看保留关键字）。

##### 使用变量

变量名前加美元符号

```shell
myname="shiyanlou"
echo $myname
echo ${myname}
echo ${myname}Good
echo $mynameGood

myname="miao"
echo ${myname}
```

加**花括号**帮助解释器**识别变量的边界**，若不加，解释器会把 mynameGood 当成一个变量（值为空）

##### 只读变量

readonly 命令可以将变量定义为**只读变量**，只读变量的值**不能被改变**

```shell
#!/bin/bash
myUrl="http://www.shiyanlou.com"
readonly myUrl
myUrl="http://www.shiyanlou.com"
```

![image-20210129164332956](C:\Users\l'g\AppData\Roaming\Typora\typora-user-images\image-20210129164332956.png)

##### 特殊变量

**局部变量**
这种变量只有在**代码块**或者**函数**中才可见

**环境变量**
这种变量将影响用户接口和 shell 的行为

**位置参数**
从命令行传递到脚本的参数：$0，$1，$2，$3...

$0 就是脚本文件自身的名字，$1 是第一个参数，$2 是第二个参数，$3 是第三个参数，然后是第四个。**$9 之后**的位置参数就必须用**大括号**括起来了，比如，${10}，${11}，${12}。

$# ： 传递到脚本的参数个数
$* ： 以一个单字符串显示所有向脚本传递的参数。与位置变量不同,此选项参数可超过 9 个
$$ ： 脚本运行的当前进程 ID 号
$! ： 后台运行的最后一个进程的进程 ID 号
$@ ： 与 $* 相同,但是使用时加引号,并在引号中返回每个参数
$： 显示 shell 使用的当前选项,与 set 命令功能相同
$? ： 显示最后命令的退出状态。 0 表示没有错误,其他任何值表明有错误

```shell
#!/bin/bash

# 作为用例, 调用这个脚本至少需要10个参数, 比如：
# bash test.sh 1 2 3 4 5 6 7 8 9 10
MINPARAMS=10

echo

echo "The name of this script is \"$0\"."

echo "The name of this script is \"`basename $0`\"."


echo

if [ -n "$1" ]              # 测试变量被引用.
then
echo "Parameter #1 is $1"  # 需要引用才能够转义"#"
fi

if [ -n "$2" ]
then
echo "Parameter #2 is $2"
fi

if [ -n "${10}" ]  # 大于$9的参数必须用{}括起来.
then
echo "Parameter #10 is ${10}"
fi

echo "-----------------------------------"
echo "All the command-line parameters are: "$*""

if [ $# -lt "$MINPARAMS" ]
then
 echo
 echo "This script needs at least $MINPARAMS command-line arguments!"
fi

echo

exit 0
```

结果

```shell
bash test30.sh 1 2 10


The name of this script is "test.sh".
The name of this script is "test.sh".

Parameter #1 is 1
Parameter #2 is 2
-----------------------------------
All the command-line parameters are: 1 2 10

This script needs at least 10 command-line arguments!
```

#### 基本运算符

##### 算数运算符

```shell
#!/bin/bash

a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a == b"
fi
if [ $a != $b ]
then
   echo "a != b"
fi
```

**expr** 使用它能完成表达式的求值操作。
**表达式**和**运算符**之间要有**空格** $a + $b 写成 $a+$b 不行
**条件表达式**要放在方括号之间，并且**要有空格 [ $a == $b ]** 写成 [$a==$b] 不行
**乘号（*）**前边必须加**反斜杠**（\)才能实现乘法运算

##### 关系运算符

只支持数字，不支持字符串，除非字符串的值是数字

-eq	两个数是否相等

-ne	两个数是否不相等

-gt	左边数是否大于右边

-lt	左边数是否小于右边

-ge	左边数是否大于等于右边

-le	左边数是否小于等于右边

```shell
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a == b"
else
   echo "$a -eq $b: a != b"
fi
```

##### 逻辑运算符

用逻辑运算符&&或者|| 连接两个表达式的时候,一个[ ]不可以,但是**[[ ]]**就可以

```shell
#!/bin/bash
a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "return true"
else
   echo "return false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "return true"
else
   echo "return false"
fi
```

##### 字符串运算符

=	两个字符串是否相等

!=	两个字符串是否相等，不相等 true

-z	字符串长度是否为0， 为0 true

-n 	字符串长度是否为0， 不为0 true

str	字符串是否为空，不为空 true

```shell
#!/bin/bash

a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a == b"
else
   echo "$a = $b: a != b"
fi
if [ -n $a ]
then
   echo "-n $a : The string length is not 0"
else
   echo "-n $a : The string length is  0"
fi
if [ $a ]
then
   echo "$a : The string is not empty"
else
   echo "$a : The string is empty"
fi
```

##### 文件测试运算符

![image-20210130153602711](C:\Users\l'g\AppData\Roaming\Typora\typora-user-images\image-20210130153602711.png)

```shell
#!/bin/bash

file="/home/shiyanlou/test.sh"
if [ -r $file ]
then
   echo "The file is readable"
else
   echo "The file is not readable"
fi
if [ -e $file ]
then
   echo "File exists"
else
   echo "File not exists"
fi
```

#### 流程控制

##### if else

```shell
if condition1
then
    command1
elif condition2
then
    command2
else
    commandN
fi
```

```shell
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo 'Two numbers are equal!'
else
    echo 'The two numbers are not equal!'
fi
```

##### for 循环

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

```shell
for str in This is a string
do
    echo $str
done
```

##### while 语句

```shell
while condition
do
    command
done
```

```shell
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
# let 命令，它用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量
```

##### 无限循环

```shell
while :
do
    command
done
或者
while true
do
    command
done
或者
for (( ; ; ))
```

##### until 循环

```shell
until condition
do
    command
done
```

##### case

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```

```shell
echo 'Enter a number between 1 and 4:'
echo 'The number you entered is:'
read aNum
case $aNum in
    1)  echo 'You have chosen 1'
    ;;
    2)  echo 'You have chosen 2'
    ;;
    3)  echo 'You have chosen 3'
    ;;
    4)  echo 'You have chosen 4'
    ;;
    *)  echo 'You did not enter a number between 1 and 4'
    ;;
esac
```

##### 跳出循环

**break**

```shell
#!/bin/bash
while :
do
    echo -n "Enter a number between 1 and 5:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "The number you entered is $aNum!"
        ;;
        *) echo "The number you entered is not between 1 and 5! game over!"
            break
        ;;
    esac
done
```

**continue**

```she
#!/bin/bash
while :
do
    echo -n "Enter a number between 1 and 5: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "The number you entered is $aNum!"
        ;;
        *) echo "The number you entered is not between 1 and 5!"
            continue
            echo "game over"
        ;;
    esac
done
```

**read** 接收键盘或其它文件描述符的输入。
read 命令格式如下：

**read [选项] [变量名]**

选项：
**-p**：“提示信息”：在等待read输入时，输出提示信息；
**-t ：秒数**：read命令会一直等待用户输入，使用此选项可以指定等待时间；
-n：字符数：read命令只接收指定的字符数就会执行；
-s：隐藏输入的数据，适用于机密信息的输入；
变量名可以自定义。如果不指定变量名，则会把输入保存到默认变量REPLY中；
如果只提供了一个变量名，则将整个输入行赋予该变量；
如果提供了一个以上的变量名，则输入行分为若干字，一个接一个地赋予各个变量，而命令行上的最后一个变量取得剩余的所有字；

```shell
#!/bin/bash
# 提示“Please enter string: ” 10秒，把输入保存到变量inputstr
read -t 10 -p "Please enter string: " inputStr
case $inputStr in
quit)
  exit 1
  ;;
*)
  echo "$inputStr"
  ;;
esac
```

最大值

```shell
#!/bin/bash
max=0
a=8
b=4
c=5
for i in $a $b $c
do
  if [ $i -gt $max ]
  then
    max=$i
  fi
done
echo $max
```

计算100以内的偶数和

```shell
# 第一种  2550
1 #!/bin/bash                 
  2          
  3 sum=0    
  4          
  5 for i in {1..100}           
  6 do       
  7     if ((i%2==0)); then     
  8         sum=`expr $sum + $i`
  9     fi   
 10 done     
 11 echo $sum
 
 # 第二种
 # seq [选项]... 尾数	seq [选项]... 首数 尾数	seq [选项]... 首数 增量 尾数
 # 选项：-f, --format=FORMAT      use printf style floating-point FORMAT
     	-s, --separator=STRING   use STRING to separate numbers (default: \n)
     	-w, --equal-width        equalize width by padding with leading zeroes
 #!/bin/bash
 
 sum=0
 for i in `seq 2 2 100`
 do
 	let sum+=$i
 done
 echo $sum
 
 # 第三种
 #!/bin/bash
 
 sum=0
 for i in `seq 2 2 100`
 do
 	sum=`expr $sum + $i`
 done
 echo $sum
```

#### 函数

##### 函数定义

```shell
[ function ] funname [()]
{
    action;
    [return int;]
}
```

```shell
#!/bin/bash

demoFun(){
    echo "This is my first shell function!"
}
echo "-----Execution-----"
demoFun
echo "-----Finished-----"

result：
-----Execution-----
This is my first shell function!
-----Finished-----
```

```shell
#!/bin/bash
funWithReturn(){
    echo "This function will add the two numbers of the input..."
    echo "Enter the first number: "
    read aNum
    echo "Enter the second number: "
    read anotherNum
    echo "The two numbers are $aNum and $anotherNum !"
    # return $(($aNum+$anotherNum)) 	写法最自由，表达式部分可以带空格也可以不带
    # return `$aNum + $bNum`	expr后面接的是具体运算表达式，采用``语法将值计算完后赋给前面变量
    let re=$aNum+$bNum	#	let后面接的是完整表达式，注意等号及运算符两边不可以有空格
    return $re
}
funWithReturn
echo "The sum of the two numbers entered is $? !"

# 函数返回值在调用该函数后通过 $? 来获得
result：
This function will add the two numbers of the input...
Enter the first number:
1
Enter the second number:
2
The two numbers are 1 and  2 !
The sum of the two numbers entered is 3 !
```

##### 函数参数

```shell
#!/bin/bash
funWithParam(){
    echo "The first parameter is $1 !"
    echo "The second parameter is $2 !"
    echo "The tenth parameter is $10 !"
    echo "The tenth parameter is ${10} !"
    echo "The eleventh parameter is ${11} !"
    # $# 传递到脚本的参数个数
    echo "The total number of parameters is $# !"
    # $* 以一个单字符串显示所有向脚本传递的参数。与位置变量不同,此选项参数可超过 9 个
    echo "Outputs all parameters as a string $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 66 88

result:
The first parameter is 1 !
The second parameter is 2 !
The tenth parameter is 10 !
The tenth parameter is 66 !
The eleventh parameter is 88 !
The total number of parameters is 11 !
Outputs all parameters as a string 1 2 3 4 5 6 7 8 9 66 88 !
```

练习

让系统随机生成一个数字，给这个数字一个范围，让用户猜数字，对输入作出判断，并且给出提示。

```shell
# 使用 RANDOM 随机整数函数，RANDOM 为系统自带的系统变量，值为 0-32767
# 使用取余算法将随机数变为 1-100 的随机数 RANDOM%100+1

#!/bin/bash
funcation randNum(){
	while :
	do 
		read num
		if (($num == $1)); then
			echo "right"
			break 1
		elif (($num > $1)); then
			echo "The answer is smaller than yours.!"
		else
			echo "The answer is bigger than yours.!"
		fi
	done
}

echo $(($RANDOM%100+1))
```

