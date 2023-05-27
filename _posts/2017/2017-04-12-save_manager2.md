---
layout: post
title: 操作系统之存储器管理（二）
date: 2017-04-12
catalog: true
tags:
    - 基础
---
上一篇中连续分配方式会形成许多“碎片”，虽然可通过重定向动态分配将许多碎片拼接成大块空间，但须付出很大的开销，为了解决上面的问题页入了离散分配方式，他包括分页存储管理方式（纯分页存储管理它不具有实现虚拟存储的功能，它要求把每个作业全部装入内存后方能运行）和分段存储管理方式，这篇还会总结虚拟存储器的基本概念和常见的页面置换算法。下面一步一步的来<!--more-->
### 基本分页存储管理
1. 页面和物理块：我们在分页存储中把进程根据<font color=red>逻辑地址空间</font>分成大小相等的片，称为页面或页.每个页都有相应的页号相应的把内存空间分成与页面一样大小的若干存储块，称为（物理）快或页框，这些物理块可以不相邻，解决了连续分配的弊端，当然最后一页经常会出现装不满的情况叫“页内碎片”。
2. 页面大小： 页面大小通常为2的幂，通常为512B~8KB,页面太大，页面碎片就大了，内存利用率就不高了，页面太小，页面之间的换出，换入就多了，速度相应就下来了，同时页表的占比相应的就高了，说起页表我们下一条说页表
3. 页表是我们为了快速找到每个进程的离散物理块，我们建立了一张映像表简称页表，进程通过查找页表，就可以找到物理块号了，页表的作用就是实现从页号到物理快号的映射，面试中经常会问到<font color>分页存储中怎么对物理块进行保护其实可以在页表中设置一个表项字段，规定其是可读还是可写等等</font>
4. 地址结构：每个页面都有分页地址分页地址结构如下![](/images/2017/0412fenye.png)后面一部分是页内地址也叫位移量（有待考证是每页的大小，这里为4k）地址空间最大为1M
5. 地址变换机构：这个就是将逻辑地址转换成物理地址，是根据页表来完成的上面说了页表的作用，当然这里分两种情况<font color=red>a.</font>基本的地址转换机构：页表的功能可以由一组寄存器来实现，这样可以提高地址转换速度，但是现实往往很残酷，页表很长，寄存器又很贵故页表是保存在内存中的，我们在系统中只设置了一个页表寄存器PTR,在其中存放页表的始址和页表的长度，但是每个进程分配一个寄存器还是不合理所以当进程需要逻辑地址中的数据时才会把那个地址换成页号和页内地址，当页号大于或等于页表长度时表示本次访问越界了，否则就得到该表项在页中的位置（页表始址与页号和页表项长度的乘积相加（页表项长度是什么））经过这样就完成了逻辑地址到物理地址的转换<font color=red>b.</font>具有块表的地址变换机构：由于页表存在内存中所以每取一次数据要进行两次内存访问，第一次页表，第二次是数据，效率有待改进，为了提高地址变换速度引入了一个具有并行查旬能力的特殊缓冲寄存器，称为“联想寄存器”或“块表” 这个感觉和缓冲差不多，把那些感觉要用到的块号放在预先放在块表里，每次进程要调用逻辑地址时先在块表里查号，如果有就直接那去用，如果块表满了，换出老的且感觉不用的页表项就好了
6. 两级和多级页表： 由于现在计算机的支持逻辑地址非常大有的是64位，在这样的情况下页表就会变的很大（页表项占一个字节），页表必须是连续的，如果占的太大要找一块连续地址不容易，所以我们要解决这个问题，引入了两级页表的问题 ，我们将页表分页，存在不同的逻辑块中，然后在建立新页表来找这些分开的页表<font color=red>此时的二级逻辑地址结构</font>变成了外层页号，外层页内地址，页内地址，这三个了。为了提高转换率同样设置了外层页表寄存器用于存放外层页表的始址。这种分级页表的方法没有减少内存的占有，采取的策略是把外层页表放入内存部分页表也放入，同时给外层页表设置一个状态位S,看哪个在内存里，其它全放入磁盘中。这样就可以更好的减少内存的使用

### 基本分段存储管理方式
引入分段存储管理不是为了提高内存利用率，他是根据作业逻辑关系分成若干段（并不是逻辑地址），每个段从0开始并有自己的名字和长度，因此希望访问的逻辑地址是由段名和段内偏移量决定的。他方便了编程，分页是根据地址化分的，而分段是根据逻辑单位化分的有利于信息的共享。动态链接也要求以段为单位进行，解决了动态增长的问题，
1. 分段： 要据逻辑分成了Main段，子程序段X,等，他们大小肯定不等为了进行管理也给他们弄了个段号，段内地址等等 ，也同样有段表，和地址变换机构和分页存储差不多，

### 虚拟存储器的基本概念
虚拟存储器的引入为了解决下面的情况，前面的几种管理方式都有一个问题就是只有作业全部装入内存才可以运行，假如有的作业非常大一次性放不到内存里怎么办（前面几种存储方式有一次性，还有一个驻留性也就是当进程全部运行完才会撤离，有些进程里的东西其实用完了）。为了解决上面的问题伟大的人类根据局部性原理（时间局部性和空间局部性）引入了虚拟存储器，也就是将一个作业的一部分先引入内存中（请求调页功能），如果内存满了用置换功能将内存中暂时不用的页调到盘上，从用户角度讲这个内存好像变大了一直可以往进装，其实是虚的所以叫虚拟存储器
1. 虚拟存储器的实现方法
虚拟存储器不适合连续分派，因为虚拟存储器就是允许一个作业分多次调入内存，那么就要提前分出足够的空间了，这样就造成了浪费，所以我们采用离散方式了，离散方式又分为下面两种情况
**分页请求系统**： （注意名字）在分页系统的基础上增加了请求调页功能和页面置换功能，主要硬件有请求分页的页表机制，缺页中断机制，地址变换机构
**分段请求系统**：（注意名字）是在分段系统的基础上加了请求调段功能及分段置换功能后形成的虚拟存储系统，主要硬件和上面的类似且一样
**段页式虚拟存储器** 在段页式的基础上增加请求调页和页面置换功能形成的，这里不说了，一样的
2. 虚拟存储器的特性
a. 多次性，一个作业分多次调入 b.对换性，包括换入和换出 c.虚拟性

### 请求分页存储管理方式
1. 请求分页中的硬件支持
a. 页表机制(和原来的对比学习),页表机制主要说的还是那个数据结构了，这里添加了几个页表项物理号。。。。。。。
b. 缺页中断机构
c. 地址变换机构： 原来是地址变换机构，这次成了交换
2. 内存分配策略和分配算法
a. 最小物理块数的确定
b. 物理块的分配策略
c. 物理块分配算法
3. 调页策略

### 请求分段存储管理方式
分段的共享与保护

### 页面置换算法
1. 最佳置换算法
2. 先进先出置换算法
3. 最近最久未使用置换算法
4. Clock置换算法