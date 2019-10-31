---
title: Shell学习笔记
date: 2019-06-23 23:20:46
tags: [Shell,Linux]
---

## Shell命令

```
#!/bin/bash
```

### 变量

```shell
#变量名:不需要使用$符号，"="不能有空格
my_name="BinZhiZhu"

#输出:需要$符号
echo $my_name
```

### 字符串

```sh
#单引号

str='I try to learn shell '
echo $str

#双引号:可以有变量,可以有转义字符

str="My name is \"$my_name\",Nice to meet u"
echo $str

#拼接

str1="What's ur name,bro? My name is "$my_name"!"

str2="What's ur name,bro? My name is "{$my_name}"!"

str3="What's ur name,bro? My name is {$my_name}!"

echo $str1
echo $str2
echo $str3

#获取长度:${# + 变量}

str='qwer'

echo ${#str}

#提取子字符串:

str="my name is $my_name "

#提取名字：BinZhiZhu
echo ${str:11}

#提取指定索引范围:name
echo ${str:3:5}
```

### 数组

```sh
#括号括起来 元素之间用空格相隔
array=(a b c d)


#可以直接赋值
array1[0]=1
array1[1]=2

echo "所有元素为：${array[*]}"
echo "第一个元素为：${array[0]}"
echo "第二个元素为：${array[1]}"
echo "第三个元素为：${array[2]}"
echo "第四个元素为：${array[3]}"
```

### 流程控制

### if的使用

```shell
#运算符 : -gt:大于; -lt:小于; -eq:等于

#if使用

a=6
b=4

if [ $a -gt $b ]
then
     echo "a大于b咯"
elif [ $a -eq $b ]
then
     echo "a等于b咯"
elif [ $a -lt $b ]
then
     echo "a小于b咯"
else
     echo "没有找到该条件判断"
fi

#test命令

let num1=2*3
let num2=3+3

if test $num1 -eq $num2
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi


#等价于


let num3=2*3
let num4=3+2

if [ $num3 -eq $num4 ]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi
```

#### for循环

```shell
arr=(1 2 3 4)
for loop in ${arr[*]}
do
    echo "The Number is: $loop"
done

#写成一行
for loop in ${arr[*]};do echo "Number is: $loop";done;
```

#### while循环

```shell
#当num小于等于5条件为true
num=1

while(($num <= 5 ))
do
   echo "num is:$num"
   let num++
done


#读取键盘信息

echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read FILM
do
    echo "是的！$FILM 是一个好网站"
    break
done
```

#### until循环

```shell
#until 循环执行一系列命令直至条件为 true 时停止，与while相反

#输入1-10数字咯
a=0
b=10

until [ $a -eq $b ]
do
    echo " value is : $a"
    let a++
done
```

#### case循环

```shell
#esac的意思与case相反 也就是结束执行
echo "case 循环"

echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac


##输入 1 到 5 之间的数字才可结束循环
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```

#### 简单的删除文件

```
#!/bin/bash

filePath="del.php"

echo '确认删除del.php文件吗？(y/n)'
read check
case $check in
     'y')
         rm $filePath;;
     'n')
         echo '取消删除';;
      *)
         echo '请输入y/n';;
esac
```


