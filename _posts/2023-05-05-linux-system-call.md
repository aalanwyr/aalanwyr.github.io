---
layout: post
title:  如何在 Linux Kernel (V5.17.7) 中添加一个系统调用（System call）
categories: [Linux]
description:  如何在 Linux Kernel (V5.17.7) 中添加一个系统调用（System call）
keywords: Linux Kernel
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
--- 
最近在学习 《linux Kernel Development》，本书用的 linux kernel 是 v2.6 版本的。看完 ”系统调用“ 一节后，想尝试添加一个系统调用，然后重编一个 kernel 。经过几个小时的尝试，实现了这个小功能，其中也遇到了不少坑，本文主要是记录分享下如何在 Linux Kernel (V5.17.7) 中添加一个系统调用（System call）。
**编 kernel之前需要注意 ：**
1. 修改的 kernel 是目前最新的 release 版本(V5.17.7), 书中v2.6版本的 kernel 太老了，gcc 需要降到4.8版本，否则无法编过。 kernel 发布地址：https://www.kernel.org/
2. 需要选用大内存，多核的机器编 kernel，否则会出现各种异常问题，而且编kernel 很费时间。15GB 内存的机器，编不过 kernel 。换用100GB 内存的机器就好了

本文主要包含以下几点内容：
1. 环境准备
2. 修改 kernel
3. rebuild kernel 以及安装 kernel
4. 测试结果

# 1 环境准备
我编kernel的机器是：Ubuntu 20.04.1 LTS，内存180GB,  cores: 88
## 1.1 更新系统的源
`sudo apt update && sudo apt upgrade -y`
## 1.2 安装编译 kernel 需要的依赖
`sudo apt install build-essential libncurses-dev libssl-dev libelf-dev bison flex -y`

我这里用的vim，没有的话也需要安装：
`sudo apt install vim -y`
## 1.3 清除已经安装的 packages
`sudo apt clean && sudo apt autoremove -y`
## 1.4 下载 kernel code
```
wget -P ~/ https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.17.7.tar.xz
tar -xvf ~/linux-5.17.7.tar.xz -C ~/
```
# 2 修改 kernel
## 2.1 检查你自己当前系统的 kernel 版本
`uname -r`
> 5.11.0-36-generic

重新安装 kernel 之后，这个版本号会被修改。
## 2.2 切换到工作目录中，然后创建自己的系统调用
```
cd ~/linux-5.17.7/
mkdir hello
```
## 2.3 创建自己的系统调用
`vim hello/hello.c`
 **添加代码**
```
#include <linux/kernel.h>
#include <linux/syscalls.h>

SYSCALL_DEFINE0(hello)
{
    printk("hello_system_call.\n");
    return 0;
}
```
## 2.4 为你的系统调用创建 Makefile
`vim hello/Makefile`
 添加下面内容：
 `obj-y := hello.o`
## 2.5 将你的系统调用添加到 Kernel 的 makefile 中
`vim Makefile`
搜索 core-y， 完成如下添加：
![image.png](https://github.com/aalanwyr/aalanwyr.github.io/blob/main/images/posts/linux/system_call/2642361-20220515162411935-1108056021.png?raw=true)

## 2.6 将系统调用的相应函数原型添加到系统调用的头文件中
`vim include/linux/syscalls.h`
添加：
`asmlinkage long sys_hello(void);`
![image.png](https://github.com/aalanwyr/aalanwyr.github.io/blob/main/images/posts/linux/system_call/2642361-20220515162600233-1130732972.png?raw=true)

## 2.7 在 system_table 中为你的系统调用开辟一个新的系统调用号
`vim arch/x86/entry/syscalls/syscall_64.tbl`
![image.png](https://github.com/aalanwyr/aalanwyr.github.io/blob/main/images/posts/linux/system_call/2642361-20220515162819442-2019006145.png?raw=true)

# 3 编译 kernel 并安装
前面的步骤都很简单，这一步可能会出现各种问题，而且很耗时。
## 3.1 创建 .config
一路默认设置就好
`make menuconfig`
## 3.2 查询你的机器 logicl cores
`nproc`
## 3.3 编译安装 kernel
```
make -j32
echo $?  // make 结束之后记得检查一下 状态
###if output is 0 then
sudo make modules_install -j32
echo $? 
sudo make install -j32
```
## 3.4 查看 kernel 是否安装成功
```
sudo update-grub
sudo reboot
```
![image.png](https://github.com/aalanwyr/aalanwyr.github.io/blob/main/images/posts/linux/system_call/2642361-20220515163432013-548984031.png?raw=true)

# 4 测试结果
## 4.1 编一个code 调用你的系统调用
由于系统调用不像普通函数那样，需要通过sys_call 以及系统调用号才能实现系统调用。创建一个test.c
```
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#define __NR_hello 451

long hello_syscall(void)
{
    return syscall(__NR_hello);
}
int main(int argc, char *argv[])
{
    long activity;
    activity = hello_syscall();
    if(activity < 0)
    {
        perror("Sorry, xxx. Your system call appears to have failed.");
    }
    else
    {
        printf("Congratulations, xxx! Your system call is functional. Run the command dmesg in the terminal and find out!\n");
    }
    return 0;
}
```
## 4.2 测试结果
```
gcc -o test test.c
./test
dmesg // 后面也能看到系统调用打印的信息
```
> Congratulations, xxx! Your system call is functional. Run the command dmesg in the terminal and find out!