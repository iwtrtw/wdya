+ ls：list directory contents

  > -l   列表形式显示
  >
  > -r  倒序
  >
  > -t  按修改时间排序，最新

+ cat

  > -n 显示行数

+ chmod：change file mode bits  修改文件读写权限

  > chmod  u+x demo.txt   添加执行权限

+ chown：change file owner and group 修改文件所属用户

+ cp：copy files and directories 复制文件

+ mv：move (rename) files  移动文件

+ diff：compare files line by line 比较文件差异

+ find：search for files in a directory hierarchy  查询文件

+ touch：创建一个空文件

+ which：在环境变量$PATH设置的目录里查找符合条件的文件

+ ssh：远程登录

  > ssh @root  192.168.132.12   //以root用户远程登录到

+ wc：print newline, word, and byte counts for each file

  > -l 统计行数
  >
  > -w 统计单词数量

+ id：查看当前用户

+ uname：查看当前主机信息

+ passwd：修改用户密码

+ df：查看磁盘空间使用情况

  > -h  查看主机的空间使用情况

+ echo：标准输出

+ head：查看文件前N行

  > -n

+ tail：查看文件后N行

  > -f  刷新显示
  >
  > -n  n行

+ mkdir：创建文件夹

+ exit：退出主机登录

文件权限：所有者(owner)+所属组(group)+其他(others)

### 基本符号

+ 声明变量与使用

  ```shell
  a=15
  echo $a #调用输出a
  echo ${a} #调用输出a 当${a}oy可以正常输出
  echo $? #返回上一条命令是否执行失败(1/0)
  ```

  ```shell
  #!/bin/bash
  # date 2020-12-11
  # demo1
  echo "脚本：$0"
  echo "第一个参数为：$1"
  echo "第二个参数为：$2"
  echo "一共有多少个参数: $#"
  echo "这些参数为: $*"
  ——————————————————————————————
  [root@localhost]sh demo1.sh a b c d 3
  脚本：demo1.sh
  第一个参数为：a
  第二个参数为：b
  一共有多少个参数: 5
  这些参数为: a b c d 3
  ```
  
+ 常见符号

```shell 
> 覆盖内容
[root] cat >abc.txt  #往abc.txt加入内容，覆盖原本

>>末尾追加内容
[root] cat >>abc.txt #往abc.txt末尾追加内容

;分号左右的命令都会执行
[root] cat abc.txt;ls -l

| 管道，把左边的输出内容传给右边处理
[root] ps -ef | grep redis

&& 当左边命令执行成功才会执行右边命令
[root] cat abcd.txt && ls

|| 当左边命令执行不成功才会执行右边命令
[root]

"" 会输出变量值，当变量被双引号引住时，可以输出变量值
'' 输出本身，变量值不会被输出
`` 当在反撇号中输入命令时会被执行
[root] a=`date`
[root]# echo $a
Fri Dec 11 07:49:12 EST 2020

2>/dev/null  #把命令执行产生的错误丢到这里，不显示在终端
1>/dev/null #把命令正常执行的内容丢到这里，不显示在终端
```

+ 整数运算

```shell
expr num1 + num2  #加号旁边需要空格
[root] expr 12 + 6
18

echo $[num1+num2]
[root] echo $[12+6]
18
```

```shell
expr num1 - num2  #减号旁边需要空格
[root] expr 12 - 6
6

echo $[num1-num2]
[root] echo $[12-6]
6
```

```shell
expr num1 \* num2  #乘号旁边需要空格
[root] expr 12 \* 6
72

echo $[num1*num2]
[root] echo $[12*6]
72
```

```shell
expr num1 / num2  #除号旁边需要空格
[root] expr 12 / 6
2

echo $[num1/num2]
[root] echo $[12/6]
2
```

```shell
[root] a=12;b=6;expr $a + $b
18

[root] a=12;b=6; echo $[a+b]
18

[root] a=12;b=6; echo $((a+b))
18
```

+ 计算器bc

```shell
[root] echo "2.4+22" | bc
24.4

[root] echo "scale=2;6/3" | bc  #scale保留小数位数(只对除法、取余、乘幂有效，使用加减时可以除以1)
2.00
```

+ 条件判断

```shell
[ 表达式 ]

-e 目标是否存在
-d 是否为路径
-f 是否为文件
[root] [ -e abc.txt ] && echo "abc.txt 存在"  #判断abc.txt文件是否存在

[root] [ -e abc.c ] || touch abc.c  # abc.c若不存在则创建

-r 是否有读取权限
-w 是否有写入权限
-x 是否有执行权限

只适用于整数：
[num1 符号 num2]
-eq 等于
-ne 不等于
-gt 大于
-lt 小于
-ge 大于或者等于
-le 小于或者等于

字符串：
= 相等
!= 不相等

小数：利用bc 当结果为true返回1，false返回0
[root] [ `echo '1.2<1.3' | bc` -eq 1 ] && echo "小于" 
小于
```

+ 输入

```shell
read -参数
-p 给出提示符，默认不支持"\n"换行
-s 隐藏输入的内容
-t 给出等待时间，超时户退出
-n 限制读取字符的个数，触发临界值会自动执行

[root] echo "please input pass：";read pass
plese input pass:
123245
[root] echo $pass
123456

[root] read -p "请输入密码:" pass  #将输入的内容赋值给变量pass
请输入密码：8888
[root] echo $pass
8888

[root] read -s -p "请输入密码:" pass  #隐藏输入内容
请输入密码：
[root] echo $pass
8888

[root] read -t 5 -s -p "请输入密码:" pass  #5秒后退出
请输入密码：
[root] echo $pass

[root] read -n 5 -p "请输入密码:" pass  #当输入字符长度到5时自动执行
请输入密码：
```

+ grep：对数据进行行的提取

  > grep [参数] [内容] [file]
  >
  > -v  内容取反提取
  >
  > grep -v 'name'  student.txt  //查找不包含name的行
  >
  > -n 显示内容在file中的行号
  >
  > grep -n 'name'  student.txt  //查找包含name的行并显示原文件行号
  >
  > -w 精确匹配只含内容的行
  >
  > grep -w 'name' student.txt //查找只含name的行
  >
  > -i 忽略大小写
  >
  > grep -ii 'name' student.txt //查找只含name的行（忽略大小写）
  >
  > ^ 匹配开头行首
  >
  > grep  '^name' student.txt //查找已name为开头的行
  >
  > -E 正则匹配
  >
  > grep  -E 'name|age|grade' student.txt //查找包含name或age或grade的行

+ cut：对数据进行行的提取（默认以制表符来进行分割,对于没有该分割符的行会显示出所有内容）

  > cat [参数]  [file]
  >
  > -d 指定分割符
  >
  > -f 指定截取区域(列)
  >
  > -c 以字符为单位进行分割

  ```shell
  以":"为分隔符，截取出/etc/passwd的第一列跟第三列
  cut -d ":" -f 1,3 demo.txt
  
  以":"为分隔符，截取出/etc/passwd的第一列到第三列
  cut -d ":" -f 1-3 demo.txt
  
  以":"为分隔符，截取出/etc/passwd的第二列到最后一列
  cut -d ":" -f 2- demo.txt
  
  cat demo.txt | cut -d ":" -f 2-
  
  截取第二字符到第九个字符
  cut -c 2-9 demo.txt
  
  截取出linux上所有可登录的普通用户
  cat /etc/passwd | grep '/bin/bash' | cut -d ":" -f 1 | grep -v root
  ```

+ awk

  ```shell
  awk '条件 {执行动作}' 文件名 
  -F 指定分隔符
  BEGIN 在读取所有行内容前就开始执行，常用于修改内置变量的值
  FS BEGIN时定义分隔符
  END 结束时执行
  NR 行号
  [root] cat /etc/passwd | awk -F":" '{print $1}'   #打印第一列
  [root] cat /etc/passwd | awk 'BEGIN {FS=":"}{print $1}'
  [root] df -h | awk 'NR==4 {print $5}'  #打印第5列第4行
  [root] df -h | awk '(NR>20 && NR<30) {print $1}' /etc/passwd  #打印第1列，第20到30行
  ```

  ```shell
  [root] printf '%s%s%s\n' 1 2 3 4 5 6
  123
  456
  [root] printf '%s\t%s\t%s\n' 1 2 3 4 5 6
  1	2	3
  4	5	6
  ```

  ```shell
  $5 代表第5列
  $0 代表一整行
  每一行数据传递给awk，然后awk会依次执行动作
  [root] df -h | grep /dev/sda1 | awk '{printf "/dev/vda1的使用率是："}{print $5}'
  /dev/vda1的使用率是：17%
  ```

+ sed：对数据进行选取、新增、替换、删除、搜索

  ```shell
  sed [选项] [动作] 文件名
  -n 把匹配到的行输出到打印到屏幕
  p 以行为单位进行查询
  d 删除
  a 在行的下面插入新的内容
  i 在行的上面插入新的内容
  c 替换
  s/要被取代的内容/新的字符串/g   #指定内容进行替换
  ------------以上命令只是修改显示，并不会修改源文件
  
  -i 会对源文件进行修改（高危操作）
  -e 表示可以执行多个动作
  ```

  ```shell
  [root] df -h | sed -n '2p' #打印第二行
  [root] df -h | sed '2d' #删除第二行
  [root] df -h | sed '2a  abc' #在第二行下面插入新内容abc
  [root] df -h | sed '2i  abc' #在第二行上面插入新内容abc
  [root] df -h | sed '2c  abc' #把第二行上面替换新内容abc
  [root] df -h | sed 's/0%/30%/g' df.txt #把所有0%改成30%
  [root] df -h | sed -n '/abc/p' #搜索包含abc的行
  ```

### 循环控制

+ if-then

>if [条件判断]；
>​	then
>fi

> if [条件判断]；
> 	then
> else
> fi

> if [条件判断]
> ​	then
> elif [条件判断]
> ​	then
> elif [条件判断]
> ​	then
> fi

```shell
#!/bin/bash
#2020-12-12
echo '请输入一个数字：'
read number
if [ $number -eq 10 ];
   then
   echo '等于10'
elif [ $number -lt 10 ]
   then
   echo '小于10'
elif [ $number -gt 10 ]
   then
   echo '大于10'
fi
```



+ for

> for 变量名 in 值1 值2 值3
> ​	do
> ​	执行动作
> ​	done

> for 变量名 in `命令`
> ​	do
> ​	执行动作
> ​	done

```shell
#!/bin/bash
for i in `seq 1 10`
	do
	echo $i
	sleep 2
	done
```

```shell
#!/bin/bash
for i in $(cat dn.txt)
	do
	ping -c 2 $i
	echo -e "\n"
	done
```

