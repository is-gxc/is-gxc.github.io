---
title: 高频Linux命令
date: 2024-09-09 09:48:50
categories: 操作系统
tags: [Linux, 操作系统]
---

# Linux目录结构介绍

- **bin**：是binary的缩写，主要存放一些常用的命令，比如ls、cp、mv之类
- boot：主要存放一些linux启动时需要用到的核心文件
- **dev**：是device的缩写，主要存放一些linux的设备文件
- **etc**：主要存放系统用户所需要的配置文件和子目录
- **home**；主要存放用户目录
- **lib**：是library的缩写，主要存放一些动态库，供应用程序调用
- lost+found：一般是空的，当系统非法关机后，相关文件会存在此目录
- **media**：自动挂在一些linux系统自动识别的设备，比如U盘，光驱等
- **mnt**：提供给用户的用于挂载临时别的文件系统（手动挂载），比如另外的硬盘
- opt：提供给主机额外安装软件所需要的目录
- proc：虚拟的目录，是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息
- **root**：超级用户的主目录
- sbin：s是super user的简称，此目录主要存放一些系统管理员所用到的系统管理程序
- srv：主要存放一些系统启动之后要用的数据
- run：主要存放一些系统运行时所需要的一些文件
- **usr**：主要存放一些用户的应用程序及文件，类似windows下的program files。bin 存放系统用户所使用的应用程序、sbin 存放超级用户所使用的高级程序及系统守护程序、src 内核源代码默认的放置目录
- tmp：存放一些临时文件
- var：存放一些经常被修改的文件，比如日志文件、电子邮件。
- ...

# Linux系统文件或目录颜色的含义

- 白色——普通文件
- 蓝色——目录
- 绿色——可执行文件
- 青色——链接文件
- 红色——压缩文件
- 黄色——设备文件
- 灰色——其他文件

# Linux系统常用终端快捷键

- ctrl + f 向前移动光标
- ctrl + b 向后移动光标
- ctrl + a 光标移动到行首
- ctrl + e 光标移动到行位
- ctrl +p 向上翻页，相当于pageUp
- ctrl + n 向下翻页，相当于pageDown
- ctrl + h 删除光标前一个字符
- ctrl + d 删除光标所在字符
- ctrl + u 删除光标至开始全部内容（不包括光标所在字符）
- ctrl + k 删除光标至末尾全部内容（包括光标所在字符）
- ctrl + w 删除光标前单词
- ctrl + y 粘贴使用 ctrl + w、ctrl + u、ctrl + k快捷删除的文本

# Tab键的作用

## 作用

提示可能要输入的命令/路径

补全命令/路径

## 用法

如果命令/路径是唯一的，则敲击一次tab键自动补全

如果命令/路径不唯一，则敲击两次，将全部可能结果列出来

# 通配符介绍

- *：匹配0个或多个字符串
- ?：匹配1个字符
- [abcd]：匹配abcd中任意一个字符
- [a-z]：匹配a-z范围里的任意一个字符
- [!abc]：不匹配方括号内任何字符，与\[^abc]一致

# touch

创建空文件与修改时间戳命令

## 作用

1. 改变已有文件的时间戳属性「注意：在修改文件的时间属性的时候，用户必须是文件的属主，或拥有文件的访问权限」
2. 创建新的空文件

## 语法

touch [OPTION] FILE

OPTION:

- -a：改变文件的读取时间记录
- -m：改变文件的修改时间记录
- -r：使用参考文件的时间记录，与 --file的效果一样
- -c：不创建新文件
- -d：使用指定字符串表示时间式
- -t：设定文件的时间格式，格式与date命令相同
- --no-create：不创建新文件

## 案例

创建一个空文件 `touch file.txt` 「前提：file.txt文件不存在」

创建多个空文件 `touch file1.txt file2.txt file3.txt 、touch file{1..3}.txt`

修改已有文件的时间戳为当前系统时间（包括修改时间及访问时间）`touch file.txt`

修改文件的access时间 `touch -a file.txt`

修改文件的modify时间 `touch -m file.txt`

强制避免创建新文件 `touch -c nofile.txt 、touch --no-create nofile.txt`

将访问和修改时间从一个文件复制到另一个文件 `touch file -r ref_file.txt`

修改文件时间为明天 `touch -d "tomorrow" file.txt`

修改文件时间为任意时间 `touch -t 202409101010 file.txt` 

# mkdir

创建目录「注意：默认状态下，如果要创建的目录已经存在，则提示已存在，而不会继续创建目录，新建的目录与它所在目录下的文件有重名也不行」

## 语法

mkdir [OPTION] DIRECTORY

OPTION:

- -p：递归创建多级目录
- -m：简历目录的同时设置目录的权限
- -v：显示目录的创建过程

## 案例

在当前目录下创建一个名为 dir1的目录 `mkdir dir1`

创建多个目录`mkdir dir2 dir3 dir4 、mkdir dir{5..7}`

在dir1目录下建立子目录dir10，并设置文件权限700 `mkdir -m 700 dir1/dir10`

显示目录创建过程 `mkdir -v dir{11..13}`

# rm

删除文件或目录

## 语法

rm [OPTION] [FILE]

OPTION：

- -f：忽略不存在的文件，不会出现警告信息
- -i：删除前会询问用户是否操作
- -r/R：递归删除
- -v：显示指令执行详细执行过程

## 案例

删除一个文件 `rm file.txt`

删除一个目录 `rm -r dir`

强制删除文件/目录（不带提示） `rm -f file.txt`

删除当前目录下所有文件 `rm -rf *`

删除前注意询问 `rm -i file1.txt file2.txt file3.txt`

# rmdir

删除空目录

## 语法

rmdir [OPTION] DIRECTORY

OPTION：

- -p：用递归的方式删除指定的目录路径中的所有父级目录，非空则报错
- -v：显示命令的详细执行过程

## 案例

删除空目录 `rmdir dir`

递归删除多重目录 `rmdir -p dir1/dir2/dir3`

显示指令详细执行过程 `rmdir -v dir`

# mv

移动文件、重命名文件

## 语法

mv [OPTION] [-T] SOURCE DEST

mv [OPTION] SOURCE DIRECTORY

mv [OPTION] -t DIRECTORY SOURCE 

OPTION：

- -i：若存在同名文件，则向用户询问是否覆盖
- -f：覆盖已有文件，不进行任何提示
- -b：当文件存在时，覆盖前为其创建一个备份
- -u：当源文件比目标文件新，或者目标文件不存在时，才执行移动此操作

## 案例

将文件file.txt移动到目录dir中 `mv file.txt /dir`

将file.txt重命名为newfile.txt `mv file.txt newfile.txt`

将目录dir1移动到目录dir2中（前提是目录dir2已经存在，若不存在则改名） `mv dir1 dir2`

将文件file1.txt改名为file2.txt，如果file2.txt已经存在，则询问是否覆盖`mv -i file1.txt file2.txt`

将文件file1.txt改名为file2.txt，如果file2.txt已经存在，则直接覆盖 `mv -f file1.txt file2.txt`

文件覆盖前做备份 `mv -b file1.txt file2.txt`

源文件比目标文件新时才执行更新 `mv -u file1.txt file2.txt`

移动当前文件夹下的所有文件到上一级目录 `mv * ../`

将当前目录的一个子目录里的文件移动到另一个子目录 `mv dir3/* dir2`

# cp

复制文件或者目录

## 语法

cp [OPTION] [-T] SOURCE DEST

cp [OPTION] SOURCE DIRECTORY

cp [OPTION] -t DIRECTORY SOURCE

OPTION:

- -f：若目标文件已存在，则会直接覆盖
- -i：若目标文件已存在，则会询问是否覆盖
- -a：通常在复制目录时使用，它保留链接、文件属性，并复制目录下所有内容
- -r：递归复制文件和目录
- -b：覆盖已存在目标文件前将目标文件备份
- -v：详细显示cp命令的执行过程

## 案例

复制文件 `cp file1.txt file2.txt`

复制目录 `cp -r dir1 dir2`

复制文件，若目标文件已存在，则询问是否覆盖 `cp -i file1.txt file2.txt`

复制文件，若目标文件已存在，则直接覆盖，不提示 `cp -f file1.txt file2.txt`

复制文件，若目标文件已存在，则先进行备份 `cp -b file1.txt file2.txt`

复制目录，并且保留目录所有属性都一致 `cp -a dir1 dir2`

# cd

切换目录

## 语法

cd [参数] 目录

### 特殊目录

- ~：用户家目录
- .：当前目录
- ..：当前目录的上一级目录
- /：根目录
- -：上次所在目录

### 相对路径和绝对路径

起始路径为 / 的称为绝对路径

起始路径不是 / 的称为相对路径

## 案例

切换到dir目录 `cd dir`

切换到上两级目录 `cd ./..`

切换到家目录 `cd ~ 或者cd `

切换到上一次所在目录 `cd -`

# pwd

显示当前路径

## 语法

pwd

# ls

显示目录信息

## 语法

ls [OPTION] [FILE]

OPTION：

- -a：显示所有文件及目录（包括以.开头的隐藏文件）
- -i：使用长格式列出文件及目录信息
- -r：将文件以相反次序显示（默认以英文字母次序）
- -t：根据最后修改时间排序
- -h：以人类可读的格式显示数字大小
- -A：同-a，但不列出 .（当前目录）及 ..（父目录）
- -S：根据文件大小排序
- -R：递归列出所有子目录
- -F：在列出的文件名称后加一符号，例如可执行的加 *，目录则加 /

## 案例

列出当前目录所有文件（包括隐藏文件） `ls -a`

列出当前目录文件的详细信息 `ls -l`

列出根目录 / 下的所有文件 `ls /`

列出当前目录下所有文件和目录的详细信息（包括子目录内容）`ls -lR`

列出当前目录下所有f开头的文件的详细信息 `ls -l f*`

列出当前目录下所有文件，并根据修改时间倒序排列 `ls -ltr`

列出目前工作目录下所有文件及目录，目录于名称后加 /，可执行文件于名称后加 * `ls -AF`

列出档期啊内目录详细信息并以可读大小显示文件大小 `ls -lh`

# tree

以树状图列出目录内容

## 语法

tree [参数]

参数：

- -a：显示所有文件和目录
- -L：层级显示
- -t：用文件和目录的更改时间排序
- -r：用文件和目录的更改时间倒序排序
- -f：在每个文件或目录之前，显示完整的相对路径名称
- -d：显示目录名称而非内容

## 案例

显示以当前目录下的文件和目录 `tree`

显示所有文件和目录 `tree -a`

只显示目录 `tree -d`

只显示n层目录（n为数字，假设n为2） `tree -L 2`

显示权限信息 `tree -p`

用文件和目录的更改时间排序 `tree -t`

以相反次序排序 `tree -r`

显示完整的相对路径 `tree -f`

# stat

显示文件或文件系统的详细信息

## 语法

stat [OPTION]  FILE

OPTION：

- -L：支持符号链接
- -f：显示文件系统的信息
- -t：以简洁的方式输出

## 三种时间

atime：access time 访问时间，读取文件（more、less、cat、tail）、修改文件（vim、nano）时改变

mtime：modify time 修改时间，修改文件（vim/nano）时改变

ctime：change time状态改变时间，修改文件（vim、nano）、文件属性变化（chmod、chown）时改变

## 案例

查看文件file.txt的信息 `stat file.txt`

查看file.txt文件所在文件系统信息 `stat -f file.txt`

以简洁方式输出信息 `stat -t file.txt`

# rename

用字符串替换的方式批量改变文件名

## 语法

rename 's/old-name/new-name/' files

元字符串：将文件名需要替换的字符串

目标字符串：将文件名中含有的原字符串替换成目标字符串

文件：指定要改变文件名的文件列表

### 参数

- -n：模拟运行，实际命令没进行重命名
- -v：输出每步执行信息
- -f：强制改写

### 通配符

？可替代单个字符

\* 可替代多个字符

## 案例

将myfile.txt 改为 myfile.doc `rename 's/.txt/.doc' myfile.txt`

模拟将file1.txt，file2.txt，file3.txt改为file01.txt，file02.txt，file03.txt  `rename -n 's/file/file0/ file*'`

实际更改上一步，并输出执行过程信息 `rename -v 's/file/file0/' myfile.txt`

# basename

提取文件路径名的文件名

## 语法

basename NAME [SUFFIX]

basename OPTION NAME

参数：

- -a：支持处理多个文件
- -s：删除指定后缀

## 案例

提取文件名 `basename /etc/passwd`

提取目录名（删除最后的/） `basename /usr/local/`

提取多个输入 `basename -a /etc/passwd /usr/local`

删除指定后缀 `basename /etc/sysctl.conf .conf 或 base -s .conf /etc/sysctl.conf`

# dirname

去除文件名中的非目录部分

## 语法

dirname [OPTION] NAME

## 案例

提取文件的路径 `dirname /usr/bin/cat`

提取目录的路径 `dirname /home/work/test`

# file

识别文件类型

## 语法

file [参数] 文件

参数：

- -b：列出文件类型，不显示文件名称
- -c：详细显示指令执行过程
- -f：指定名称文件，显示多个文件类型信息
- -L：直接显示符号链接所指向的文件类型
- -m：指定魔法数字文件
- -z：尝试去解读压缩文件的内容
- -i：显示MIME类别

## 案例

查看文件的类型 `file file.txt   file dir   file /dev/sda`

查看文件类型，但不显示文件名 `file -b file.txt`

查看某个符号链接文件（快捷方式）的类型 `file /dev/cdrom`

直接查看某个符号链接文件（快捷方式）所对应的目标文件的文件类型 `file -L /dev/cdrom`

# md5sum

生成和校验文件的md5值

## 语法

md5sum [OPTION] [FILE]

OPTION：

- -b：以二进制模式读取文件
- -t：以文本模式读入文件内容
- -c：根据已生成的md5值，对现存文件进行校验
- --status：校验完成后，不生成错误或正确的提示信息，可以通过命令的返回值来判断

## 案例

生成file.txt文件的md5值 `md5sum file.txt`

不同模式计算同一个文件的md5值 `md5sum -b file.txt  md5sum -t file.txt`

将生成md5值重定向到指定的文件 `md5sum file.txt > file.md5`

md5校验 `md5sum -c file.md5`

md5校验，不显示任何输出，用返回码表示成功与否 `md5sum -c --status file.md5   然后 echo $?`

# find

搜索指定文件

## 语法

find [路径] [参数] [条件] 

参数：

- -name name，-iname name：文件名符合name的文件，iname会忽略大小写
- -perm 匹配权限（mode为完全匹配-mode为包含即可）
- -user 匹配所有者
- -group 匹配所有组
- -mtime -n +n 匹配修改内容的时间（-n指n天以内，+n指n天以前）
- -atime -n +n 匹配访问文件的时间（-n指n天以内，+n指n天以前）
- -ctime -n +n 匹配修改文件权限的时间（-n指n天以内，+n指n天以前）
- -nouser 匹配无所有者的文件
- -nogroup 匹配无所有组的文件
- -newer f1 !f2 匹配比文件f1新但比f2旧的文件
- -type b/d/c/p/l/f 匹配文件类型（后面的字母依次表示 块设备、目录、字符设备、管道、链接文件、文本文件）
- -size 匹配文件的大小（+50KB为查找超过50KB的文件，而-50KB为查找小于50KB的文件）
- -prune 忽略某个目录
- -exec ....{}\; 后面可跟用于进一步处理搜索结果的命令

## 案例

全盘搜索系统中所有以 .conf 结尾的文件 `find / -name *.conf`

在/etc 目录中搜索所有大约 1k 大小的文件 `find /etc -size +1k`

在 /home 目录中搜索所有属于指定用户的文件 `find /home -user xxx`

搜索当前目录下所有的文件 `find . -type f`

搜索当前目录下所有权限为 664的文件，并列出 `find . -perm 664 -exec ls -l {}\;`

列出当前工作目录中的所有文件、目录以及子文件信息 `find .`

在当前目录下搜索所有指定后缀的文件，后缀不需要大小写 `find . -name "*.txt"`

在当前目录下下搜索所有后缀不是 .txt的文件 `find . ! -name "*.txt"`

搜索当前工作目录的所有7天内被修改过的文件，并删除  `find . -mtime -7 -exec rm --i {}\;`

# which

查找命令文件所在位置

## 语法

which [-a] filename 

## 案例

查找文件所在位置 `which bash  which is  which fdisk`

# chown

改变文件或目录的用户和用户组

## 语法

chown [参数] 所属主:所属组 文件

参数：

- -R 对目前目录下的所有文件与目录进行相同的变更
- -c 显示所属信息变更信息

## 案例

改变file.txt文件的所属主与所属组 `sudo chown xxx:xxx file.txt`

只改变file.txt文件的所属主 `sudo chown xxx file.txt`

只改变file.txt文件的所属组 `sudo chowm :xxx file.txt`

改变指定文件的所属主与所属组，并显示过程 `chown -c xxx:xxx file.txt`

改变指定目录及其所有子文件的所属主与所属组 `sudo chown -R xxx:xxx dir`

# chgrp

更改文件用户组

## 语法

chgrp [参数] [目录]

参数：

- -v：显示指令执行过程
- -R：递归处理，并将指定目录下的所有文件及子目录一并处理
- --reference：把指定文件或目录的所属群组全部设成参考文件或目录的所属群组

## 案例

改变文件的用户组 `sudo chgrp xxx file.txt`

改变文件的用户组，并显示命令执行过程 `sudo chgrp -v xxx file.txt`

根据参照文件改变文件的用户组 `sudo chgrp --reference=reffile.txt file.txt`

将dir及其子目录下的所有文件的用户组改为xxx `sudo -R xxx dir`

# chmod

改变文件或目录的权限（只有自己是文件或目录的属主、或者是root用户才能执行）

## 语法

chomd [OPTION] MODE[MODE] FILE

chmod [OPTION] OCTAL -MODE FILE

chmod [OPTION] --reference=RFILE FILE

### 两种模式

1. 符号模式

   格式： [ugoa\][+-=] [rwx]

   对象：

   | 对象 | 用户类型 | 说明                |
   | ---- | -------- | ------------------- |
   | u    | user     | 文件所有者          |
   | g    | group    | 文件所有者所在组    |
   | o    | other    | 所有其他用户        |
   | a    | all      | 所有用户，相当于ugo |

   操作：

   | 操作 | 说明                                                   |
   | ---- | ------------------------------------------------------ |
   | +    | 为指定的用户类型增加权限                               |
   | -    | 为指定的用户类型去除权限                               |
   | =    | 设置指定用户权限的设置，即将用户类型的所有权限重新设置 |

   权限：

   | 权限 | 名字 | 说明             |
   | ---- | ---- | ---------------- |
   | r    | 读   | 设置为读权限     |
   | w    | 写   | 设置为写权限     |
   | x    | 执行 | 设置为可执行权限 |

2. 数字模式

   ![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/linux_cmd_chmod.png)

参数：

- -R：对当前目录下的所有文件与子目录进行相同的权限变更（即以递归的方式逐个变更）

## 案例

将文件file.txt设置为所有人都可读取 `chmod a+r file.txt`

将当前目录下所有文件及递归目录文件设置为所有人可读取 `dhmod -R a+r *`

将file.txt设置为只有该文件拥有者才可以执行 `chmod -u+x file.txt`

将file.txt设置为文件拥有者及其同组人员可写入，但其他人不可写入 `chmod ug+w, o-w file.txt`

将file.txt设置为所有人都可读可写可执行 `chmod a+r,a+w,a+x file.txt  chmod 777 file.txt`

将file.txt设置为文件拥有者可读可写可执行 `chmod 755 file.txt`

将file.txt设置为文件拥有者可读可写可执行，此外其他人都没任何权限 `chmod u=rw, go= file.txt`

# grep

文本搜索工具

## 语法

grep [参数] 文件

参数：

- -i：忽略大小写
- -c：只输出匹配行的数量
- -l：只列出匹配的文件名，不列出具体的匹配行
- -n：列出所有的匹配行，显示行号
- -h：查询多文件时不显示文件名
- -s：不显示不存在、没有匹配文本的错误信息
- -v：显示不包含匹配文本的所有行
- -w：匹配整词
- -x：匹配整行
- -r：递归搜索
- -q：禁止输出任何结果，已退出状态表示搜索是否成功
- -b：打印匹配行距文件头部的偏移量，以字节为单位
- -o：与-b结合使用，打印匹配的词距文件头部的偏移量，以字节为单位
- -F：匹配固定字符串的内容
- -E：支持扩展的正则表达式

## 案例

搜索某个文件中，包含某个关键词的内容 `grep xxx /etc/passwd`

搜索多个文件中，包含某个关键词的内容 `grep xxx /etc/passwd /etc/shadow`

搜索多个文件中，包含某个关键词的内容，不显示文件名称 `grep -h xxx /etc/passwd /etc/shadow`

递归搜索，不仅搜索指定目录，还搜索其子目录内是否有关键词文件 `grep -r hello *`

输出在某个文件中，包含某个关键词行的数量 `grep -c root /etc/passwd /etc/shadow`

忽略大小写 `grep -i XXX /etc/passwd`

在文件中查找要搜索的内容，并显示行号 `grep -n xxx /etc/passwd`

反向查找 `grep -v xxx /etc/passwd`

搜索当前工作目录中，包含某个关键词内容的文件，未找到的则提示 `grep -l hello *`

搜索某个文件中，精准匹配到某个关键词的内容 `grep -x hello file1.txt`

判断某个文件中，是否包含某个关键词，通过返回状态输出结果（0未包含，1未不包含），方便在shell脚本中判断和调用`grep -q hello file1.txt`

# egrep

查找指定的字符串

## 语法

egrep [参数] [文件]

参数 同grep

## 案例

查找包含一个或一个以上 "a" 的内容 `egrep 'a+' file.txt`

查找包含linux或xxx的内容 `egrep 'linux|xxx' file.txt`

查找包含linux整体的内容 `egrep '(linux)' file.txt`

查找包含一个或多个linux整体的内容 `egrep '(linux)+' file.txt`

查找以#开头的内容 `egrep '^#' file.txt`

查找以linux结尾的内容 `egrep 'linux$' file.txt`

查找连续出现3次ab的内容 `egrep '(ab){3}' file.txt`

查找包括abc或abd的内容 `egrep 'ab[cd]' file.txt`

# cat

在终端上显示文件内容

## 语法

cat [OPTION] [FILE]

OPTION：

- -n：显示行数（空行也编号）
- -s：去除重复的行
- -b：显示行数（空行不编号）
- -E：每行结束处显示$符号
- -T：将TAB字符显示为 ^| 符号

## 案例

查看file.txt文件内容 `cat file.txt`

查看多个文件内容 `cat file1.txt file2.txt`

查看file.txt文件内容，并显示行数编号 `cat -n file.txt`

去除重复的空行 `cat -s file.txt`

重定向文件内容 `cat file.txt > file.txt`

将file1.txt和file2.txt合并为一个文件 `cat file1.txt file2.txt > combinedifile.txt`

使用cat创建文件 `cat > file.txt`

# more

分页显示文本文件内容

## 语法

more [OPTION] file

基本操作：回车 下滚一行 空格 下翻一页

OPTION：

- -num：指定每屏显示的行数
- +num：从第nun行开始显示
- -p：先清除屏幕再显示文本文件的剩余内容
- -c：与-p相似，不滚屏，先显示内容再清楚旧内容
- -s：多个空行压缩成一行显示

## 案例

分页显示指定的文本文件内容`more ~/.bashrc`

先进行清屏操作，随后以每次10行内容的格式显示指定的文本文件内容 `more -c 10 ~/.bashrc`

分页显示指定的文本文件内容，遇到连续两行以上空白行的情况，则以一行空白行显示 `more -s ~/.bashrc`

从第10行开始，分页显示指定的文本文件内容 `more +10 ~/.bashrc`

# less

分页显示文本内容

## 语法

less [参数] 文件

参数：

- -N：显示每行的行号。
- -i：忽略大小写进行搜索。
- -R：以原始格式显示文件内容，不解释特殊字符。
- -F：强制less一次性显示整个文件，不进行分页。
- /：用于搜索指定的字符串。
- ?：用于向后搜索指定的字符串。
- h：显示帮助信息。

快捷操作

- 空格键：下翻一页
- 回车键：下滚一行
- j：下滚一行
- k：上滚一行
- b：回翻一页
- f：下翻一页
- q：退出
- /word：搜索word关键词
- n：跳转下一个搜索匹配项
- N：跳转上一个搜索匹配项
- g：跳转到文件开头
- G：跳转到文件末尾

## 案例

查看文件 `less ~/.bashrc`

查看多个文件 `less ~/.bashrc ~/.bash_history` :n浏览下一个文件 :p浏览前一个文件

查看历史命令并使用less命令分页显示 `history | less`

# head

显示文件开头的内容

参数：

- -n：加数字，定义显示行数
- -c：加数字，指定显示头部内容的字符数

## 案例

显示文件的前10行内容（默认） `head ~/.bashrc`

显示文件的前5行内容`head -n ~/.bashrc `

显示文件除了最后6行的全部内容 `head -n -6 ~/.bashrc`

显示文件前20个字符 `head -c 20 ~/.bashrc`

显示文件除了最后30个字符的全部内容`head -c -30 ~/.bashrc`

# tail

查看文件尾部内容

## 语法

tail [OPTION] [FILE]

参数：

- -c：N输出文件尾部N个字节内容
- -f：显示文件最新追加的内容
- -n：N输出文件的尾部N行内容

## 案例

显示file.txt文件的最后10行内容 `tail file.txt`

显示file.txt文件的最后20行内容 `tail -n 20 file.txt`

动态显示文件的最后10行内容 `tail -f file.txt`

# tac

反向显示文件的内容，与cat相反，先显示倒数第一行，依次到第一行

## 语法

tac [参数] [FILE]

## 案例

`tac file.txt`

# nl

添加行号

## 语法

nl [OPTION] [FILE]

OPTION：

- -b a：也给空行添加行号(类似cat -n)
- -b t：空行不显示行号
- -n：列出行号表示的方法。「-n nl 行号显示在屏幕的左方显示」「-n rn 行号显示在自己栏位的最右方显示，且不加0」「-n rz 行号在自己栏位的最右方显示，且加0」
- -w：行号栏位的占用位数

## 案例

用nl列出file.txt的内容 `nl file.txt`

用nl列出file.txt的内容，空行也加上行号 `nl -b a file.txt`

行号在自己栏位的最右方显示，且加0对齐格式 `nl -b a -n rz file.txt`

行号宽度设置为3 `nl -b -a -n rz -w 3 file.txt`

空行不显示行号 `nl -b file.txt`

# WC

统计文本信息

## 语法

wc [OPTION] [FILE]

OPTION：

- -w：统计字数，或--words，只显示字数，一个字被定义为由空白、跳格或换行字符分隔的字符串
- -c：统计字节数，或--bytes或--chars，只显示Bytes数
- -l：统计行数，或--lines，只显示行数
- -m：统计字符数
- -L：打印最长行的长度（不包含不可见字符）

## 案例

统计file.txt文件的行数、字数、以及字节数 `wc file.txt`

统计file.txt文件的字数 `wc -w file.txt`

统计file.txt文件的字符数 `wc -m file.txt`

统计file.txt文件的字节数 `wc -c file.txt`

统计file.txt文件的行数 `wc -l file.txt`

打印file.txt文件最长行的长度 `wc -L file.txt`

使用管道符统计文本行数 `cat file.txt | wc -l`

# split

文件分割

## 语法

split [OPTION] [FILE [PREFIX]]

OPTION:

- -<行数> 或 -l 行数：指定每多少行切成一个小文件
- -b<字节>：指定每多少个字节切成一个小文件
- -d：使用数字作为后缀
- -a：指定后缀长度（默认为2）
- [输出文件名]设置切割后文件的前置文件名，split会自动在前置文件名后再加上编号

## 案例

将file.txt每2行切割成一个小文件 `split -2 file.txt`

将file.txt每10kb切割成一个小文件 `split -b 10k file.txt`

以数字作为后缀，并指定后缀宽度为3 `split -b 10k -d -a 3 file.txt`

为分割后的文件指定文件名前缀 `split -b 10k -d -a 3 file.txt split_file`

# cut

从文件中提取文本的一部分

## 语法

cut [OPTION] [FILE]

OPTION:

- -b：以字节为单位进行分割，仅显示行中指定直接范围的内容
- -c：以字符为单位进行分割，进现实行中指定范围的字符
- -d：自动以分隔符，默认为制表符"TAB"
- -f：显示指定字段的内容，与-d一起使用
- -n：取消分割多字节字符
- --complement：补足被选择的字节、字符或字段
- --out-delimiter：指定输出内容是字段分隔符

## 案例

提取file1.txt第2列的内容 `cut -f 2 file.txt  或  cut -f2 file.txt`

提取filt.txt除第2行之外的其他内容 `cut -f2 --complement file.txt`

使用-d选项指定字段分隔符 `cut -f2 -d"." file.txt`

提取第2,3,4,6个字符 `cut -b 2-4, 6 file.txt`

提取指定数量字符：

	- 提取第一个到第三个字符 `cut -c1-3 file.txt`
	- 提取前两个字符 `cut -c -2 file.txt`
	- 提取第4个之后的字符 `cut -c4- file.txt`

# paste

合并两个或多个文件

## 语法

paste [OPTION] [FILE]

OPTION：

- -d：默认域的分隔符是空格或者tab键，设置行的域分隔符
- -s：将每个文件粘贴成一行
- -：从标准输入中读取数据

## 案例

将file1.txt和file2.txt粘贴成一个新的文件 `paste file1.txt file2.txt`

顺序不一样，结果不一样 `paste file2.txt file1.txt`

多文件拼接 `paste file1.txt file2.txt file3.txt`

设置域分隔符为":" 粘贴成新的文件 `paste -d":" file1.txt file2.txt`

将每个文件粘贴成一行 `paste -d":" -s file1.txt file2.txt`

从标准输入中读取数据，每行显示3个文件名 `ls | paste -d "" - - - -`

# sort

对文件内容进行排序

## 语法

sort [参数] 文件

参数：

- -n：依照数值的大小排序
- -t <分隔字符>：指定排序时所用的栏位分割字符
- -k：指定需要排序的栏位
- -f：以相反的顺序来排序

## 案例

按照字母顺序排序 `sort file.txt`

反向排序 `sort -r file.txt`

按照数字大小排序 `sort -n file.txt`

以冒号为间隔符，对指定的文件内容按照数字大小对第三列进行排序 `sort -t : -k 3 -n file.txt`

# uniq

去除文件中的重复行

## 语法

uniq [OPTION] [INPUT] [OUTPUT]

OPTION：

- -c：打印每行在文本中重复出现的次数
- -d：只显示有重复的记录，每个重复记录只出现一次
- -u：只显示没有重复的记录

## 案例

删除连续文件中连续的重复行 `uniq file.txt`

打印每行在文件中出现重复的次数 `uniq -c file.txt`

只显示有重复的记录，且每个记录只出现一次 `uniq -d file.txt`

只显示没有重复的记录 `uniq -u file.txt`

# diff

比较文件的差异

## 语法

diff [OPTION] FILES

OPTION：

- y：以并列的方式显示文件的异同之处
- c：显示全部内文：并标出不同之处
- u：以合并的方式来显示文件内容的不同
- -W：设置宽度

显示提示：

- a - add
- c - change
- d - delete
- | 前后2个文件内容有不同
- < 后面文件比前面文件少了1行内容
- \> 后面文件比前面文件多了1行内容
- \+ 比较的文件的后者比前者多一行
- \- 比较的文件的后者比前者少一行
- ! 比较的文件两者有差别的行

## 案例

比较两个文件 `diff file1.txt file2.txt`

并排格式输出 `diff -y -W 50 file1.txt file2.txt`

上下文格式输出 `diff -c file1.txt file2.txt`

统一格式输出 `diff -u file1.txt file2.txt`

生成补丁 `diff file1.txt file2.txt > file.patch`

打补丁 `patch file1.txt file.patch`

# join

连接两个文件

## 语法

join [OPTION] FILE1 FILE2

OPTION：

- -a1或-a2：-a1或-a2除了显示共同域的记录之外，-a1显示第一个文件没有共同域的记录，-a2显示第二个文件中没有共同域的记录
- -o：设置结果显示的格式
- -t：改变域的分隔符
- -v1或-v2：-v1或-v2不显示共同域的记录之外，-v1显示第一个文件没有共同域的记录，-v2显示第二个文件中没有共同域的记录
- -j：指定一个域作为匹配字段

## 案例

- 连接两个文件：

  - 默认以第一列作为连接字段 `join file1.txt file2.txt`

  - 显示左边文件中的所有记录(右边文件中没有匹配的不显示) `join -a1 file1.txt file2.txt`

  - 显示右边文件中的所有记录(左边文件中没有匹配的不显示) `join -a2 file1.txt file2.txt`

  - 全连接(显示左边和右边所有记录) `join -a1 -a2 file1.txt file2.txt`

- 指定输出字段(第一个文件第二列、第三列 第二个文件第三列) `join -o 1.2 1.3 2.3 file1.txt file2.txt`
- 指定分隔符（默认情况分隔符是空格） `join -t '' file1.txt file2.txt`
- 只显示第1个文件中没有相同栏位的行 `join -v1 file1.txt file2.txt`
- 指定了以两个文件里第2行做匹配字段 `join -j 2 file1.txt file2.txt`

# tr

转换或删除文件中的字符

## 语法

tr [OPTION] SET1 [SET2]

OPTION：

- -c：反选设定字符，也就是符合SET1的部分不做处理，不符合的剩余部分才进行转换
- -d：删除字集合中出现的所有字符
- -s：缩减连续重复的字符成指定的单个字符

可是使用的字符类：

- 字母和数字 ["alnum"]
- 字母 [:alpha:]
- 控制（非打印）字符 [:cntr:]
- 数字 [:digit:]
- 图形字符 [:graph:]
- 小写字母 [:lower:]
- 可打印字符 [:print:]
- 标点符号 [:punct:]
- 空白字符 [:space:]
- 大写字母 [:upper:]
- 十六进制字符 [:xdigit:]

## 案例

- 大小写转换
  - `tr "[a-z]" "[A-Z]" < file.txt`
  - `cat file.txt | tr a-z A-Z`
  - `echo "hello world"| tr [:lower:] [:upper:]`
  - 大小写互换`echo "Hello World"| tr '[A-Za-z]''[a-zA-Z]`
- 删除字符
  - 删除小写字母 `echo "abcABdefCD"| tr -d "[a-z]"`
  - 删除数字 `echo "abc123def888hello" | tr -d 0-9`
  - 删除空格和tab符号 `echo " Hello World" | tr -d "[ \t]"`
  - 删除不在指定集合中的字符(不加-e会认为\n是普通字符不是转义) `echo -e "abc123*&\n def888"| tr -d -c '0-9\n'`
- 删除重复字符，只保留一个
  - 压缩重复的空白行 `echo -e "1\n\n2\n\n3" | tr -s '\n'`
  - 删除字符集中的重复字符 `echo "Hellooo Linuxxxxxx" | tr -s "[xo]"`
  - 将多个连续空格合并为一个空格，并将空格替换为破折号'-' `echo "2024      10 01" | tr -s ' ' '-'`
- 将制表符转换为空格 `echo -e "hello\tworld"| tr '\t' ' '`

# sed

批量编辑文本文件

## 语法

sed [选项] [动作] 文件名

选项：

- -n：仅显示script处理后的结果
- -e：以选项中指定的script来处理输入的文本文件
- -i：此选项会直接修改源文件，要慎用

动作：

- a：新增
- c：取代
- d：删除
- i：插入
- p：打印
- s：取代

## 案例

- 输出文件内容
  - 仅输出第2行 `sed -n '2p' file.txt`
  - 输出3-5行 `sed -n '3,5p' file.txt`
  - 搜索含有an关键字的行 `sed -n '/an/p' file.txt`
- 删除文件内容
  - 删除第2-4行内容 `sed '2,4d' file.txt`
  - 删除含有an关键字的行 `sed '/an/d' file.txt`
- 在第二行后追加nice `sed '2a nice' file.txt`
- 在第二行前插入一行数据 `sed '2i 123456789' file.txt`
- 整行替换数据
  - `sed '2c asdfhg' file.txt`
  - `sed '2,5c data changed' file.txt`
- 字符串替换
  - 全局替换，把xiaoming改为xiaohong `sed 's/xiaoming/xiaohong/g' file.txt`
  - 只替换第三行 `sed '3s/xiaoming/xiaohong/g' file.txt`
  - 多个条件(-e不要漏了) `sed -e 's/xiaoming//g;s/xiaohong//g' file.txt`
  - 将操作写入文件 `sed -i '3s/xiaoming/xiaohong/g' file.txt`

# awk

文本和数据进行处理的变成语言。（命令是三个创始人姓的缩写）

## 语法

awk '条件1 {动作1} 条件2 {动作2} ...' file.txt

参数：

- -F：指定输入时用到的字段分隔符
- -v：自定义变量
- -f：从脚本中读取awk命令
- -m：对val值设置内在限制



## 案例

显示第一和第三列的内容 `df -h | awk '{print $1 "\t" $3}'`

指定冒号为分隔符，显示第1和第3列的内容 `awk -F : '{print $1 "\t" $3}' /etc/passwd`

指定冒号为分隔符，显示系统中所有UID号码大于500的用户信息（第三列） `awk -F : '$3>=500' /etc/passwd`

搜索 /etc/passwd 有xxx关键字的所有行 `awk -F : '/xxx/' /etc/passwd`

搜索 /etc/passwd 有xxx关键字的所有行，并显示对应的shell `awk -F : '/xxx/{print $7}' /etc/passwd`

BEGIN与END的使用 `head /etc/passwd | awk -F : 'BEGIN{print "name\tuid"}{print $1 "\t" $3}END{print "from file /etc/passwd"}'`

# du

查看磁盘使用空间

## 语法

du [OPTION]...[FILE]...

du [OPTION]...--files0-from=F

参数：

- -a：显示目录中所有文件大小
- -h：以易读方式显示文件大小

## 案例

列出当前目录下所有文件和目录的容量大小 `du`

以易读方式显示dir文件夹及其子文件夹大小 `du -h dir`

以易读方式显示dir文件夹内所有文件大小 `du -ah dir`

显示文件 file.txt 所占用的磁盘空间`du file.txt`

进现实目录的总大小 `du -s dir   du --max-depth=0 dir`

显示指定目录下每个文件或目录的容量大小，并且以易读方式显示 `du -sh dir`

# df

显示磁盘空间使用情况

## 语法

df [OPTION]...[FILE]...

参数：

- -a：显示所有文件系统，包含所有的具有0 Blocks的文件系统
- -h：以容易阅读的方式显示
- -i：显示inode信息
- -t：<文件系统类型>只显示指定类型的文件系统
- -T：输出时显示文件系统类型

## 案例

显示磁盘空间使用情况 `df`

以易于阅读的方式显示磁盘空间使用情况 `df -h`

显示指定文件/目录所在分区的磁盘使用情况 `df /home`

显示指定文件类型的磁盘使用情况 `df -t squashfs`

以inde模式来显示磁盘使用情况 `df -i`

显示所有信息 `df -a`

列出文件系统的类型 `df -T`

# mount

把文件系统挂载到目录

## 语法

mount [参数] [设备] [挂载点]

参数：

- -o
  - loop：用来把一个文件当成硬盘分区挂接上系统
  - ro：采用只读方式挂接设备
  - rw：采用读写方式挂接设备
  - iocharset：指定访问文件系统所有字符集
- -t：指定挂载类型

## 案例

查看当前系统中挂载的所有文件系统信息 `mount`

查看指定类型挂载的文件系统 `mount -t tmpfs`

将U盘挂载到指定目录，先使用fdisk -l查看系统有哪些磁盘`sudo fdisk -l` ` sudo mount /dev/sdb/mnt/udisk`

只读模式挂载 `sudo mount -o ro /dev/sdb /mnt/udisk`

将iso镜像挂载到 /mnt/iso 目录 `sudo mount -o loop /home/xxx/ command/mount/mydisk/iso /mnt/iso`

# umount

写在文件系统

## 语法

umount [参数]

- -v：执行时显示详细的信息

## 案例

通过设备名卸载 `umount -v /dev/sdb`

通过挂载点卸载 `umount -v /media/xxx`

# dd

拷贝及转换文件

## 语法

dd 参数 对象

## 基本格式 

dd if=path/to/input_file of=/path/to/output_file bs=block_size count=number_of_blocks

参数：

- if=文件名：输入文件名，默认为标准输入，即指定源文件
- of=文件名：输出文件名，默认为标准输出。即指定目的文件
- ibs=bytes：一次读入bytes个字节，即指定一个块大小为bytes个字节
- obs=bytes：一次输出bytes个字节，即指定一个块大小为bytes个字节
- bs=bytes：同时设置读入/输出的快大小为bytes个字节
- cbs=bytes：一次转换bytes个字节，即指定转换缓冲区大小
- skip=blocks：从输入文件开头跳过blocks个块后再开始复制
- seek=blocks：从输出文件开头跳过blocks个块后开始复制
- count=blocks：仅拷贝blocks个块，块大小等于 ibs/obs 指定的字节数
- conv=<关键字>：指定关键字
  - conversion：用指定的参数转换文件
  - ascii：转换为ebcdic 为 ascii
  - ebcdic：转换ascii为 ebcdic
  - ibm：转换ascii 为 alternate ebcdic
  - block：把每一行转换为长度 cbs，不足部分用空格填充
  - unblock：使每一行的长度都为 cbs，不足部分用空格填充
  - lcase：把大写字符转换为小写字符
  - ucase：把小写字符转换为大写字符
  - swab：交换输入的没对字节
  - noerror：出错时不停止
  - notrunc：不截断输出文件
  - sync：将每个输入块填充到ibs个字节，不足部分用空（NULL）字符补齐
  - conversion：用指定的参数转换文件

## 案例

生成一个指定大小（500M）的新文件 `dd if=/dev/zero of=file.txt bs=500M count=1`

拷贝指定文件的前50个字节 `dd if=file.txt of=file2.txt bs=50 count=1`

拷贝指定文件的内容，并将所有字符转换成大写后输出到新文件中 `dd if=file.txt of=file3.txt conv=ucase`

由标准输入设备读入字符串，并将字符串转换成大写后，再输出到标准输出设备 `dd conv=ucase (按ctrl+d结束)`

# tar

打包/解压工具

参数：

- -c：新建打包文件
- -x：解压文件，配合-C解压到对应的文件目录
- -f：(压缩或解压缩时)指定要处理的文件
- -j：通过bzip2方式压缩或解压，最后以tar.br2为后缀。压缩后大小小于tar.gz
- -z：通过gzip方式压缩或解压，最后以tar.gz为后缀
- -v：显示操作过程
- -t：查看打包文件中内容
- -C dir：指定压缩/解压缩的目录，若无指定，默认是当前目录

## 案例

将当前目录下所有.txt文件打包(未压缩)，丙烯那是操作过程 `tar -cvf files.tar *.txt`

打包文件之后，使用 gzip 方式压缩 `tar -zcvf files.tar.gz *.txt`

解压文件到当前目录 `tar -zxvf files.tar.gz`

解压文件到家目录下 `tar -zxvf files.tar.gz -C dir`

列出压缩包里的内容 `tar -tf file.tar.gz`

# zip/unzip

压缩/解压缩文件

## 语法

zip 参数 文件

unzip 参数 文件

参数：

​	zip：

	- -v：显示指令执行过程
	- -d：更新压缩包内文件
	- -r：递归处理，将指定目录下的所有文件和子目录一并处理

​	unzip

	- -l：显示压缩文件内所包含的文件
	- -v：显示指令执行过程
	- -d <目录>：指定文件解压缩后所要存储的目录

## 案例

将指定目录机器内全部文件都打包成zip格式压缩包文件 `zip -r dir.zip dir`

将当前目录下所有txt文件全部压缩成file.zip `zip files.zip *.txt`

将newfile.txt添加到files.zip压缩包 `zip -dv files.zip newfile.txt`

查看压缩文件中包含的文件 `unzip -l files.zip`

查看显示的文件列表还包含压缩比率 `unzip -v files.zip`

检查zip文件是否损坏 `unzip -t files.zip`

解压files.zip到当前目录 `unzip files.zip`

解压files.zip到指定目录 `unzip files.zip -d udir/`

# gzip/gunzip

压缩/解压文件

## 语法

gzip [参数] 文件 

gunzip [参数] 压缩包

参数：

- -d：解开压缩文件
- -k：保留源文件
- -l：列出压缩文件的相关信息
- -r：递归处理，将指定目录下的所有文件及子目录一并处理
- -v：显示指令执行过程
- -t：测试压缩文件是否正确无误

## 案例

压缩指定的文件，原文件将被删除 `gzip file.txt`

压缩指定的目录 `gzip -r dir`

显示指定压缩包的压缩信息 `gzip -l file.txt.gz`

解压指定的压缩包文件 `gzip -dv file.txt.gz  gunzip -v file.txt.gz`

递归解压目录 `gzip -dr dir.gz   gunzip -r dir.gz`

压缩指定的文件，源文件不被删除 `gzip -k file.txt`

测试指定的压缩包文件内容是否损坏，能够正常解压 `gunzip -t file.txt.gz`

# uname

显示系统信息

## 语法

uname [OPTION] ...

参数：

- -a：显示系统所有相关信息
- -m：显示计算机硬件架构
- -n：显示主机名称
- -r：显示内核发行版本号
- -s：显示内核名称
- -v：显示内核版本
- -p：显示主机处理器类型
- -o：显示操作系统名称
- -i：显示硬件平台

## 案例

显示系统主机名、内核版本、硬件架构等信息 `uname -a`

仅显示主机名 `uname -n`

仅显示内核发行版本   `uname -r`

仅显示内核名称 `uname -s`

仅显示当前系统的硬件平台 `uname -i`

打印操作系统类型 `uname -o`

# hostname

显示和设置系统的主机名

## 语法

hostname [参数]

参数：

- -a：显示主机别名
- -d：显示DNS域名
- -f：显示FQDN名称
- -i：显示主机的ip地址
- -s：显示单主机名称，在第一个点处截断
- -y：显示NIS域名

## 注意

环境变量HOSTNAME也保存了当前的主机名 `echo $HOSTANME`

使用hostname可临时设置主机名

永久修改主机名：

1. 修改配置文件 /etc/hostname   /etc/hosts
2. hostnamectl命令 `sudo hostnamectl set-hostname <newhostname>` 一定要修改 /etc/hosts

## 案例

显示主机名 `hostname`

临时改变主机名 `hostname newname`

显示主机的所有IP地址 `hostname -l`

# dmesg

显示开机信息

## 语法

dmesg [参数]

参数：

- -c：显示信息后，清除ring buffer中的内容
- -s <缓冲区大小>：预设为8196，刚好等于ring buffer的大小
- -n：设置记录信息的层级

## 案例

显示开机信息 `dmesg   dmesg | less`

显示和内存、硬盘、USB、TTY相关的信息 

	- `dmesg | grep -i memory`
	- `dmesg | grep -i dma`
	- `dmesg | grep -i usb`
	- `dmesg | grep -i tty`

显示信息级别 `dmesg -x` emerg\alert\crit\err\warn\notice\info\debug，只查看错误信息 `dmesg --level=err`

只输出特定级别的信息`dmesg --level=err,warn`

显示时间戳 `dmesg -T`

显示原始数据 `dmesg -r`

清空dmesg环形缓冲区的日志 `sudo dmesg -c`

# uptime

查看系统启动时间及负载信息

## 语法

uptime [OPTION]

OPTION：

- -p：显示机器正常运行的时间
- -s：系统启动时间，格式为 uuuu-mm-dd hh:mm:ss

## 案例

显示当前系统运行负载信息 `uptime`

显示系统正常运行时间 `uptime -p`

显示系统启动时间 `uptiome -s`

# free

显示系统内存使用量情况

## 语法

free [参数]

参数：

- -b：以Byte显示内存使用情况
- -k：以kb为单位显示内存使用情况
- -m：以mb为单位显示内存使用情况
- -g：以gb为单位显示内存使用情况
- -s：持续显示内存
- -t：显示内存使用总合
- -h：以易读的单位显示内存使用情况

输出内容介绍

- Mem 行（第二行）是内存的使用情况
- Swap行（第三行）是交换空间的使用情况
- total列：显示系统总的物理内存和交换空间大小
- used列：显示已经被使用的物理内存和交换空间大小
- free列：显示还有多少物理内存和交换空间可以使用
- shared列：显示被共享使用的物理内存大小
- buff/cache列：显示被buffer和cache使用的物理内存大小
- available列：显示还可以被应用程序使用的物理内存大小

## 案例

以默认的容量单位显示内存使用量信息 `free`

以MB为单位显示内存使用量信息 `free -m`

以易读的单位显示内存使用量信息 `free -h`

以易读的单位显示内存使用量信息，每个3秒刷新一次 `free -hs 3`

# ulimit

是shell的内嵌命令，控制shell程序的资源，通过对资源的控制，达到系统调优的的用作

## 语法

ulimit [参数]

参数：

- -a：显示目前资源限制的设定
- -c<core文件上限>：设定core文件的最大值，单位为区块
- -d<数据节区大小>：程序数据节区的最大值，单位为KB
- -f<文件大小>：shell所能建立的最大文件，单位为区块
- -H：设定资源的硬性限制，也就是管理员所设下的限制
- -m<内存大小>：指定可使用的内存的上线，单位为KB
- -n<文件数目>：指定同一时间最多可开启的文件数
- -p<缓冲区大小>：指定管道缓冲区的大小，单位512字节
- -s<堆叠大小>：制定堆叠的上线，单位为KB
- -S：设定资源的弹性限制
- -t<CPU时间>：指定CPU使用时间的上线，单位为秒
- -u<程序数目>：用户最多可开启的程序数目
- -v<虚拟内存大小>：指定可使用的虚拟内存上线，单位为KB

## 案例

显示系统资源的设置 `ulimit -a`

设置单一用户程序数目上线 `ulimit -u 500`

将每个进程可以打开的文件数目加大到4096 `ulimit -n 4096`

指定可使用的虚拟内存上线为12800KB `ulimit -v 12800`

指定CPU使用时间的上限位2s `ulimit -t 2`

# init

切换系统运行级别

## 案例

关机 `init 0`

单用户模式 `init 1`

多用户，没有NFS不联网 `init 2`

切换到多用户-命令行模式 `init 3`

没有用到 `init 4`

图形化界面模式 `init 5`

重新启动 `init 6`

# service

控制系统服务

## 语法

service [参数]

参数：

- --status-all ：显示所有服务的状态
- start：启动服务
- stop：停止服务
- restart：重新启动服务
- status：查看服务运行状态
- reload：重新载入服务配置

## 案例

查看系统中所有服务现在的运行状态 `service --status-all`

查看sshd运行状态`service sshd status`

启动sshd服务 `service sshd start`

停止sshd服务 `service sshd stop`

重启sshd服务 `service sshd restart`

# vmstat

显示虚拟内存状态

## 语法

vmstat [选项] [时间间隔] [次数]

参数：

- -a：显示活跃和非活跃内存
- -f：显示从系统启动至今的fork数量
- -m：显示slab信息
- -n：只在开始时显示一次各字段名称
- -s：显示内存相关统计信息及多种系统活动数量
- delay：刷新时间间隔。如果不指定，只显示一条结果
- count：刷新次数。如果不指定刷新次数，但指定了刷新时间间隔，这时刷新次数为无穷
- -d：显示磁盘相关统计信息
- -p：显示指定磁盘分区统计信息
- -S：使用指定单位显示。参数有k、K、m、M，分别代表1000、1024、1000000、1048576字节(byte)。默认单位为K(1024bytes)

## 案例

显示虚拟内存的使用情况`vmstat`

指定状态信息刷新的时间间隔为1秒 `vmstat 1`

显示活跃和非活跃内存统计信息 `vmstat -a`

显示从系统启动以来的fork数量 `vmstat -f`

显示内存使用的详细信息 `vmstat -s`

显示磁盘相关统计信息(磁盘的读/写情况) `vmstat -d`

查看/dev/sda1磁盘分区的读/写情况 `vmstat -p /dev/sba1`

显示系统的slabinfo `sudo vmstat -m`

# iostat

监视系统输入输出设备和CPU的使用情况

## 语法

iostat [参数] [设备]

参数：

- -c：仅显示CPU使用情况
- -d：仅显示设备利用率
- -k：显示状态以千字节每秒为单位，而不使用块每秒
- -m：显示状态以兆字节每秒为单位
- -p：仅显示块设备和所有被使用的其他分区的状态
- -t：显示每个报告产生时的时间
- -x：显示扩展状态

## 案例

显示所有设备负载情况 `iostat`

每隔2秒显示一次，总共显示3次 `iostat 2 3`

显示指定磁盘信息`iostat -d sda1`

显示tty和cpu信息 `iostat -t`

以M为单位显示所有信息 `iostat -m`

查看TPS和吞吐量信息 `iostat -d -k 11`

查看磁盘I/O的详细情况 `iostat -x /dev/sda1`

查看设备使用率(%utiol)、响应时间(await) `iostat -d -x -k 11`

查看cpu状态 `iostat -c 1 3`

# ipcs

显示进程间通讯设备的信息

## 语法

ipcs [参数]

参数：

- -a：默认的输出信息
- -m：打印出使用共享内存进行进程间通信的信息
- -q：打印出使用消息队列进行进程间通信的信息
- -s：打印出使用信号进行进程间通信的信息

## 案例

显示所有的IPC `ipcs  ipcs -a`

输出信息的详细变化时间 `ipcs -t`

输出ipc方式的进程ID `ipcs -p`

输出ipc方式的创建者/拥有者 `ipcs -c`

输出当前系统下ipc各种方式的状态信息 `ipcs -u`

查看各个资源的系统限制信息 `ipcs -l`

# ipcrm

删除一个或更多的消息队列、信号量集或者共享内存标识

## 语法

ipcrm [options]

ipcrm <shm | msg | sem> \<id> [...]

参数：

- -m，--shmem-id \<id>：按id号移除共享内存段
- -M，--shmem-id \<id>：按键值移除共享内存段
- -q，--queue-id \<id>：按id号移除消息队列
- -Q，--queue-key <键>：按键值移除消息队列
- -s，--semaphore-id \<id>：按id号移除信号量
- -S，--semaphore-key <键>：按键值移除信号量
- -a，--all[=<shm | msg | sem>] 全部移除
- -v，--verbose：解释正式进行的操作

## 案例

通过id删除共享内存 `ipcrm -m 262133`

通过key删除共享内存 `ipcrm -M 0x55`

通过id删除消息队列 `iprm -q 252432`

通过key删除消息队列 `iprm -Q 0x88`

通过id删除信号量 `iprm -S 0x65`

删除所有共享内存、信号量和消息队列`iprm -a`

删除所有共享内存、信号量和消息队列，并且显示过程 `iprm -v -a`

# route

显示并设置路由

## 语法

route [-f] [-p] [Command [Destination] [mask Netmask] [Gateway] [metric Metric]] [if Interface]

## 参数

- -n：不要使用通讯协定或主机名称，直接使用IP 或 port number
- -net：标识后边接的路由为一个网域
- -host：标识后面接的为连接到单部主机的路由
- netmask：与网域有关，可以设定 netmask 决定网域的大小
- gw：gateway的简写，后续接的是IP的数值，与dev不同
- dev：如果只是要指定由那一块网路卡连线出去，则使用这个设定，后面接 eth0 等

flag：

	- U：up表示此路由当前为启动状态
	- H：Host，表示此网关为一主机
	- G：Gateway，表示此网关为一路由器
	- R：Reinstate Route，使用动态路由重新初始化的路由
	- D：Dynamically，此路由是动态性写入
	- M：Modified，此路由是由路由守护程序或导向器动态修改
	- !：表示此路由当前为关闭状态

## 案例

显示当前路由 `route  route -n`

添加网关/设置网关 `route add -net 224.0.0.0 netmask 240.0.0.0 dev ens33`

屏蔽一条路由 `route add -net 224.-.-.- netmask 240.0.0.0 reject`

删除路由记录 `route del -net 224.0.0.0 netmask 240.0.0.0`  `route del -net 224.0.0. netmask 240.0.0.0 reject`

# ping

测试主机间网络连通性

## 语法

ping [参数] 目标主机

参数：

- -c：指定发送报文的次数
- -i：指定收发信息的间隔时间
- -s：设置数据包的大小
- -t：设置存活数值TTL的大小 
  - Linux系统的TTL的值为64或255
  - Windows NT/2000/XP系统的TTL值为128
  - Windows 98系统的TTL值为32
  - Unix主机的TTL值为255

## 案例

测试与 www.baidu.com 网站的连通性 `ping www.baidu.com`

连续ping 4次 `ping -c 4 www.baidu.com`

连续ping 4次，间隔3秒 `ping -c 4 -i 3 www.baidu.com`

测试局域网连通性 `ping 192.168.107.128`

设置数据包为1024字节，TTL为255 `ping -s 1024 -t 255 www.baidu.com`

# traceroute

追踪数据包在网络上的传输时的全部路径

## 语法

traceroute [参数] [域名或者IP]

参数：

- -m <存活数值>：设置检测数据包的最大存活数值TTL的大小
- -n：直接使用IP地址而非主机名换
- -p<通信端口>：设置UDP传输协议的通信端口
- -q：探测包个数设置
- -r：忽略普通的Routing Table，直接将数据包送到远端主机上
- -w：设置等待远端主机回报的时间

## 案例

追踪本地数据包到百度的传输路径 `traceroute www.baidu.com`

跳数设置 `traceroute -m 7 www.baidu.com`

显示IP地址，不查主机名 `traceroute -n www.baidu.com`

把探测包的个数设置为4 `traceroute -q 4 www.baidu.com`

把对外发探测包的等待响应时间设置为3秒 `traceroute -w 3 www.baidu.com`

探测包使用的基本UDP端口设置为6888 `traceroute -p 6888 www.baidu.com`

绕过正常的路由表，直接发送到网络相连的主机  `traceroute -r www.baidu.com`

# netstat

显示网络状态

## 语法

netstat [参数]

参数：

- -a：显示所有连线中的socket
- -p：显示正在使用套接字的程序识别码和程序名称
- -l：仅列出在监听的服务状态
- -t：显示TCP传输协议的连线状况
- -u：显示UDP传输协议的连线状态
- -i：显示网络界面信息表单
- -r：显示路由表信息
- -n：直接使用IP地址，不通过域名服务器

## 案例

显示系统网络状态中的所有连接信息 `netstat -a`

显示系统网络状态中的TCP连接信息 `netstat -at`

显示系统网络状态中的UDP连接信息 `netstat -au`

输出中显示PID和进程名称 `netstat -p`

只显示监听端口 `netstat -l`

只列出所有监听TCP端口 `netstat -lt`

只列出所有监听UDP端口 `netstat -lu`

列出所有监听 UNIX 端口 `netstat -lx`

显示所有端口的统计信息 `netstat -s`

显示TCP或UDP端口的统计信息 `netstat -st  netstat -su`

显示系统网络状态中的UDP连接端口号使用信息 `netstat -apu`

显示网卡当前状态信息 `netstat -i`

显示网络路由表状态信息 `netstat -r`

找到某个服务所对应的连接信息 `netstat -ap | grep ssh`

## ss

显示活动套接字信息

## 语法

ss [参数]

## 案例

显示TCP套接字 `ss -at`

显示UDP套接字 `ss -au`

显示套接字使用概况 `ss -s`

列出所有打开的网络连接端口 `ss -l`

查看进程使用的socket `ss -pl`

找出打开套接字/端口应用程序 `ss -lp | grep 6010`

查看主机监听的端口 `ss -tnl`

解析IP和端口号 `ss -tlr`

# telnet

远程登入服务器

## 语法

telnet [参数] [主机] [端口]

## 案例

登录远程主机 `telnet 192.168.107.133`

指定端口 `telnet 192.168.107.133 6379`

# ssh

远程连接工具

## 语法

ssh [参数] [主机]

参数：

- -l <登录名>：指定连接远程服务器的登录用户名
- -p <端口>：指定远程服务器上的端口

配置文件 `/etc/ssh/sshd_config`

## 案例

登录远程服务器 `ssh 192.168.0.10`

以xx身份远程等于服务器 `ssh -l xxx 192.168.0.10   ssh xxx@192.168.0.10`

指定端口及用户名登录服务器 `ssh -p 2222 xxx@192.168.0.10`

远程执行命令 `ssh 192.168.0.10 date`

# ftp

文件传输协议客户端

## 语法

ftp [参数] [主机名或IP]

## 常用ftp命令

- ‌**open**‌：与FTP服务器相连接。
- ‌**send (put)**‌：上传文件。
- ‌**get**‌：下载文件。
- ‌**mget**‌：下载多个文件。
- ‌**cd**‌：切换目录。
- ‌**mkdir**‌：在服务器上创建目录。
- ‌**rmdir**‌：删除远程目录。
- ‌**delete**‌：删除文件。
- ‌**quit**‌：结束与服务器的FTP会话并退出。
- ‌**help**‌：显示帮助信息。
- ‌**pwd**‌：显示当前目录的路径。
- ‌**lpwd**‌：列出本地当前目录。
- ‌**status**‌：请求服务器返回当前目录的状态信息‌。

## 增加ftp写权限

第一步： `sudo gedit /etc/vsftpd.conf`

第二步：`去除#write_enable=YES前的#`

第三步：`sudo service vsftpd restart`

## 案例

安装ftp `sudo apt install vsftpd`

建立FTP连接 `ftp 192.168.107.133`

下载一个文件 `get file.txt`

下载多个文件 `mget file1.txt file2.txt`

上传一个文件 `put newfile.txt`

上传多个文件 `mput newfile1.txt newfile2.txt`

# sftp

交互式的文件传输程序

## 语法

sftp [参数] [IP或主机名]

## 案例

使用SFTP进行客户端连接 `stfp xxx@192.168.107.120`

指定端口连接 `sftp -P 30 xxx@192.168.107.120`

查看sftp支持的命令 `help   ?`

从远程服务器下载文件到本地 `get file.txt`

从远程服务器下载目录到本地 `get -r dir`

从本地上传文件到远程服务器 `put newfile.txt`

从本地上传目录到远程服务器 `put -r newdir`

执行本地Shell命令(在命令前加叹号) `!command   !echo hello world`

退出 `bye  exit`

# lftp

文件客户端程序

## 语法

lftp [参数]

## 案例

登录远程服务器 `lftp xxx@192.168.107.133`

查看lftp支持的命令 `help   ?`

从远程服务器下载文件到本地 `get file.txt   mget file*.txt   mget -c *.txt`

从远程服务器下载目录到本地 `mirror dir`

从本地上传文件到远程服务器 `put newfile.txt   mput newfile*.txt`

从本地上传目录到远程服务器 `mirror -R newdir`

退出 `bye   exit`

# wget

文件下载

## 语法

wget [OPTION] [URL]

OPTION：

- -i：下载指定文件里列出的地址
- -O：下载后重命名文件
- -c：打开断点续传
- -b：启动后转入后台执行
- -P：指定保存路径

## 案例

使用wget下载单个文件 `wget xxxurl`

使用wget下载多个文件 `vim rullist.txt   wget -i urllist.txt`

下载后以不同的文件名保存 `wget -O xx.zip xxx.com/x.zip`

下载后保存到指定目录 `wget -P dir xxxurl`

wget限速下载 `wget --limit-rate=300k xxxurl`

打开断点续传功能 `wget -c xxxurl`

使用wget后台下载 `wget -b xxxurl` 查看下载进度 `tail -f wget-log`

# scp

远程拷贝文件

## 语法

scp [-346BCpqrTV] [-c cipher] [-F ssh_config] [-i identity_file] [-J destination] [-l limit] [-o ssh_option] [-P port] [-S program] source ... target

参数：

- -r：递归复制整个目录
- -p：保留原文件的修改时间，访问时间和模式
- -P：指定数据传输用到的端口号

## 案例

从远程复制文件到本地 `scp xxx@192.168.107.128:/home/xxx/file.txt ~`

从远程复制目录到本地 `scp -r xxx@192.168.107.128:/home/xxx/dir/ ~`

上传本地文件到远程机器指定目录 `scp ~/newfile.txt xxx@192.168.107.128:/home/xxx/`

上传本地目录到远程机器指定目录 `scp -r ~/newdir xxx@192.168.107.128:/home/xxx/`

使用指定的端口号传输数据 `scp -P 2222 xxx@192.168.107.128:/home/xxx/file.txt ~`

保留文件的最后修改时间及访问时间 `scp -p xxx@192.168.107.128:/home/xxx/file.txt ~`

# curl

文件传输工具

## 语法

curl [参数] 网址

参数：

- -o：指定新的本地文件名
- -O：保留远程文件的原始名
- -u：通过服务端配置的用户名和密码授权访问
- -l：打印HTTP响应头信息
- -u：指定登录账户密码信息
- -A：设置用户代理标头信息
- -b：设置用户cookie信息
- -C：支持断点续传
- -s：静默模式，不输出任何信息
- -T：上传文件

## 案例
获取指定网站的网页源码 `curl www.baidu.com`

保存网页 `curl -o baidu.html www.baidu.com`

下载指定网站中的文件 `curl -O xxx.com/xx.zip`

下载文件并重命名 `curl -o newname.zip xxx.com/xx.zip`

断点续传 `curl -C --O xxx.com/xx.zip`

打印指定网站的HTTP响应头信息 `curl -l xxx.com/xx.zip`

通过ftp下载指定文件服务器中的文件 `curl -u xxx:pwsswd ftp://baidu.com/file.txt`

上传文件 `curl -T newfile.txt -u 用户名:密码 ftp://www.baidu.com/dir/`

# host

域名查询

## 语法

host [参数] [域名]

参数：

- -a：显示详细的DNS信息
- -v：显示指令执行的详细信息

## 案例

查询域名对应的ip地址 `host www.baidu.com`

显示执行域名查询的详细信息 `host -v www.baidu.com`

显示详细的DNS信息 `host -a www.baidu.com`

# tcpdump

监听网络流量，需要管理员权限

## 案例

监视第一个网络接口上所有流过的数据包 `tcpdump`

监视指定网络接口的数据包 `tcpdump -i ens33`

显示指定数据包 `tcp -c 20`

精简模式显示10个包 `tcpdump -c 10 -q`

监视指定主机的数据包（主机名）`tcpdump host www.baidu.com`

监听指定主机的数据包（ip地址）`tcpdump -i any port 22 -A`

# useradd

创建并设置用户信息

## 语法

useradd [参数] 用户名

参数：

- -d<登入目录>：指定用户登入时的目录
- -g<群组>：初始群组
- -G<群组>：非初始群组
- -m：自动创建用户的家目录
- -M：不要创建用户的家目录
- -N：不要创建以用户名称为名的群组
- -s：指定用户登入后所使用的shell

## 四个重要配置文件

- /etc/passwd
- /etc/shadow
- /etc/group
- /ect/gshadow

## 案例

直接创建新用户 `useradd user1`

床用创建方法 `useradd -m -s /bin/bash user2`

自动创建家目录 `useradd -m username`

指定家目录 `useradd -m -d /new/dir username`

指定用户ID `useradd -u 1500 username`

指定组ID `useradd -g group username`

分配多个组 `useradd -g group -G group1,group2 username`

指定登录shell `useradd -s /bin/bash username`

自定义注释 `useradd -c "Test User Account" username`

# passwd

修改用户的密码

## 语法

passwd [参数] 用户名

参数：

- -d：删除已有密码
- -l：锁定用户的密码值，不允许修改
- -U：解锁用户的密码值，允许修改
- -e：下次登录强制修改密码
- -k：用户在期满后仍能使用
- -S：查询密码状态

## 案例

修改当前登录用户密码 `passwd`

修改指定用户的密码值 `passwd xxx`

锁定指定用户的密码值，不允许其进行修改 `passwd -l xxx`

解锁指定用户的密码值，允许其进行修改 `passwd -u xxx`

强制指定的用户在下次登录时必须重置其密码 `passwd -e xxx`

删除指定用户的密码值 `passwd -d xxx`

查看指定用户的密码状态 `passwd -S xxx`

# userdel

删除用户账户

## 语法

userdel [参数] 用户名

参数：

- -r：删除用户主目录及其中的任何文件

## 案例

删除指定的用户账户信息 `userdel xxx`

删除指定的用户账户信息及其家目录 `userdel -r xxx`

# su

切换用户身份

## 语法

su [参数] 用户名

参数：

- -：完全身份变更
- -c：执行完指定的指令后，即恢复原来的身份
- -f：适用于csh与tsch，使shell不用去读取启动文件
- -l：改变身份时，也同事变更工作目录
- -m：变更身份时，不要变更环境变量
- -s：指定要执行的shell

## 案例

切换超级用户 `su`   `su root`

变更账号为xxx并执行whomai指令后退出变回原使用者 `su -c whoami xxx`

变更账号为xxx并保留在当前工作目录 `su xxx`

变更账号为xxx并改变工作目录至xxx的家目录 `su - xxx`

# sudo

以系统管理员的身份执行指令

## 语法

sudo [参数]

参数：

- -l：显示当前用户的权限
- -u<用户>：以指定的用户作为新的身份。若不加上此参数，则预设以root作为新的身份

## 配置文件

/etc/sudoers  授权用户/组 主机=[(切换到哪些用户或组)] 命令

查看配置文件  `sudo visudo` `sudo vim /etc/sudoers`

## 案例

列出当前用户的权限 `sudo -l`

指定用户身份执行命令 `sudo -u xxx whoami`

以root权限执行上一条命令 `sudo !!`

## id

显示用户ID和组ID

## 语法

id [参数] [用户名]

参数：

- -g：显示用户所属群组的ID
- -G：显示用户所属附加群组的ID
- -n：显示用户，所属群组或附加群组的名称
- -r：显示实际ID
- -u：显示用户ID

## 案例

显示当前用户的所有信息 `id`

显示指定用户信息 `id xxx`

显示用户所属群组的ID `id -g xxx`

显示用户所属附加群组的ID `id -G xxx`

## usermod

修改用户账号信息

## 语法

usermod [参数] 用户名

参数：

- -c<备注>：修改用户账号的备注文字
- -d<登入目录>：修改用户登入时的家目录
- -e<有效期>：修改账号的有效期限
- -f<缓冲天数>：修改在密码过期后多少天即关闭该账号
- -g<群组>：修改用户所属的群组
- -G<群组>：修改用户所属的附加群组
- -l<账号名称>：修改用户账号名称
- -L：锁定用户密码，使密码无效
- -s\<shell>：修改用户登入后所使用的shell
- -u\<uid>：修改用户ID
- -U：解除密码锁定

## 案例

修改指定用户的家目录路径 `usermode -d /home/hometest xxx`

修改指定用户的UID号码 `usermode -u 1234 xxx`

修改指定用户的名称 `usermode -l xxx aaa`

锁定指定用户的账户 `usermode -L xxx`

解锁指定用户的账户 `usermode -U xxx`

# group

显示一个用户所加入的所有用户组

## 语法

groups用户

## 案例

显示xxx用户所加入的所有组 `groups xxx`

# groupadd

创建新的用户组

## 语法

groupadd [参数] 用户组

参数：

- -g：指定新建工作组的id
- -r：创建系统工作组

## 案例

创建一个新的用户组 `groupadd work`

创建一个新的用户组，并指定GID号码 `groupadd -g 1234 family`

创建一个新的用户组，设定为系统工作组 `groupadd -r grouptest`

# groupdel

删除用户组

## 语法

groupdel [参数] [群组名称]

## 案例

删除指定用户组 `groupdel family`

# whoami

打印当前登录用户

## 语法

whoami

## 案例

查询当前登录的用户名 `whoami`

# who

查看当前登录用户信息

## 语法

who [参数]

参数

- -a：全面信息
- -b：系统最近启动时间
- -l：系统登录进程
- -H：带有列标题打印用户名，终端和时间
- -t：系统上次锁定时间
- -u：已登录用户列表
- -q：列出所有已登录的用户的名称和数量

## 案例

查看当前登录用户信息 `who`

查看当前登录用户信息，并加上标题栏 `who -H`

查看当前全部的登录用户信息 `who -H -a`

查看系统的最近启动时间 `who -b`

显示终端属性 `who -T -H`

精简模式显示 `who -q`

# w

显示已登录用户

## 语法

w [参数]

参数：

- -h：不打印头信息
- -u：当显示当前进程和CPU时间时忽略用户名
- -s：使用短输出格式
- -f：显示用户从哪登录
- -o：老式输出
- -i：显示IP地址而不是主机名

## 案例

显示目前登入系统的用户信息 `w`

不打印头信息 `w -h`

显示用户从哪登录 `w -f`

使用短输出格式 `w -s`

# last

显示用户或终端的登录情况

## 语法

last [选项]

参数：

- -R：省略hostname的栏位
- -a：把从何处登入系统的主机名或IP地址，显示在最后一行
- -n <显示行数> 或 -<显示行数>：显示名单的行数
- -i：显示ip地址而不是主机名

## 案例

显示近期用户或终端的登录情况 `last`

简略显示，并指定显示的个数 `last -n 5 -R` `last -5 -R`

最后一列显示主机IP地址 `last -n 5 -a -i` 

# top

实时显示进程动态

## 语法

top -hv|-bcEHiOSs1 -d secs -n max -u|U user -p pid -o fld -w [cols]

参数：

- -d：指定每两次屏幕信息刷新之间的时间间隔（单位为秒）
- -c：切换显示命令名称和完整命令行
- -p：通过指定监控进程ID来仅仅监控某个进程的状态
- -n：信息更新最大次数

## 快捷键

- c：显示进程绝对路径
- P：根据CPU使用率排序
- M：根据物理内存使用率排序
- 1：显示每个核的CPU状态

## 案例

实时显示进程动态 `top`

显示完整的进程信息 `top -c`

指定信息刷新时间为5秒 `top -d 5`

仅监控进程 1877 的状态 `top -p 1877`

设置信息更新次数 `top -n 2`

# ps

显示进程状态

## 语法

ps [OPTION]

OPTION：

- -A：显示所有进程
- -a：显示所有终端机下执行的程序
- -x：通常与a这个参数一起使用，可列出比较完整的信息
- -e：列出程序时，显示每个程序所使用的环境变量
- -f：用ASCII字符显示树状结构，表达程序间的相互关系
- -u<用户识别码>：列出属于该用户的程序的状况，也可以使用用户名称来指定

## 信息说明

- USER：用户名称
- PID：进程号
- %CPU：该进程所占用CPU百分比
- %MEM：该进程所占用内存百分比
- VSZ：进程所占用的虚拟内存大小
- RSS：进程所占用的实际内存大小
- TTY：该进程运行在哪个终端上面
- STATUS：进程状态
  - R：运行
  - S：可中断睡眠
  - D：不可中断睡眠
  - T：停止
  - Z：僵死
- START：进程启动时间
- TIME：进程实际占用CPU的时间
- COMMAND：该进程对应的执行程序

## 案例

显示当前系统所有进程状态：

	- 列出目前所有的正在内存中的程序 `ps -aux`
	- 显示所有进程信息 `ps -A`
	- 显示所有进程信息，连同命令行 `ps -ef`

树形显示所有进程 `ps -axf`

查找特定进程信息 `ps -aux | grep ssh`

显示指定用户信息 `ps -u xxx`

配合less命令使用 `ps -aux | less`

结合管道操作符与sort命令，依据处理器使用量(第三列)情况降序排序 `ps aux | sort -rnk 3`

# pstree

以树状图显示进程

## 语法

pstree [参数]

参数：

- -a：显示每个程序的完整指令，包含路径，参数或是常驻服务的标示
- -c：不使用精简标示法
- -G：使用VT100终端机的列绘图字符
- -h：列出树状图时，特别标明现在执行的程序

## 案例

以树状图显示进程 `pstree`

显示该进程的完整指令及参数，遇到相同的进程名可以压缩显示 `pstree -a`

显示当前所有进程的进程号和进程id `pstree -p`

同事显示用户名称 `pstree -u`

# pgrep 

检索当前正在运行的进程

## 语法

pgrep [参数] [模式]

参数：

- -d：设置一个字符串，用于分隔输出的每个进程ID
- -f：模式参数仅用于匹配进程名
- -l：列出进程名及进程ID
- -u：选择仅匹配指定有效用户ID进程

## 案例

查找ssh进程的pid `pgrep sshd`

指定分隔输出 `pgrep sshd -d ''`

查询用户xxx启动的bash进程的PID `pgrep -u xxx sshd`

显示进程名称和PID `pgrep -l sshd`

使用正则表达式 `pgrep '^sshd$' -l`

匹配所有的参数列表 `pgrep -f ssh`

# lsof

查看进程打开的文件

## 语法

lsof [参数] [文件]

参数：

- -a：列出打开文件存在的进程
- -c<进程名>：列出指定进程所打开的文件
- -g：列出GID号进程详情
- -d<文件号>：列出占用该文件号的进程
- +d<目录>：列出目录下被打开的文件
- +D<目录>：递归列出目录下被打开的文件
- -n<目录>：列出使用NFS的文件
- -i<条件>：列出符合条件的进程
- -p<进程号>：列出指定进程号所打开的文件
- -u：列出UID号进程详情

## 案例

查看当前系统中全部文件与进程对应信息 `lsof`

显示指定目录中目录中被调用的文件信息 `lsof +d /home`

递归显示指定目录中全部被调用的文件信息 `losf +D /home`

查看谁正在使用某个文件 `losf /bin/bash`

列出某个用户打开的文件信息 `lsof -u xxx`

列出某个程序进程所打开的文件信息 `lsof -c bash`

通过某个进程号显示该进程打开的文件 `lsof -p 1930`

根据文件描述符列出对应的文件信息 `lsof -d 1`

# jobs/bg/fg

终端任务调度

## 案例

列出当前shell的任务 `jobs -l`

将test2.sh 调至前台运行 `fg 2`

将当前进程切至后台运行 `ctrl + z`

回复test2.sh在后台运行 `bg 2`

杀死test3.sh `kill 3` `kill pid`

# kill

发送信号到进程

## 语法

kill [OPTION] \<pid> [...]

-l 列出系统支持的信号

## 常用信号

- HUO 1 终端断线
- INT 2 中断(同 ctrl+c)
- QUIT 3 退出(同ctrl+\)
- TERM 15 终止
- KILL 9 强制终止
- COUNT 18 继续(与STOP相反，fg/bg命令)
- STOP 19 暂停 (同 ctrl+z)

## 案例

列出系统支持的所有信号列表 `kill -l`

不指定信号 `kill 1951`

得到指点信号的数量 `kill -l SIGKILL`

杀死指定进程 `kill -9 1951`

杀死指定用户所有进程 `kill -9 $(ps -ef | grep xxx)`

# killall

使用进程名称来杀死进程

## 语法

killall [参数] [进程名称]

参数：

- -l：打印所有已知信号列表
- -u：杀死指定用户的进程

## 案例

杀死所有sleep进程 `killall sleep`

查看killall支持的所有信号 `killall -l`

发送指定信号 `killall -9 sleep`

杀死指定用户所有进程 `sudo killall -u xxx`

# nice/renice

调整进程的优先级

## 语法

nice [参数] [命令]

renice [参数] [命令]

参数

nice

	- -n：后面接一个数值，范围在 -10~19

renice

	- -g：指定进程组id
	- -p：改变该程序的优先权等级，此参数为预设值
	- -u：指定开启进程的用户名

## 案例

查看nice值 `ps -l`

设置优先级为15 `nice -n 15 vim &` `nice -15 vim &`

根据pid重新设置进程的nice值 `renice 6 -p 5200`

将xxx的进程的nice值全部设置为-5 `renice -5 -u xxx`

# nohup

后台运行程序

## 案例

让进程在后台运行 `nohup ./test &`

输出重定向到file.txt文件  `nphup ./test > file.txt 2>&1 &`

# apt

包管理器

## 语法

apt [选项] 软件包

## 案例

列出所有可更新的软件清单 `sudo apt update`

升级软件可更新的软件包 `sudo apt upgrade`

单独升级某个安装包 `sudo apt upgrade vim`

安装软件包 `sudo apt install bat`

删除软件包 `sudo apt remove bat`

打印软件包信息 `sudo apt show bat`

清理不再使用的依赖和库文件 `sudo apt autoremove`

列出已安装的所有软件包 `sudo apt list --installed`

# export

设置或显示环境变量

## 语法

export [选项] 文件

参数：

- -f：代表[变量名称]中为函数名
- -n：删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中
- -p：列出所有的shell赋予程序的环境变量

## 注意

export的作用效果仅限于本次登录

如果想永久生效，可修改配置文件 `/etc/profile #所有用户` `~/.bashrc  #当前用户`

## 案例

列出当前所有的环境变量 `export -p`

定义环境变量 `export MYENV`

定义环境变量并复制 `export MYENV=7`

修改环境变量PATH `export PATH=$PATH:/usr/local/mysql/bin`

# source

在当前shell环境中从指定文件读取和执行命令

该命令通常用命令 "." 来代替

## 语法

source [文件]

## 案例

读取和执行bashrc文件 `source ~/.bashrc` `. ~/.bashrc`

执行刚修改的初始化文件，使之立即生效 `source /etc/profile`

# set/unset

设置/删除shell变量

## 语法

set [参数] [变量]

unset [参数] [变量]

参数：

set：

	- -a：标示已修改的变量，以供输出至环境变量

unset：

	- -f：仅删除函数
	- -v：仅删除变量

## 案例

显示所有的shell变量 `set`

设置环境变量 `第一步：定义新的环境变量 declare myenv='666'` `第二步：将定义的变量输出为环境变量 set -a myenv` `第三步：查看是否设置成功 env|grep myenv`

删除环境变量 `unset -v myenv` `unset myenv#优先删除变量`

# history

显示与管理历史命令记录

## 语法

history [参数]

参数：

- -c：清楚命令记录
- -d：删除指定序号的命令记录
- -n：读取命令记录

## 案例

显示执行过的全部命令记录 `histroy`

显示执行过的最近5条命令 `history 5`

重新执行2039命令 `!2039`

重新执行上一条命令 `!!`

将搜索与你输入相匹配的最近一个命令，并重新执行 `!ls`

调用历史记录的递归搜索 `ctrl+r`

如果想要删除特定命令 `hostory -d 1234`

清空全部历史记录 `hostiry -c`

 
