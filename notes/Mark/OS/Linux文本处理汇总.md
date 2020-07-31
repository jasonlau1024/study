


---
title: Linux文本处理总结
date: 2019-11-07 23:00:38
tags:
  - Shell
categories:
  - 'Shell Bash'

---

Linux下文本处理命令简单总结。

<!-- more -->

# 文件查看（cat/tac/rev）

## cat 命令

- **作用：**

  文本输出命令，用于查看文件内容。

- **格式：**

  cat [options]... [files]...

  - -E ：显示行结束符$。
  - -n ：对显示结果的每一行编号。
  - -s ：压缩连续的空行成一行。

- **功能：**

  - 用于查看完整的文件内容。

    `cat /etc/passwd`

  - 结合重定向">"，创建新文件。

    `cat > file1`

  - 合并多个文件。

    `cat file1 file2 > file12`

## tac 命令

- **作用：**

用法同cat，以行为单位反序输出。

## rev 命令

- **作用：**

每行内容以字符为单位反序输出。

# 分页查看（more/less）

## more 命令

- **作用：**

分页查看文件内容。

左下角会显示当前显示距离文末的百分比，通过空格键可以每次向下显示一屏内容，回车键每次向下翻一行。按q键可以退出。

## less 命令

一页一页地查看文件或STDIN输出。

使用 “/ 文本” 可以搜索文件中符合要求的文本并且，使用n/N 可以跳到下一个 或 上一个匹配。

# 前后查看（head/tail）

## head 命令

- **作用：**

  显示文件开头的内容，默认显示文件头10行。

- **参数：**

  - -c# ：指定获取前#个字节。
  - -n# ：指定获取前#行。

## tail 命令

- **作用：**

  显示文件的结尾的内容，默认显示文件末尾10行。

- **参数：**

  -c / -n : 用法同head。

  -f ：跟踪显示文件fd新追加的内容,常用日志监控。

# 按列操作（cut/paste）

## cut 命令

- **作用：**

  显示行中的指定部分，删除文件中指定字段。

- **参数：**

  -d# ：指定分隔符#，默认tab。

  -f# ：指定字段。

  - #：第#个字段
  - #,#[,#] ：离散的多个字段，如：-f1,3,6
  - #-# ：连续的多个字段，如：-f1-6

- **实例：**

  - 截取显示/etc/passwd文件中用户名和对应的shell类型。

    ```shell
    $ cut -d: -f1,7 /etc/passwd
    root:/bin/bash
    bin:/sbin/nologin
    daemon:/sbin/nologin
    ```

## paste 命令

- **作用：**

将多个文件按照队列进行合并。

# 文本分析（wc/sort/uniq）

## wc 命令

- **作用：**

  计数单词总数、行总数、字节总数和字符总数。

- 参数：

  -l ：只计数行总数。

  -w ：只计数单词总数。

  -c ：只计数字节总数。

  -m ：只计数字符总数。

  -L ：显示文件中最长行的长度。

## sort 命令

- **作用：**

  行排序并把整理过的文本显示在StdOut。

- **参数：**

  -n ：按数字大小排序，从小到大。

  -r ：反方向排序。

  -R ：随机排序。

  -f ：忽略字符串中的字符大小写。

  -u ：删除输出中的重复行。

## uniq 命令

- **作用：**

  从输出中删除前后相接的重复行。

- **参数：**

  -c ：显示每行重复出现的次数。

  -d ：仅显示重复过的行。

  -u ：仅显示不曾重复的行。

- **注：**连续且完全相同方为重复，常结合sort使用。

  `sort files.txt | uniq -c`

# 三剑客（grep/sed/awk）

## 正则表达式

### 基本正则表达式和扩展正则表达式

| 字符匹配   | 基本元字符  | 扩展元字符  | 作用                                                         |
| ---------- | ----------- | ----------- | ------------------------------------------------------------ |
|            | .           | .           | 匹配任意单个字符                                             |
|            | [ ]         | [ ]         | 匹配指定范围内的任意单个字符 [:digit:] [:upper:] [:lower:] [:alpha:] |
|            | [^]         | [^]         | 匹配指定范围外的任意单个字符 [:alnum:] [:punct:] [:space:] ... |
| 匹配字数   |             |             | 用在指定的字符后表示其出现次数，默认为贪婪模式               |
|            | *           | *           | 匹配前面字符任意次：0、1、n次                                |
|            | .*          | .*          | 匹配任意长度任意字符                                         |
|            | \?          | \?          | 匹配其前面字符0次或者1次                                     |
|            | \+          | \+          | 匹配其前面字符至少1次                                        |
|            | \{m\}       | {m}         | 匹配其前面字符m次                                            |
|            | \{m,n\}     | {m,n}       | 匹配其前面字符至少m次,至多n次                                |
|            | \{m,\}      | {m,}        | 匹配其前面字符至少m次                                        |
| 位置锚定   |             |             |                                                              |
|            | ^           | ^           | 行首锚定                                                     |
|            | $           | $           | 行尾锚定                                                     |
|            | ^pattern$   | ^pattern$   | 用于pattern 来匹配整行 ^$：空白行、^[[:space:]]*$：空行或包含空白字符的行 |
|            | \< ,\b      | \< ,\b      | 词首锚定                                                     |
|            | \>, \b      | \>, \b      | 词尾锚定                                                     |
|            | \<pattern\> | \<pattern\> | 匹配完整单词                                                 |
| 分组及引用 |             | ()          | 括号内的模式匹配到的字符会被记录于 正则表达式引擎的内部变量中 ，后向引用：\1, \2, ... |

### POSIX 字符类

| 表示      | 说明                                   |
| --------- | -------------------------------------- |
| [:upper:] | 表示大写字母 [A-Z]                     |
| [:lower:] | 表示小写字母 [a-z]                     |
| [:alpha:] | 表示大小写字母 [a-zA-Z]                |
| [:digit:] | 表示阿拉伯数字 [0-9]                   |
| [:alnum:] | 表示大小写字母及阿拉伯数字 [0-9a-zA-Z] |
| [:space:] | 表示空格或Tab键                        |

## grep 命令

- **作用：**

  过滤来自一个文件或标准输入匹配模式内容。

- **格式：**

  Usage: grep [Options]... Pattern [FILE]...

- **参数：**

  - 正则支持：

    -E ，--extended-regexp 支持扩展正则表达式。

    -P ，--perl-regexp 支持Perl正则表达式。

    -e ，--regexp=Pattern 指定多个模式匹配。

    -f ，--file=File 从文件每一行获取匹配模式。

    -i ，--ignore-case 忽略大小写。

    -w ，--word-regexp 模式匹配整个单词。

    -x ，--line-regexp 模式匹配整行。

    -v ，--invert-match 打印不匹配的行。

  - 输出控制：

    -m ，--max-count=Num 输出匹配的结果 Num 数。

    -H ，--with-filename 打印每个匹配的文件名。

    -r ，--recursive 递归目录。

    -c ，--count 只打印每个文件匹配的行数。

    --include=FilePattern 只检索匹配的文件。

    --exclude=FilePattern 跳过匹配的文件。

  - 行控制：

    -B ，--before-context=Num 打印匹配的前几行。

    -A ，--after-context=Num 打印匹配的后几行。

    -C ，--context=Num 打印匹配的前后几行

- **实例：**

  1. 输出 b 文件中在 a 文件相同的行。

     `grep -f a b`

  2. 输出 b 文件中在 a 文件不同的行。

     `grep -v -f a b`

  3. 去除文件空行或#开头的行。

     `grep -E -v "^$|^#" /etc/httpd/conf/httpd.conf`

  4. 输出匹配的前5个结果。

     `seq 1 20 |grep -m 5 -E '[0-9]{2}'`

  5. 统计匹配多少行。

     `seq 1 20 |grep -c -E '[0-9]{2}'`

  6. 递归搜索 /etc 目录下包含 ip 的 conf 后缀文件。

     `grep -r '192.168.1.1' /etc --include *.conf`

  7. 排除搜索 bak 后缀的文件。

     `grep -r '192.168.1.1' /opt --exclude *.bak `

  8. 匹配所有 IP。

     `ifconfig |grep -E -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"`

  9. 打印匹配结果及前后3行。

     `seq 1 10 |grep 5 -C 3`

**补充：**

Linux支持三种形式的grep命令：

- grep：标准 grep 命令，支持基本正则表达式。
- egrep：扩展 grep 命令，支持基本和扩展正则表达式，等价于 grep -E
- fgrep：快速 grep 命令，不支持正则表达式，按字符串字面意思进行匹配，等价于 grep -F

## sed 命令

- **作用：**

  流编辑器，过滤和替换文本。

- **工作原理：**

  sed 命令将当前处理的行读入模式空间进行处理，处理完把结果输出，并清空模式空间。然后再将下一行读入模式空间进行处理输出，以此类推，直到最后一行。还有一个空间叫保持空间，又称暂存空间，可以暂时存放一些处理的数据，但不能直接输出，只能放到模式空间输出。

- **语法：**

  sed [OPTION]... {script-only-if-no-other-script} [input-file]...

  sed [选项] 'sed 命令(定位+操作)' file

- **选项：**

  | 选项 | 说明                     |
  | ---- | ------------------------ |
  | -n   | 不打印模式空间。         |
  | -e   | 执行脚本、表达式来处理。 |
  | -f   | 执行动作从文件读取执行。 |
  | -i   | 修改原文件。             |
  | -r   | 使用扩展正则表达式。     |

  

- **命令：**

  | 命令 | 说明                     |
  | ---- | ------------------------ |
  | P    | 打印匹配行               |
  | =    | 打印文件行号             |
  | a\   | 在定位行之后追加文本信息 |
  | i\   | 在定位行之前插入文本信息 |
  | c\   | 用新文本替换定位文本     |
  | d    | 删除定位行               |
  | s    | 使用替换模式             |
  | {}   | 在定位行执行的命令组     |

- **地址：**

  | 命令              | 说明                           |
  | ----------------- | ------------------------------ |
  | x                 | x 为指定的行号                 |
  | x,y               | 指定从 x 到 y 行的范围         |
  | /pattern/         | 查询包含模式的行               |
  | /pattern/pattern/ | 查询包含两个模式的行           |
  | /pattern/,x       | 从pattern的匹配行到x行之间的行 |
  | x,/pattern/       | 从x行到pattern匹配行之间的行   |
  | x,y!              | 查询不包括 x 和 y 行的行       |

- **删除//d：**

  `sed '/^#/d;/^$/d' /etc/httpd/conf/httpd.conf`

- **替换s///：**

- **多重编辑-e：**

- **实例：**

  1. 位置调换

     `echo "abc:cde;123:456" |sed -r 's/([^:]+)(;.*:)([^:]+$)/\3\2\1/' ==>abc:456;123:cde `

  2. 注释匹配行及往后n行（匹配5及往后的3行）

     `seq 10 |sed '/5/,+3s/^/#/'`

  3. 注释多个匹配行（匹配3或4的行）

     `seq 5 |sed -r '/^3|^4/s/^/#/'`

  4. 去除开头和结尾的空格或制表符

     `echo " 1 2 3 " |sed 's/^[ \t]*//;s/[ \t]*$//'`

  5. 匹配添加新行（a/i/c）

     `sed '/Strings/i \testline' filename`

  6. 获取总行数

     `seq 10 |sed -n '$='`

## awk 命令

- **作用：**

  awk 是一个处理文本的编程语言工具，能用简短的程序处理标准输入或文件、数据排序、计算以及

  生成报表等等。

- **基本命令语法：**

  awk option 'pattern {action}' file

  pattern：匹配模式。

  action：找到匹配内容时所执行的一系列命令。

- **工作流程：**

  在 awk 中，缺省的情况下将文本文件中的一行视为一个记录，逐行放到内存中处理，而将一行中的

  某一部分作为记录中的一个字段。用 1,2,3...数字的方式顺序的表示行（记录）中的不同字段。用

  $后跟数字，引用对应的字段，以逗号分隔，0 表示整个行。

- **参数：**

  -f ：从文件中读取 awk 程序源文件。

  -F # ：# 为输入字段分隔符。

  -v ：var=value 变量赋值。

- **常用模式：**

  BEGIN{} ：给程序赋予初始状态，先执行得工作。

  END{} ：程序结束之后执行的一些扫尾工作。

  /regular expression/ ：为每个输入记录匹配正则表达式。

  pattern && pattern ：逻辑 and，满足两个模式。

  pattern || pattern ：逻辑 or，满足其中一个模式。

  ! pattern ：逻辑 not，不满足模式

  pattern1, pattern2 ：范围模式，匹配所有模式 1 的记录，直到匹配到模式 2

- **内置变量：**

  | 内置变量 | 说明                                                 |
  | -------- | ---------------------------------------------------- |
  | FS       | 保存或设置分隔符，例如：FS=","                       |
  | $N       | 指定分隔符的第N个字段，例如：$1,$5表示第一列和第五列 |
  | $0       | 当前读入整行的文本内容                               |
  | NF       | 记录当前处理行的字段（列）个数                       |
  | NR       | 记录当前处理行的数量                                 |
  | FNR      | 保存当前处理行在原文本内的行号                       |
  | FILENAME | 当前处理的文本名                                     |
  | ENVIRON  | 调用shell环境变量                                    |

- **实例：**

  1. 基本数据列处理

     ```shell
     ## 获取用户名和 shell
     $ awk -F: '{print $1,"usershell="$7}' /etc/passwd
     root usershell=/bin/bash
     bin usershell=/sbin/nologin
     daemon usershell=/sbin/nologin
     ```

  2. 获取当前剩余内存

     ```shell
     $ free | grep Mem | awk '{print"当前剩余内存:\n",$7}'
     当前剩余内存:
      1548084
     ```

  3. 访问 IP 过滤

     ```shell
     ## /var/log/secure是用于记录访问的信息，可以通过这个日志来看出来是否遭受到恶意攻击
     $ grep "Accepted" /var/log/secure | awk '{print $11}'
     101.95.130.134
     101.95.130.134
     ```

  4. 统计每行有多少列（NF）

     ```shell
     $ awk 'BEGIN{FS=":"}{print NF}' /etc/passwd
     7
     7
     7
     ```

  5. 统计uid小于30的用户有多少和大于30的余户有多少（判断）

     ```shell
     $ awk -F: 'BEGIN{i=0;j=0}{if($3<=30){i++}else{j++}}END{print "<=30:"i,"\n",">=30:"j}' /etc/passwd
     <=30:13
      >=30:11
     ```

  6. 统计指定字段出现的个数（for循环）

     ```shell
     $ awk '{ip[$1]++} END{for(i in ip){print i,ip[i]}}' /var/log/httpd/access_log
     192.168.3.7 2
     ```

  7. 访问日志分析（Nginx为例）

     ```shell
     ## 日志格式：
     '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"'
     ## 统计访问 IP 次数：
     $ awk '{a[$1]++}END{for(v in a)print v,a[v]}' access.log
     ## 统计访问访问大于100次的IP：
     $ awk '{a[$1]++}END{for(v ina){if(a[v]>100)print v,a[v]}}' access.log
     ## 统计访问IP次数并排序取前10：
     $ awk '{a[$1]++}END{for(v in a)print v,a[v]|"sort -k2 -nr |head -10"}' access.log
     ## 统计时间段访问最多的IP：
     $ awk'$4>="[02/Jan/2017:00:02:00" &&$4<="[02/Jan/2017:00:03:00"{a[$1]++}END{for(v in a)print v,a[v]}'access.log
     ## 统计上一分钟访问量：
     $ date=$(date -d '-1 minute'+%d/%d/%Y:%H:%M)
     $ awk -vdate=$date '$4~date{c++}END{printc}' access.log
     ## 统计访问最多的10个页面：
     $ awk '{a[$7]++}END{for(vin a)print v,a[v]|"sort -k1 -nr|head -n10"}' access.log
     ## 统计每个URL数量和返回内容总大小：
     $ awk '{a[$7]++;size[$7]+=$10}END{for(v ina)print a[v],v,size[v]}' access.log
     ## 统计每个IP访问状态码数量：
     $ awk '{a[$1" "$9]++}END{for(v ina)print v,a[v]}' access.log
     ## 统计访问IP是404状态次数：
     $ awk '{if($9~/404/)a[$1" "$9]++}END{for(i in a)print v,a[v]}' access.log
     ```







