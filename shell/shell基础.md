## 各种语言的执行的区别

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

   



















