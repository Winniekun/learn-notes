## 概论

shell是我们通过命令行与操作系统沟通的语言。shell脚本可以直接在命令行中执行，也可以将一套逻辑组织成一个文件，方便复用。

Linux中常见的shell脚本有很多种，常见的有：

- Bourne Shell(/usr/bin/sh或/bin/sh)
- Bourne Again Shell(/bin/bash)

- C Shell(/usr/bin/csh)
- K Shell(/usr/bin/ksh)

- zsh

Linux系统中一般默认使用bash，所以接下来bash中的语法，文件开头需要写#! /bin/bash，指明bash为脚本解释器。

## 注释

### 单行注释

```bash
# 这是一行注释
echo 'Hello World'  #  这也是注释
```

### 多行注释

```bash
:<<EOF
第一行注释
第二行注释
第三行注释
EOF

其中EOF可以换成其它任意字符串。例如：

:<<abc
第一行注释
第二行注释
第三行注释
abc

:<<!
第一行注释
第二行注释
第三行注释
!
```

## 变量

### 定义变量

**定义变量，不需要加$符号，例如：**

```bash
name1='kwk'  # 单引号定义字符串
name2="kwk"  # 双引号定义字符串
name3=kwk    # 也可以不加引号，同样表示字符串
```

### 使用变量

**使用变量，需要加上$符号，或者${}符号。花括号是可选的，主要为了帮助解释器识别变量边界。**

```bash
name=kwk
echo $name  # 输出kwk
echo ${name}  # 输出kwk
echo ${name}linux  # 输出kwklinux
```

### 只读变量

**使用****readonly****或者****declare****可以将变量变为只读。**

```bash
name=kwk
readonly name
declare -r name  # 两种写法均可
name=abc  # 会报错，因为此时name只读
```

### 删除变量

**unset****可以删除变量。**

```bash
name=wkk
unset name
echo $name  # 输出空行
```

### 变量类型

1. 自定义变量（局部变量）

1. 1. 子进程不能访问的变量

1. 环境变量（全局变量）

1. 1. 子进程可以访问的变量

自定义变量改成环境变量：export、declare

```bash
name=wkk  # 定义变量
export name  # 第一种方法
declare -x nam ie  # 第二种方法
```

环境变量改为自定义变量：

```bash
export name=kwk
# 定义环境变量
declare +x name  # 改为自定义变量
```

### 字符串

字符串可以用单引号，也可以用双引号，也可以不用引号。



单引号与双引号的区别：

1. 单引号中的内容会原样输出，不会执行、不会取变量；
2. 双引号中的内容可以执行、可以取变量；

```bash
name=wkk  # 不用引号
echo 'hello, $name \"hh\"'  # 单引号字符串，输出 hello, $name \"hh\"
echo "hello, $name \"hh\""  # 双引号字符串，输出 hello, wkk "hh"
```

获取字符串长度

```bash
name="wkk"
echo ${#name}  # 输出3
```

提取子串

```bash
name="hello, wkk"
echo ${name:0:5}  # 提取从0开始的5个字符
```

## 默认变量

### 文件参数变量

在执行shell脚本时，可以向脚本传递参数。$1是第一个参数，$2是第二个参数，以此类推。特殊的，$0是文件名（包含路径）。例如：

**创建test.sh：**

```bash
#! /bin/bash

echo "文件名："$0
echo "第一个参数："$1
echo "第二个参数："$2
echo "第三个参数："$3
echo "第四个参数："$4
```

**执行test.sh：**

```bash
chmod +x test.sh 
./test.sh 1 2 3 4
文件名：./test.sh
第一个参数：1
第二个参数：2
第三个参数：3
第四个参数：4
```

**其它参数相关变量**

| 参数       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| $#         | 代表文件传入的参数个数，如上例中值为4                        |
| $*         | 由所有参数构成的用空格隔开的字符串，如上例中值为**$1 $2 $3 $4** |
| $@         | 每个参数分别用双引号括起来的字符串，如上例中值为**$1 $2 $3 $4** |
| $$         | 脚本当前运行的进程ID                                         |
| $?         | 上一条命令的退出状态（注意不是stdout，而是exit code）。0表示正常退出，其他值表示错误 |
| $(command) | 返回command这条命令的stdout（可嵌套）                        |
| command    | 返回command这条命令的stdout（不可嵌套）                      |

## 数组

数组中可以存放多个不同类型的值，只支持一维数组，初始化时不需要指明数组大小。数组下标从0开始。

### 定义

数组用小括号表示，元素之间用空格隔开。例如：

```bash
array=(1 abc "def" kwk)
```

也可以直接定义数组中某个元素的值：



```bash
array[0]=1
array[1]=abc
array[2]="def"
array[3]=kwk
```

### 读取数组中某个元素的值

**格式：**

```bash
${array[index]}
```

**例如：**

```bash
array=(1 abc "def" kwk)
echo ${array[0]} # 
echo ${array[1]} # 1
echo ${array[2]} # abc
echo ${array[3]} # def
echo ${array[4]} # kwk
```

### 读取整个数组

**格式：**

```bash
${array[@]}  # 第一种写法
${array[*]}  # 第二种写法
```

**例如：**

```bash
array=(1 abc "def" kwk)
echo ${array[@]}  # 第一种写法 1 abc def kwk
echo ${array[*]}  # 第二种写法 1 abc def kwk
```

### 数组长度

**类似于字符串**

```bash
${#array[@]}  # 第一种写法
${#array[*]}  # 第二种写法
```

**例如：**

```bash
array=(1 abc "def" kwk)
echo ${#array[@]}  # 第一种写法
echo ${#array[*]}  # 第二种写法
```

## expr命令

**格式：**

```bash
expr 表达式
```

**表达式说明：**

- 用空格隔开每一项
- 用反斜杠放在shell特定的字符前面（发现表达式运行错误时，可以试试转义）

- 对包含空格和其他特殊字符的字符串要用引号括起来
- expr会在stdout中输出结果。如果为逻辑关系表达式，则结果为真，stdout为1，否则为0。

- expr的exit code：如果为逻辑关系表达式，则结果为真，exit code为0，否则为1。



------

### 字符串表达式：

- length **STRING**

- - 返回**STRING**的长度

- index **STRING CHARSET**

- - **CHARSET**中任意单个字符在**STRING中**最前面的字符位置，下标从1开始。如果在**STRING**中完全不存在**CHARSET**中的字符，则返回0。

- substr **STRING POSITION LENGTH**

- - 返回**STRING**字符串中从**POSITION**开始，长度最大为**LENGTH**的子串。如果**POSITION**或**LENGTH**为负数，0或非数值，则返回空字符串。

**示例：**

```bash
str="Hello World!"
echo `expr length "$str"`  # ``不是单引号，表示执行该命令，输出12
echo `expr index "$str" aWd`  # 输出7，下标从1开始
echo `expr substr "$str" 2 3`  # 输出 el
```

### 整数表达式：

expr支持普通的算术操作，算术表达式优先级低于字符串表达式，高于逻辑关系表达式。



- \+ -

- - 加减运算。两端参数会转换为整数，如果转换失败则报错。

- \* / %
  乘，除，取模运算。两端参数会转换为整数，如果转换失败则报错。

- () 可以该表优先级，但需要用反斜杠转义

**示例：**

```bash
a=3
b=4

echo `expr $a + $b`  # 输出7
echo `expr $a - $b`  # 输出-1
echo `expr $a \* $b`  # 输出12，*需要转义
echo `expr $a / $b`  # 输出0，整除
echo `expr $a % $b` # 输出3
echo `expr \( $a + 1 \) \* \( $b + 1 \)`  # 输出20，值为(a + 1) * (b + 1)
```

### 逻辑关系表达式：

- |

- - 如果第一个参数非空且非0，则返回第一个参数的值，否则返回第二个参数的值，但要求第二个参数的值也是非空或非0，否则返回0。如果第一个参数是非空或非0时，不会计算第二个参数。

- &

- - 如果两个参数都非空且非0，则返回第一个参数，否则返回0。如果第一个参为0或为空，则不会计算第二个参数。

- < <=  == != => >

- - 比较两端的参数，如果为true，则返回1，否则返回0。”==”是”=”的同义词。”expr”首先尝试将两端参数转换为整数，并做算术比较，如果转换失败，则按字符集排序规则做字符比较。

- () 可以该表优先级，但需要用反斜杠转义

```bash
a=3
b=4

echo `expr $a \> $b`  # 输出0，>需要转义
echo `expr $a '<' $b`  # 输出1，也可以将特殊字符用引号引起来
echo `expr $a '>=' $b`  # 输出0
echo `expr $a \<\= $b`  # 输出1

c=0
d=5

echo `expr $c \& $d`  # 输出0
echo `expr $a \& $b`  # 输出3
echo `expr $c \| $d`  # 输出5
echo `expr $a \| $b`  # 输出3
```

## read命令

**read**命令用于从标准输入中读取单行数据。当读到文件结束符时，**exit code**为1，否则为0。

参数说明

- -p: 后面可以接提示信息
- -t：后面跟秒数，定义输入字符的等待时间，超过等待时间后会自动忽略此命令

**实例：**

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635318782884-6f63197d-a6fa-44c8-957f-53f3b27cabf2.png)

## echo命令

用于输出字符串。

**格式：**

```bash
echo STRING
```

### 显示普通字符串

```bash
echo "Hello kwk"
echo Hello kwk  # 引号可以省略
```

### 显示转义字符

```bash
echo "\"Hello kwk\""  # 注意只能使用双引号，如果使用单引号，则不转义
echo \"Hello kwk\"  # 也可以省略双引号
```

### 显示变量

```bash
name=kwk
echo "My name is $name"  # 输出 My name is kwk
```

### 显示换行

```bash
echo -e "Hi\n"  # -e 开启转义
echo "kwk"
```

输出结果：

```bash
Hi

kwk
```

### 显示不换行

```bash
echo -e "Hi \c" # -e 开启转义 \c 不换行
echo "kwk"
```

**输出结果：**

```bash
Hi kwk
```

### 显示结果定向至文件

```bash
echo "Hello World" > output.txt  # 将内容以覆盖的方式输出到output.txt中
```

### 原样输出字符串，不进行转义或取变量(用单引号)

```bash
name=wkk
echo '$name\"'
```

**输出结果：**

```bash
$name\"
```



### 显示命令的执行结果

```bash
echo `date`
```

**输出结果：**

```bash
Wed Oct 27 15:20:56 CST 2021
```

## printf

printf命令用于格式化输出，默认不会在字符串末尾添加换行符。

**格式：**

```bash
printf format-string [arguments...]
```

**用法示例：**

```bash
printf "%10d.\n" 123  # 占10位，右对齐
printf "%-10.2f.\n" 123.123321  # 占10位，保留2位小数，左对齐
printf "My name is %s\n" "kwk"  # 格式化输出字符串
printf "%d * %d = %d\n"  2 3 `expr 2 \* 3` # 表达式的值作为参数
```

**输出结果：**

```bash
       123.
123.12    .
My name is kwk
2 * 3 = 6
```

## test命令与判断符号[]

- && 表示与，|| 表示或
- 二者具有短路原则：

- - expr1 && expr2：当expr1为假时，直接忽略expr2
  - expr1 || expr2：当expr1为真时，直接忽略expr2

- 表达式的exit code为0，表示真；为非零，表示假。（与C/C++中的定义相反）

### test命令

在命令行中输入man test，可以查看test命令的用法。

1. test命令用于判断文件类型，以及对变量做比较。
2. test命令用exit code返回结果，而不是使用stdout。0表示真，非0表示假。

**例如：**

```bash
test 2 -lt 3  # 为真，返回值为0
echo $?  # 输出上个命令的返回值，输出0
```

#### 文件类型判断

**格式：**

```bash
test -e filename  # 判断文件是否存在
```

| **测试参数** | **代表意义** |
| ------------ | ------------ |
| -e           | 文件是否存在 |
| -f           | 是否为文件   |
| -d           | 是否为目录   |

#### 文件权限判断

**格式：**

```bash
test -r filename  # 判断文件是否可读
```

| **测试参数** | **代表意义**   |
| ------------ | -------------- |
| -r           | 文件是否可读   |
| -w           | 文件是否可写   |
| -x           | 文件是否可执行 |
| -s           | 是否为非空文件 |

#### 整数间的比较

**格式：**

```bash
test $a -eq $b  # a是否等于b
```

| **测试参数** | **代表意义**   |
| ------------ | -------------- |
| -eq          | a是否等于b     |
| -ne          | a是否不等于b   |
| -gt          | a是否大于b     |
| -lt          | a是否小于b     |
| -ge          | a是否大于等于b |
| -le          | a是否小于等于b |

#### 字符串比较

| **测试参数**      | **代表意义**                                           |
| ----------------- | ------------------------------------------------------ |
| test -z STRING    | 判断STRING是否为空，如果为空，则返回true               |
| test -n STRING    | 判断STRING是否非空，如果非空，则返回true（-n可以省略） |
| test str1 == str2 | 判断str1是否等于str2                                   |
| test str1 != str2 | 判断str1是否不等于str2                                 |

#### 多重条件判定

**格式：**

```bash
test -r filename -a -x filename
```

| **测试参数** | **代表意义**                                        |
| ------------ | --------------------------------------------------- |
| -a           | 两条件是否同时成立                                  |
| -o           | 两条件是否至少一个成立                              |
| -r           | 取反。如 test ! -x file，当file不可执行时，返回true |

### 判断符号[]

[]与test用法几乎一模一样，更常用于if语句中。另外[[]]是[]的加强版，支持的特性更多。

```bash
[ 2 -lt 3 ]  # 为真，返回值为0
echo $?  # 输出上个命令的返回值，输出0
```

注意：

- []内的每一项都要用空格隔开
- 中括号内的变量，最好用双引号括起来

- 中括号内的常数，最好用单或双引号括起来

例如：

```bash
name="hello world wkk"
[ $name == "hello world wkk" ]  # 错误，等价于 [ acwing kwk == "acwing kwk" ]，参数太多
[ "$name" == "hello world wkk" ]  # 正确
```

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635333938673-c174236c-495a-4843-80e2-13ccfc91b825.png)

## 判断

和正常的**if/else**逻辑类似

### if…then形式

**格式：**

```bash
if condition
then
    语句1
    语句2
    ...
fi
```

**示例：**

```bash
a=3
b=4

if [ "$a" -lt "$b" ] && [ "$a" -gt 2 ]
then
    echo ${a}在范围内
fi
```

**输出结果：**

```bash
3在范围内
```

#### 单层if-else

**格式：**

```bash
if condition
then
    语句1
    语句2
    ...
else
    语句1
    语句2
    ...
fi
```

**示例：**

```bash
a=3
b=4

if ! [ "$a" -lt "$b" ]
then
    echo ${a}不小于${b}
else
    echo ${a}小于${b}
fi
```

**输出结果：**

```bash
3小于4
```

#### 多层if-elif-elif-else

**格式：**

```bash
if condition
then
    语句1
    语句2
    ...
elif condition
then
    语句1
    语句2
    ...
elif condition
then
    语句1
    语句2
else
    语句1
    语句2
    ...
fi
```

**示例：**

```bash
a=4

if [ $a -eq 1 ]
then
    echo ${a}等于1
elif [ $a -eq 2 ]
then
    echo ${a}等于2
elif [ $a -eq 3 ]
then
    echo ${a}等于3
else
    echo 其他
fi
```

**输出：**

```bash
其他
```

### case…esac形式

**格式：**

```bash
case $变量名称 in
    值1)
        语句1
        语句2
        ...
        ;;  # 类似于C/C++中的break
    值2)
        语句1
        语句2
        ...
        ;;
    *)  # 类似于C/C++中的default
        语句1
        语句2
        ...
        ;;
esac
```

**示例：**

```bash
a=4

case $a in
    1)
        echo ${a}等于1
        ;;  
    2)
        echo ${a}等于2
        ;;  
    3)                                                
        echo ${a}等于3
        ;;  
    *)
        echo 其他
        ;;  
esac
```

**输出结果：**

```bash
其他
```

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635337139596-01f9f78b-2dd9-4e05-a5d6-0c538533a0ad.png)

## 循环

类似正常的for loop

### for…in…do…done

**格式：**

```bash
for var in val1 val2 val3
do
    语句1
    语句2
    ...
done
```

#### I 正常示例：

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635337381653-a8161f96-f1d5-431f-a49b-e7bba523d93d.png)

#### II 输出当前路径下的所有文件名，每个文件名一行:

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635337485738-481fcb41-0a9d-4336-aff0-cb261949ce1f.png)

#### III 输出1-10

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635337835287-6cba1dbf-e097-4080-b615-5a0c5efe7f85.png)

#### IV使用{1..10} 或者 {a..z}

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635338063907-1f025108-8610-4aa9-8fc3-d1ceb10509cb.png)

### for ((…;…;…)) do…done

**格式：**

```bash
for ((expression; condition; expression))
do
    语句1
    语句2
done
```

#### I 输出1-10，每个数占一行：

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635338526631-bc9ed425-fcef-4f32-8ea4-b1b49156c9f5.png)

### while…do…done循环

**格式：**

```bash
while condition
do
    语句1
    语句2
    ...
done
```

#### 文件结束符为Ctrl+d，输入文件结束符后read指令返回false

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635338649144-c2b4a8eb-24fd-408e-9f98-9467d78f502f.png)

### until…do…done循环

**格式：**

```bash
until condition
do
    语句1
    语句2
    ...
done
```

#### 当用户输入yes或者YES时结束，否则一直等待读入

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635338942622-32895d62-afe4-475c-bf95-bfad68f9bc62.png)

### break命令

注意：break不能跳出case语句。

示例：

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635339120492-9e9c3076-d93e-417b-b8f7-fbfb457402eb.png)

### continue命令

跳出当前循环

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635339178975-0461bd27-86a0-4029-b205-deef2ded71b4.png)

### 编写shell时造成的死循环问题：

1. Ctrl+c。
2. 使用top命令找到进程的PID、输入kill -9 PID即可关掉此进程



## 函数

bash中的函数类似于Java中的方法，但return的返回值与Java不同，返回的是exit code，取值为0-255，0表示正常结束。

如果想获取函数的输出结果，可以通过echo输出到stdout中，然后通过$(function_name)来获取stdout中的结果。**函数的return值可以通过$?来获取**

**格式：**

```bash
[function] func_name() {  # function关键字可以省略
    语句1
    语句2
    ...
}
```

### 不获取 return值和stdout值

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635339797006-ff75f5ca-ee54-402d-a5a7-58d71e7c66bd.png)

### 获取 return值和stdout值

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635339980595-c40f3ef5-7b80-4362-b8bd-72beff69c5c6.png)

### 函数的入参

在函数内，**$1**表示第一个输入参数，**$2**表示第二个输入参数，依此类推。

注意：函数内的$0仍然是文件名，而不是函数名。

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635340381800-89fb0ea6-55fe-4aeb-abe0-f13c451e990f.png)

### 局部变量

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635340563612-b38ad57c-d9c5-44c3-bfb8-ee36c1a9d379.png)

## exit

- exit命令用来退出当前shell进程，并返回一个退出状态；使用$?可以接收这个退出状态。
- exit命令可以接受一个整数值作为参数，代表退出状态。如果不指定，默认状态值是 0。

- exit退出状态只能是一个介于 0~255 之间的整数，其中只有 0 表示成功，其它值都表示失败。



![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635341329021-bc1d10cf-d3ec-473e-ac04-686af10ba529.png)



## 文件重定向

每个进程默认打开3个文件描述符：

1. stdin标准输入，从命令行读取数据，文件描述符为0
2. stdout标准输出，向命令行输出数据，文件描述符为1

1. stderr标准错误输出，向命令行输出数据，文件描述符为2

可以用文件重定向将这三个文件重定向到其他文件中。

### 重定向命令列表

| 命令             | 说明                                  |
| ---------------- | ------------------------------------- |
| command > file   | 将stdout重定向到file中                |
| command < file   | 将stdin重定向到file中                 |
| command >> file  | 将stdout以追加方式重定向到file中      |
| command n> file  | 将文件描述符n重定向到file中           |
| command n>> file | 将文件描述符n以追加方式重定向到file中 |

### 输入和输出重定向

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635341841809-ede8b3a9-0d55-459d-bc95-8f1ef0dc792d.png)

### 同时重定向stdin和stdout

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635342088452-be41a62e-5d2d-4ff7-b1e6-24c19a53b87c.png)

## 引入外部脚本

类似import

**格式：**

```bash
. filename  # 注意点和文件名之间有一个空格

or

source filename
```

示例：

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1635342741941-a4043101-8fbf-4888-b98e-6beddab253d7.png)