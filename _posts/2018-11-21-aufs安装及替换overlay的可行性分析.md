---
title: aufs安装及替换overlay的可行性分析
date: 2018-11-23 12:17:13
tags: CSDN迁移
---
 版权声明：本文为博主原创文章，未经博主允许不得转载。 https://blog.csdn.net/qq_37788558/article/details/84382545   
 ### []()概述：

 [https://blog.csdn.net/qq_37788558/article/details/84346641](https://blog.csdn.net/qq_37788558/article/details/84346641)  
 在上面但文章里，使用联合文件系统的一种实现：**overlayfs** 遇到问题，尝试换用另一种实现：**aufs**  
 (具体是什么问题可以看上面的文章)

 
### []()aufs介绍

 aufs是一种实现了联合挂载（union mount）的文件系统，同unionfs类似，它能够将不同类型的文件系统透明地层叠在一起，实现一个高效的分层文件系统。说白了aufs就是能将不同的目录挂载到某一目录下，并将各个源目录下的内容联合到目标目录下，这里每个源目录对应aufs中的一层，用户在目标目录读写时，感觉不到此目录是联合而来的。aufs中的每一层都可以有不同的权限(只读，读写)，这个特性使得它在很多开源项目中有应用，比如Knoppix，Live CD, Docker等等。

 
### []()实践过程

 检测系统是否支持aufs

 
```
# grep aufs /proc/filesystems
# 

```
 发现不支持

 
> aufs默认没有合入内核，据说被拒绝多次，此生无望。
> 
>  
 升级内核

 
```
cd /etc/yum.repo.d
# 下载文件
wget https://yum.spaceduck.org/kernel-ml-aufs/kernel-ml-aufs.repo
# 安装
yum install kernel-ml-aufs
#编辑 grub.conf 文件，修改 Grub 引导顺序
 vim /etc/grub.conf
#重启
reboot
#再次查看
uname -r
3.10.5-3.el6.x86_64

grep aufs /proc/filesystems 
nodev   aufs

#成功

```
 aufs配置

 
```
mkdir a b uniondir
mount -t aufs -o br=/mnt/test/a:/mnt/test/b none /mnt/test/uniondir

```
 运行正常。

 
### []()总结

 同时结合对overlay的使用的比较，发现aufs相当于只设置了多个叠加的下层，相当于overlay的lowdir=./a:./b,那么可能overlay也可以只设置多个叠加下层，不设置upperidr、workdir两项。

 
> mount -t overlay overlay -olowerdir=/mnt/mybucket:/mnt/source  
>  /mnt/merged
> 
>  
 **overlay中，upper可写，lower只读，都在lower层，则不可写，所以无法使用（没尝试都在upper层）**

 aufs中，多层会写最上层

   
  