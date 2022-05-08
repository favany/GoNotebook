# 一文搞定 Go 并发的实现原理：goroutine

### 一、Goroutine调度器

我们知道，一切的软件都是跑在操作系统上，真正用来计算的是 CPU。早期的操作系统每个程序就是一个进程，直到一个程序运行完，才能进行下一个进程，就是 “单进程时代”。一切的程序只能串行发生。

早期的单进程操作系统，面临两个问题：单一的执行流程，计算机只能一个任务一个任务处理。进程阻塞所带来的 CPU 时间浪费。那么能不能有多个进程来宏观一起来执行多个任务呢？后来操作系统就具有了最早的并发能力：多进程并发，当一个进程阻塞的时候，切换到另外等待执行的进程，这样就能尽量把 CPU 利用起来，CPU 就不浪费了。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53961de42b7c4a34a9a24bdae64dab05\~tplv-k3u1fbpfcp-watermark.image?)

在多进程/多线程的操作系统中，就解决了阻塞的问题，因为一个进程阻塞 cpu 可以立刻切换到其他进程中去执行，而且调度 cpu 的算法可以保证在运行的进程都可以被分配到 cpu 的运行时间片。这样从宏观来看，似乎多个进程是在同时被运行。

但新的问题就又出现了，进程拥有太多的资源，进程的创建、切换、销毁，都会占用很长的时间，CPU 虽然利用起来了，但如果进程过多，CPU 有很大的一部分都被用来进行进程调度了。怎么才能提高 CPU 的利用率呢？但是对于 Linux 操作系统来讲，cpu 对进程的态度和线程的态度是一样的。

很明显，CPU 调度切换的是进程和线程。尽管线程看起来很美好，但实际上多线程开发设计会变得更加复杂，要考虑很多同步竞争等问题，如锁、竞争冲突等。多进程、多线程已经提高了系统的并发能力，但是在当今互联网高并发场景下，为每个任务都创建一个线程是不现实的，因为会消耗大量的内存 (进程虚拟内存会占用 4GB, 而线程也要大约 4MB)。

大量的进程/线程出现了新的问题：`高内存占用`和`调度的高消耗CPU`。好了，然后工程师们就发现，其实一个线程分为`内核态线程`和`用户态线程`。一个`用户态线程`必须要绑定一个`内核态线程`，但是 CPU 并不知道有 “用户态线程” 的存在，它只知道它运行的是一个 “内核态线程”(Linux 的 PCB 进程控制块)。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9bf641c349c49d9890ab3acc8fdb4f4\~tplv-k3u1fbpfcp-watermark.image?)

这样，我们再去细化去分类一下，内核线程依然叫 “线程 (thread)”，用户线程叫 “协程 (co-routine)”.

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d61579539c9d4639a66f6304516685e7\~tplv-k3u1fbpfcp-watermark.image?)

那么线程和协程该如何绑定呢？

* 1个协程绑定 1 个线程，这种最容易实现。协程的调度都由 CPU 完成了，缺点是 协程的创建、删除和切换的代价都由 CPU 完成，对CPU消耗很大，略显昂贵。
* N个协程绑定1个线程，优点就是协程在用户态线程即完成切换，不会陷入到内核态，这种切换非常的轻量快速。但也有很大的缺点，某个程序用不了硬件的多核加速能力；一旦某协程阻塞，造成线程阻塞，本进程的其他协程都无法执行了，根本就没有并发的能力了。
* 因此，Go选择了 M 个协程绑定 N 个线程，是 N:1 和 1:1 类型的结合，克服了以上 2 种模型的缺点。协程跟线程是有区别的，线程由 CPU 调度是抢占式的，**协程由用户态调度是协作式的**，一个协程让出 CPU 后，才执行下一个协程。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f828b9f8ccb349fabe84f6c93fbb5b11\~tplv-k3u1fbpfcp-watermark.image?)

Go采用了**用户层轻量级thread**或者说是**类co-routine**的概念来解决并发问题，Go将之称为`goroutine`。goroutine占用的资源非常小，非常轻量，只占几 KB，并且这几 KB 就足够 goroutine 运行完，这就能在有限的内存空间内支持大量 goroutine，支持了更多的并发。goroutine调度的切换也不用深入操作系统内核层完成，代价很低。因此，一个Go程序中可以创建成千上万个并发的goroutine。所有的Go代码都在goroutine中执行，哪怕是go 的 runtime（运行函数）也不例外。将这些 `goroutine` 按照一定算法放到“CPU”上执行的程序就称为**goroutine调度器** (goroutine scheduler) 。

Go 使用 goroutine 和 channel，提供了更容易使用的并发方法。goroutine 来自协程的概念，让一组可复用的函数运行在一组线程之上，即使有协程阻塞，该线程的其他协程也可以被 runtime 调度，转移到其他可运行的线程上。最关键的是，程序员看不到这些底层的细节，这就降低了编程的难度，提供了更容易的并发。

一个Go程序对于操作系统来说只是一个**用户层程序**，对于操作系统而言，它的眼中只有thread，它甚至不知道有什么叫Goroutine的东西的存在。goroutine的调度全要靠Go自己完成，实现Go程序内goroutine之间公平的竞争CPU资源，这个任务就落到了Go runtime头上，要知道在一个Go程序中，除了用户代码，剩下的就是go runtime了。

于是Goroutine的调度问题就演变为go runtime如何将程序内的众多goroutine按照一定算法调度到CPU资源上运行了。在操作系统层面，Thread竞争的“CPU”资源是真实的物理CPU，但在Go程序层面，各个Goroutine要竞争的CPU资源是操作系统线程。这样Go scheduler的任务就明确了：将goroutines按照一定算法放到不同的操作系统线程中去执行。这种在语言层面自带调度器的，我们称之为**语言原生支持并发**。

### 二、GPM模型

Dmitry Vyukov亲自操刀改进Go scheduler，在Go 1.1中实现了**GPM调度模型**，这个模型一直沿用至今：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c0a31c54938434aab7e80c21e2e90a2\~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3f53efd43f847a2880819cbfc34ae4f\~tplv-k3u1fbpfcp-watermark.image?)

P是一个“逻辑Proccessor”，每个G要想真正运行起来，首先需要被分配一个P。对于G来说，P就是运行它的“CPU”，可以说：G的眼里只有P 。但从Go scheduler视角来看，真正的“CPU”是M，只有将P和M绑定才能让P的runq中G得以真实运行起来。这样的P与M的关系，就好比Linux操作系统调度层面用户线程(user thread)与核心线程(kernel thread)的对应关系那样(N x M)。

* G: **表示goroutine，存储了goroutine的执行stack信息、goroutine状态、goroutine的任务函数以及与所在P的绑定等信息。G对象可以重用。**
* P: **表示逻辑processor，P的数量决定了系统内最大可并行的G的数量（前提：系统的物理cpu核数>=P的数量）。P 还有一个很重要的作用是他拥有的各种 G 对象队列、链表、一些cache和状态。** P管理着一组goroutine队列，P里面会存储当前goroutine运行的上下文环境（函数指针，堆栈地址及地址边界），P会对自己管理的goroutine队列做一些调度（比如把占用CPU时间较长的goroutine暂停、运行后续的goroutine等等）当自己的队列消费完了就去全局队列里取，如果全局队列里也消费完了会去其他P的队列里抢任务。
* M: **M表示machine，代表着真正的执行计算资源。M是Go运行时（runtime）对操作系统内核线程的虚拟， M与内核线程一般是一一映射的关系， 一个goroutine最终要放到M上执行的。**\* 在绑定有效的p后，进入schedule循环；而schedule循环的机制大致是从各种队列、p的本地队列中获取G，切换到G的执行栈上并执行G的函数，调用goexit做清理工作并回到m，如此反复。M并不保留G状态，这是G可以跨M调度的基础。

P与M一般也是一一对应的。他们关系是： P管理着一组G挂载在M上运行。当一个G长久阻塞在一个M上时，runtime会新建一个M，阻塞G所在的P会把其他的G 挂载在新建的M上。当旧的G阻塞完成或者认为其已经死掉时 回收旧的M。

P的个数是通过`runtime.GOMAXPROCS`设定（最大256），Go1.5版本之后默认为物理线程数。在并发量大的时候会增加一些P和M，但不会太多，切换太频繁的话得不偿失。

单从线程调度讲，Go语言相比起其他语言的优势在于OS线程是由OS内核来调度的，`goroutine`则是由Go运行时（runtime）自己的调度器调度的，这个调度器使用一个称为m:n调度的技术（复用/调度m个goroutine到n个OS线程）。 其一大特点是goroutine的调度是在用户态下完成的， 不涉及内核态与用户态之间的频繁切换，包括内存的分配与释放，都是在用户态维护着一块大的内存池， 不直接调用系统的malloc函数（除非内存池需要改变），成本比调度OS线程低很多。 另一方面充分利用了多核的硬件资源，近似的把若干goroutine均分在物理线程上， 再加上本身goroutine的超轻量，以上种种保证了go调度方面的性能。

Go语言中的操作系统线程和goroutine的关系：

1. 一个操作系统线程对应用户态多个goroutine。
2. go程序可以同时使用多个操作系统线程。
3. goroutine和OS线程是多对多的关系，即m:n。

#### 参考文献：

> Aceld [Golang 调度器 GMP 原理与调度全分析](https://learnku.com/articles/41728)\
> Tony Bai [也谈goroutine调度器](https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/)\
> 李文周 [Go语言基础之并发](https://www.liwenzhou.com/posts/Go/14\_concurrence/)\
>
