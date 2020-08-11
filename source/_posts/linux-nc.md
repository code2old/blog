---
title: Linux-nc命令详解
data: 2020-08-10 20:00:00
updated: 2020-08-10 20:00:00
tags:
  - Linux
categories: 
  - Linux命令详解
---

### Linux命令详解-netcat

netcat是网络工具中的瑞士军刀，它能读写网络中的TCP和UDP数据，并且可以通过重定向与其它工具结合使用，从而发挥出令人惊讶的功能。以下是常用nc命令的示例

#### 端口扫描

netcat可以作为端口扫描工具来发现机器上开放的端口，帮助系统管理员或者黑客发现系统上存在的漏洞

```shell
$nc -z -v -n 192.168.63.35 21-25
```
这个命令会打印21-25所有开放的端口信息，默认为TCP模式（-u可调整为UDP模式）。
-z 表示0 IO，即连接成功后马上关闭连接，不进行数据交换
-v 表示详细输出
-n 表示netcat不使用DNS反向查询IP地址的域名

#### 文件传输

Linux中有很多工具可以进行文件的网络传输，比如：FTP、Samba和scp等工具，但是ftp、samba都需要安装配置，而在一些嵌入式设备中，根本不支持scp命令。在这种情况下，可以使用netcat命令作为传输文件的桥梁进行跨机器的文件传输。

```shell
# A--->B
$ nc -l 8000 < file.txt            # 机器A，server端
$ nc -n 192.168.63.35 > file.txt   # 机器B，client端
```

以上命令在A机器上创建了一个文件服务器，并且将文件file.txt重定向到netcat，当客户端连接到该端口时，netcat会向客户端发送file.txt的内容。而客户端B会将接收到的文件内容重定向file.txt。

也可以在A机器上创建文件服务器，等待客户端的连接和发送数据，并将netcat结果重定向到文件中。

```shell
# A<---B
$ nc -l 8000 > file.txt             # 机器A，server端
$ nc 192.168.63.35 8000 < file.txt  # 机器B，client端
```

#### 目录传输

发送整个目录或者多个文件时，和发送单个文件的方式原理差不多，只不过需要使用压缩工具tar工具，然后再进行重定向即可。

```shell
# A--->B
$ tar -cvf - dirname | nc -l 8000           # 机器A，server端
$ nc -n 192.168.63.35 8000 | tar -xvf -     # 机器B，client端
```

在A服务器上创建了一个tar的归档包并将它重定向到netcat服务器，客户端连接到服务器时，服务器便将归档数据发送给客户端，客户端netcat收到数据后再重定向到tar工具。

如果想要节省带宽，可以通过压缩工具压缩后再进行发送

```shell
$ tar -cvf - dirname | bzip2 -z | nc -l 8000         # 机器A，server端
$ nc -n 192.168.63.35 8000 | bzip2 -d | tar -xvf -   # 机器B，client端
```

使用tar归档和bzip2压缩和解压

#### 网络加密传输

在发送数据之前，可以使用mcrypt工具保证数据安全

```shell
$ nc 192.168.63.35 8000 | mcrypt --flush --bare -F -q -d -m ecb > file.txt    # 客户端
$ mcrypt --flush --bare -F -q -m ecb < file.txt | nc -l 8000   #服务端
```

以上两个命令都需要输入密码，而且两端使用相同的密码。这里使用mcrypt工具进行加解密，使用其他加解密工具也可以的。

（加密工具的用法测试的时候失败了，没有错误的打印，但是客户端输出的文件里面是空的，有知道原因的小伙伴可以交流一下）

#### 视频流

个人觉得用netcat发送视频流不是一个好的方法，linux上用于收发视频流的工具很多，这里只是介绍netcat发送视频流的用法

```shell
$ cat video.mp4 | nc -l 8000
$ nc 192.168.63.35 8000 | mplayer -vo x11 -cache 3000 -
```

以上命令netcat从socket中读取数据并重定向到mplayer中

#### 克隆一个设备

如果已经安装配置一台Linux机器并且需要对其他的机器进行同样的操作，只需要启动另一台机器的一些引导就可以克隆机器。假如系统在磁盘/dev/sda上，克隆Linux PC的操作如下

```shell
$ dd if=/dev/sda | nc -l 8000            # server端
$ nc -n 192.168.63.35 | dd of=/dev/sda   # client端
```

dd是一个从磁盘读取原始数据的工具，通过netcat服务器重定向它的输出流到其他机器并且写入到磁盘中，它会随着分区表拷贝所有的信息。但是如果已经做过分区并且只需要克隆root分区，我们可以根据我们系统root分区的位置，更改sda 为sda1，sda2.等等。

#### 打开一个shell

如果没有安装telnet或ssh，可以使用netcat创建远程shell

```sh
$ nc -v -c /bin/bash -lp 8000
$ nc -v 192.168.63.35 8000
```

#### nc常用参数选项

nc -h查看

```

[v1.10-41.1]
connect to somewhere:   nc [-options] hostname port[s] [ports] ... 
listen for inbound:     nc -l -p port [-options] [hostname] [port]
options:
        -c shell commands       as `-e'; use /bin/sh to exec [dangerous!!]
        -e filename             program to exec after connect [dangerous!!]
        -b                      allow broadcasts
        -g gateway              source-routing hop point[s], up to 8
        -G num                  source-routing pointer: 4, 8, 12, ...
        -h                      this cruft
        -i secs                 delay interval for lines sent, ports scanned
        -k                      set keepalive option on socket
        -l                      listen mode, for inbound connects
        -n                      numeric-only IP addresses, no DNS
        -o file                 hex dump of traffic
        -p port                 local port number
        -r                      randomize local and remote ports
        -q secs                 quit after EOF on stdin and delay of secs
        -s addr                 local source address
        -T tos                  set Type Of Service
        -t                      answer TELNET negotiation
        -u                      UDP mode
        -v                      verbose [use twice to be more verbose]
        -w secs                 timeout for connects and final net reads
        -C                      Send CRLF as line-ending
        -z                      zero-I/O mode [used for scanning]
port numbers can be individual or ranges: lo-hi [inclusive];
hyphens in port names must be backslash escaped (e.g. 'ftp\-data').

```
