---
layout: blog
author: Jiessie.Cao
comments: true
date: 2010-08-04 14:48:04+00:00
slug: None
title: XenMon工作机制
categories:
- Virtualization
- Architecture
tag:
- XenMon
- Virtualization
- xentrace
- xenbaked

---

一、 背景：


XenMon是一个支持资源分配和功能配置部件，它可以精确地监视和展现分析结果的基础部件，它报告在不同VMs资源利用率，而且还提供一个对Xen的共享内部资源接口和资源安排。


以下为XenMon的架构图：
![XenMon Architecture]({{ site.urlimg }}{{ post/xenmon.jpeg }})

<!-- more -->
  Xenbaked 是一个高度可配置的模块，它能让用户配置多久记录一帧，保存多久的历史记录。

 

Xenmon frontend 是作为前段应用，它是基于xentrace事件产生器的数据收集收集到的数据进行展示。

 

Xentrace是事件触发器，其机制在之后会介绍。

 

二、 xentrace机制：

 

Xentrace 是以共享内存的方式来进行操作的，内核部分对Xen中的一些系统调用函数都会对这片共享内存进行写入。在用户区，用户通过一些方法得到共享内存内的数据，并对其进行解析并显示，以下是xentrace工作机制的流程图：

 ![Xen trace]({{ site.urlimg }}{{ post/xentrace.gif }})

三、 t_buf结构：

 

在xentrace的代码中会经常发现t_buf这个结构体：

<pre>
<code>

Struct t_buf{
	unsigned long cons;
	unsigned long prod; 
};
</code>
</pre>
 

这个结构提记录的生产者和消费者的位置指针，而且它还是一个环状结构的共享内存，在代码中不难发现生产者和消费者的地址不是无限增大的，它是有一定大小的限制。

 

此处讨论的t_buf只是单纯的结构体，而环形缓存的结构大小是由t_buf结构体占用的空间和t_buf中消费者和生产者指针指向的地址data区所占用空间的总和。

 

四、 t_rec结构：

 
<pre>
<code>
struct t_rec {
uint32_t event:28; //事件的宏定义，在处理的时候根据宏定义来处理响应的函数
uint32_t extra_u32:3; /* # entries in trailing extra_u32[] array */
uint32_t cycles_included:1; /* u.cycles or u.no_cycles?是否是cycles方式 *
union {
struct {
uint32_t cycles_lo, cycles_hi; /* cycle counter timestamp */
uint32_t extra_u32[7]; /* event data items */
} cycles;

struct {
uint32_t extra_u32[7]; /* event data items */
} nocycles;
} u;      
}
</code>
</pre>


t_rec结构是t_buf映射到用户区的内存空间，同样也是一个环状结构的缓存。从t_rec的结构上，可以得到事件类型等信息，然后在对这些事件进行处理。
