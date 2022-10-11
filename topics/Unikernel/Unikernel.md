### 其他相关资料
>[(YdrMaster)幺内核技术](https://github.com/YdrMaster/notebook/blob/main/topics/unikraft/20220926-unikernel.md)
>[(YdrMaster 论文翻译)Unikraft：构造快速、专用的幺内核的简单方法](https://github.com/YdrMaster/notebook/blob/main/tranlation/20220923-unikraft.md)
>[[论文分享] Unikraft: fast, specialized unikernels the easy way](https://mstmoonshine.github.io/p/unikraft/)

### 什么是幺内核
> Unikraft 是一个高度模块化的 unikernel 构建系统。

>在远古时代，计算机只是用来辅助计算的工具，计算机上跑的是一些特定的计算任务，给定一些输入，得到一些输出，这个过程非常朴素。操作系统的诞生来自于共享硬件的需求。当几个人希望同时使用一台硬件设备、或者一个人希望同时运行多个计算任务的时候，就需要一个更高权限的管理员来统筹调度，按照一定的规则来给每一个 task 分配资源。
>
>如果一台设备上只是运行少数几个 tasks，那么只需要一个简单的管理员进行 brute-force 的资源分配就可以了，这甚至可以通过人工管理员手动来完成。然而随着硬件能力的迅速发展，一台设备上只运行少数几个 tasks 变得非常不划算，对 tasks 数量 scalability 的需求急剧增长，这时候就不能再使用简单的方式进行资源分配了，对操作系统的需求应运而生。操作系统中的 kernel 就是资源的管理员。
>
>而在操作系统的运行和产生过程中不可避免的部分就是对于抽象的理解` The art of operating system is the art of virtualization `
>
>同时，随着硬件水平的逐渐上升，我们可以将更多的资源赋予给一个功能强大的计算机，在这个计算机上面运行各种`hypervisor`，使得一个计算机上能够跑多个操作系统
>
>[欠缺的知识点](https://mstmoonshine.github.io/p/unikraft/)
>
>一个 unikernel 包含一个 application 以及运行这个 application 所需要的自上而下的所有环境，包括标准库、协议栈、设备驱动等等。舍弃了不需要的部分使得 unikernel 的 image size 更小，运行速度更快。比如一个基于 UDP 协议的服务器就不需要一套 Linux 中完整的网络协议栈。
>
>为了达到高性能，需要为每个不同的 application 做定制，但是这些定制中又经常包含重复的部分。
>
Unikernel 经常不兼容 POSIX，使得 application 本身也需要进行大量的移植工作。


#### 核心思路
-   Kernel 应该完全模块化，这样 unikernel 才会高度可定制。比如传统 OS 可以分成 memory allocator, scheduler, network stack, filesystem, device drivers 和 early boot code 等。
-   每个模块提供的 API 应该经过精心设计，从而达到很高的 performance。并且同一类模块要实现同样的一套 API，以达到可以随意替换的目标。

#### 结构（design decision）
-   Single Address Space
-   Fully Modular System
-   Single Protection Level（没有 User mode / Kernel mode 的划分）
-   Static Linking
-   POSIX Support
-   Platform Abstraction

![unikernel image](https://mstmoonshine.github.io/p/unikraft/arch_hu7f0d69b317e270f9760328c44fe46977_106314_1024x0_resize_box_3.png)

Unikraft 遵循 _Everything is a micro-library_ 的原则。图中每个黑色的方块，代表的是一套 API，除此之外每个小方块代表一个 library，处于同一个白色方框中的方块可以互相取代。自上而下比较重要的组件有：

-   经过若干个小 patch 移植的 libc，比如 musl 和 newlib，
-   syscall-shim：将 syscall 变成 function call，实现 kernel bypass 提升性能，
-   其他提供 posix 标准的 library，
-   网络协议栈，比如 lwip、mtcp，
-   文件系统，比如 9pfs，ramfs，
-   调度器，
-   Booter，
-   内存分配器，比如 buddy allocator，
-   不同 hypervisor 平台需要的 driver，

其中只有 Booter 是每个 application 都必须使用的，其余的模块都是可选的。

