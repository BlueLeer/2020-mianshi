### 各种语言的执行的区别

* Java 先编译成字节码文件，然后由JVM执行字节码文件（其实是将字节码文件转化为可执行的二进制文件）
* Python 解释执行
* Shell 解释执行



### 输入输出重定向

输入重定向

`command < file1` 这样，原本需要从键盘得到的输入变成了从文件获取输入

输出重定向

`command > file2` 这样，将输出写入到outfile文件当中去

一般情况下，每个linux/Unix命令运行时，都会打开三个文件：

* 标准输入文件（stdin）：文件描述符为0，程序默认从stdin读取数据
* 标准输出文件（stdout）：文件描述符为1，程序默认想stdout输出数据
* 标准错误文件（stderr）：文件描述符为2，程序默认会向stderr写入错误信息

以上列举出来的 command < file1，就是stdin定向到file1文件；command > file2，就是将stdout定向到file2文件；若想将标准错误输出到指定的文件可以这样写：`command 2>>file3` 

如果希望将stdin和stderr合并后重定向到指定的文件，可以这样来写：`command > file 2>&1`，&1代表文件描述符，指的是标准输入，2>&1表明将标准错误重定向到文件的标准输出，> file指的是将标准输出重定向到文件，所以上面执行的效果就是命令的标准输入和标准错误输出都会输出到文件file中去。

> command > file 2>&1 还有其他两种等价的写法：
>
> command &> file
>
> command >& file
>
> 可以看到这2种写法更加的简洁

``` shell
# 上面的命令是一种特殊的输入输出重定向方式，注意：结尾处的delimiter是需要顶格写的，不然会识别不出来
command << delimiter
	docment
	docment
delimiter

# 有时候，我们为了代码格式的缩进便于阅读，可以在delimiter前面加上一个-,这样，在结束的delimiter前面就可以加上一个tab缩进了（空格是不可以 的）
command <<-EOF
	docment
	docment
	EOF
```

### shell 脚本执行的方式

比如我们现在有个test.sh脚本：

```shell
#!/bin/sh

cd /home
echo $?
```

* 在sub shell（在子shell中执行）种执行

  * bash test.sh
  * ./test.sh

* 在当前shell种执行

  * . test.sh
  * source test.sh

  我们可以看到，这两种执行方式执行以后，当前的目录变成了/home目录，其实这也说明了，这两种方式是在当前的shell中执行的。

  在当前的shell当中执行其他的shell脚本也有他的使用场景。

> **source深入解读**
>
> 当我们在修改完/etc/profile里面的文件后，或者系统的其他配置文件以后，我们需要执行下 source xxx，然后使得刚刚的修改生效。
>
> `source filename`，这个命令只是简单地读取脚本里面的语句，然后依次放到当前的shell中执行，没有建立新的子shell。使用source（和使用点事一样的）那么脚本里面所有新建、改变变量都会在当前shell中看到，它**没有另开进程**；使用sh执行脚本文件（或者./xxx.sh）,是在当前进程**另开子进程**来执行命令，脚本中设置的变量在当前shell中不能看到。
>
> 问：有一个alias.sh文件：
>
> ```shell
> #!/bin/bash
> alias get_po="kubectl get po --namespace default"
> ```
>
> 使用`. alias.sh`执行以后，然后再另外一个shell窗口中能使用get_po命令吗？答案是不能，我们定一个一个命令的别名，虽然相当于是在当前shell中执行的，但是它不会全局添加这个别名，还是对当前shell生效的。

> **文件权限**
>
> -rwxr-xr-x 1 root root    248 7月  13 17:23 test1.sh
>
> 第一位代表文件类型，234位代表文件所有者的权限，r代表可读，w代表可写，x代表可执行，-代表没有权限，用数字表示分别为4 2 1；567位代表同组用户的权限列表；8910位代表其他组用户所具有的权限。
>
> chomd 777 test.sh 代表赋予当前文件所有者，同组用户，其他组用户的读写可执行权限。

### shell中的别名 alias

在当前的session中，可以直接通过 alias alias_name="xxxx"来定义，比如：

```shell
[root@lee ~]# alias current_time="date"
[root@lee ~]# current_time
2021年 08月 01日 星期日 16:42:53 CST
```

这个alias只是在当前的session中有效；如果想要永久的保存alias，可以将其写入到/etc/profile或者～/.bash_rc中去，两个的区别是影响的用户范围不一样而已。



### 常用命令

1. 按键`ctrl+R`同时按下，然后可以搜索历史输入过的命令 

2. 按键`ctrl+D` 同时按下，推出当前会话

3. 按键`ctrl+A`对命令进行编辑，使得光标移动到命令的第一个字符处；`ctrl+E`，移动到末尾；`ctrl+K`删除光标之后的命令；`ctrl+U`删除光标之前的；`ctrl+S`锁屏，`ctrl+Q`解锁；`ctrl+Y`撤销

4. screen的使用

   screen需要另外安装，他是一个工具，用户保存回话，便于恢复，使用方法如下：

   ```shell
   # 1 创建一个screen
   [root@lee ~]# screen -S lee
   # 2 输入我们的命令
   [root@lee ~]# echo abc
   abc
   [root@lee ~]# echo def
   def
   [root@lee ~]#
   # 3 关闭窗口
   # 4 查看当前有哪些屏幕
   [root@lee ~]# screen -list
   There are screens on:
           31327.lee       (Attached)
           30251.test      (Attached)
           26647.pts-2.lee (Attached)
           25229.wl        (Attached)
   4 Sockets in /var/run/screen/S-root
   # 5 恢复会话
   [root@lee ~]# screen -r 31327
   There is a screen on:
           31327.lee       (Attached)
   There is no screen to be resumed matching 31327.
   [root@lee ~]#
   ```

5. &  和 nohup 的使用

   一般一起使用，nohup指的是不挂断地运行命令，&指的是在后台运行

   ```shell
   nohup command &
   # 进入后台运行以后就变成job，可以使用 job -l 命令来查看
   ```

6. fg和bg的使用（前台任务和后台任务）

   fg是让进程在前台工作，bg（ctrl+z组合按键的效果是一样的）是让进程在后台工作

   ```shell
   # bg命令查看后台进程
   [root@lee shell]# bg
   [4]+ vi bb.txt &
   
   [4]+  已停止               vi bb.txt
   # fg+作业号，让后台进程恢复至前台运行
   [root@lee shell]# fg 4
   
   # 日常当中，我们使用最多的是，比如我们正在通过vi编辑一个文件，这个时候需要回到主进程shell查看其他的信息，这个时候可以在vi的编辑页面同时按下ctrl+Z，这个时候vi进程就会退到后台运行，我们在主进程操作完了以后，输入fg命令可以快速让上一次的vi任务回到前台运行，这个时候就可以愉快的继续编辑了。如果有多个后台job，我们可以输入 jobs -l来查看作业号，然后通过 fg+作业号 使得后台的进程回到前台来继续运行。
   ```

7. kill命令 （可以杀掉进程，也可以杀掉作业）

   kill pid 停止进程pid的进程；kill %jobId 杀掉当前进程的作业号为jobId的job。

8. 输入输出重定向

   \> 	>>	<	<<	>&1	2>&1	>&	&> 

9. 管道和tee管道

   ```shell
   # 即上个命令的输出作为下个命令的输入
   [root@lee shell]# ps ef|grep 'sh'|grep -v ps
   
   # 使用tee管道使得管道里面的内容在某一处输出到文件中，而且不截流 tee -a是追加
   [root@lee shell]# cat EOF.sh |tee EOF2.sh
   #!/bin/bash
   
   echo <<-EOF nihao
   	EOF
   [root@lee shell]# cat EOF2.sh
   #!/bin/bash
   
   echo <<-EOF nihao
   	EOF
   [root@lee shell]#
   ```

### 命令排序

1. ；分号

   对命令进行分隔

   ```shell
   [root@lee ~]# cd /home/ksjfsidf;ls
   -bash: cd: /home/ksjfsidf: 没有那个文件或目录
   k8s-install  kubecfg.crt  kubecfg.key  kubecfg.p12  nohup.out
   # 我们可以看到，；对命令进行了分隔，使得一行可以同时执行多个命令，前一个命令不管执行成功与否，后面的命令会照常执行
   ```

2. && 

   也是分隔一行的多个命令，前一个命令执行成功，后面的命令才会执行，带有逻辑判断的功能

   ```shell
   [root@lee ~]# cd /home/ksjfksjd && ls
   -bash: cd: /home/ksjfksjd: 没有那个文件或目录
   ```

3. || 

   用于命令的分隔，只有当前一个命令执行失败，后面的命令才会执行



### shell中的通配符（元字符）表示的不是本意

* \* 任意多个字符

  ```shell
  [root@lee home]# ls *.txt
  aaa.txt  a.txt  b.txt
  ```

* ？匹配任意一个字符

  ```shell
  [root@lee home]# ls
  aaa.txt  abcd  a.txt  b.txt  shell  wl
  [root@lee home]# rm -f ?.txt
  [root@lee home]# ls
  aaa.txt  abcd  shell  wl
  ```

* [] 匹配括号中任意一个字符

  ```shell
  [root@lee home]# ls
  aaa.txt  abcd  liive.txt  live.txt  love.txt  shell  wl
  [root@lee home]# ls l[a-z]ve.txt
  live.txt  love.txt
  ```

* () 在子shell中执行（cd /home/wl）

  ```shell
  [root@lee ~]# (a=1;echo $a)
  1
  [root@lee ~]# echo $a
  
  [root@lee ~]#
  ```

* {} 集合 touch file{1..9} -- 创建了file1、file2、file3....文件

  ```shell
  [root@lee test]# cp a.txt{,.old}
  [root@lee test]# ls
  a.txt  a.txt.old  file-guoxiaolv  file-wangdahong
  ```

* \ 转义字符，让字符回归本意

  ```shell
  curl http://localhost:8080/person?id=5\&name=lee
  # 执行上面的命令的时候就需要用到转义字符
  
  # 我们知道\符号在这里可以理解为换行符，其实我们可以把它当作enter的转义 
  root@lee test]# touch abc \
  > echo abc >> abc
  ```

### sudo命令

Linux的sudo命令以系统管理员的身份执行命令，也就是说命令之前加上了sudo以后执行命令就好像是root用户亲自执行命令一样。

### echo

> NAME
>        echo - display a line of text
>
> SYNOPSIS
>        echo [SHORT-OPTION]... [STRING]...
>        echo LONG-OPTION
>
> DESCRIPTION
>        Echo the STRING(s) to standard output.
>
>        -n     do not output the trailing newline
>        -e     enable interpretation of backslash escapes 启用反斜杠转义的解释功能
>        -E     disable interpretation of backslash escapes (default)

```shell
root@lee ~]# echo "a\nb"
a\nb
[root@lee ~]# echo -e "a\nb"
a
b
```

带有颜色的输出(30-37是文本的颜色，40-47是文字背景的颜色)

```shell
[root@lee ~]# echo -e "\e[1;31mHello mac. \e[0m"
Hello mac.
[root@lee ~]# echo -e "\e[1;32mHello mac. \e[0m"
Hello mac.
[root@lee ~]# echo -e "\e[1;33mHello mac. \e[0m"
Hello mac.
[root@lee ~]# echo -e "\e[1;34mHello mac. \e[0m"
Hello mac.
[root@lee ~]# echo -e "\e[1;35mHello mac. \e[0m"
Hello mac.
```



### prinft 格式化输出

`printf FORMAT [ARGUMENT]...` FORMAT代表格式，ARGUMENT是参数

```shell
[root@lee ~]# printf "%-10s %-10s %-10s %-10s\n" 姓名 年龄 身高 体重
姓名     年龄     身高     体重
[root@lee ~]# printf "%-10s %-10s %-10s %-10s\n" 张三 18 177 130
张三     18         177        130
[root@lee ~]# printf "%-10s %-10s %-10s %-10s\n" 李四 28 190 150
李四     28         190        150
```

* \- 代表左对齐 +或者不填代表右对齐 s字符串 d整数 f小数 c单个字符
* 10代表宽度，如果宽度不够会用空格补齐，超出宽度也会原样输出



### read 命令

`read -p "Please input ip: " ip` -p代表给出一个提示，在键盘输入的内容按下回车键以后会赋值给ip这个变量。

### shell中的变量

1. 自定义变量

   定义变量：变量名=xxx

   引用变量：$变量名 或者 ${变量名}

   查看变量：echo $变量名；set（查看所有的变量，包含自定义变量和环境变量）

   取消变量：unset 变量名

   作用范围：**只在当前shell中起作用（子shell中也不起作用）**

   > 通过命令替换的方式来给变量赋值，通过反引号或者$(command)的方式
   >
   > ```shell
   > [root@lee wl]# a=\`date +%F\`
   > [root@lee wl]# echo $a
   > 2021-08-07
   > [root@lee wl]# b=$(date +%Y%m%d)
   > [root@lee wl]# echo $b
   > 20210807
   > 
   > ```
   >
   > 通过read对变量进行赋值
   >
   > ```shell
   > [root@lee wl]# read i
   > 123
   > [root@lee wl]# echo $i
   > 123
   > ```
   >
   > 

2. 环境变量

   定义变量：export 变量名=xxx；或者 分作两步：变量名=xxx；export 变量名

   引用变量：$变量名 或者 ${变量名}

   查看变量：echo $变量名；set（查看所有的变量，包含自定义变量和环境变量）；env（查看所有的环境变量）

   取消变量：unset 变量名

   作用范围：**只在当前shell和子shell中起作用**

3. 位置变量

   $1 $2 $3 $4..... $0是shell脚本本身的名字

4. 预定义变量

   | 变量名 | 变量含义                                              |
   | ------ | ----------------------------------------------------- |
   | $0     | 脚本名（带有路径，可以使用basename $0来输出文件名字） |
   | $*     | 所有的参数                                            |
   | $@     | 所有的参数                                            |
   | @#     | 参数的长度                                            |
   | $$     | 当前进程的PID                                         |
   | $!     | 上一个后台进程的PID                                   |
   | $?     | 上一个命令返回的结果 0表示成功                        |

   注：$*会把所有的位置参数当做一个整体，它是一个string；$@会把所有的位置参数当做一个个单独的字段，它是一个列表。























