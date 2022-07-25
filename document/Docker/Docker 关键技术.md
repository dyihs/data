[toc]

**2021/8/23**

### 一、Docker

- 基于Linux 内核的Cgroup，Namespace，以及Union FS等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术，由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。
- 最初实现是基于LXC，从0.7 以后开始去除LXC，转而使用自行开发的Libcontainer，从1.11开始，则进一步演进为使用runC和Containerd。
- Docker 在容器的基础上，进行了进一步的封装，从文件系统（Union FS）、网络互联到进程隔离（Namespace）等等，极大的简化了容器的创建和维护，使得Docker 技术比虚拟机技术更为轻便、快捷。

### 二、Namespace

#### （1）、Namespace

Linux Namespace是一种Linux Kernel提供的资源隔离方案：

- 系统可以为进程分配不同的Namespace；
- 并保证不同的Namespace资源独立分配、进程彼此隔离，即不同的Namespace下的进程互不干扰。

#### （2）、隔离性-Linux Namespace

1. Pidnamespace

   * 不同用户的进程就是通过Pidnamespace隔离开的，且不同namespace 中可以有相同Pid。
   * 有了Pidnamespace, 每个namespace中的Pid能够相互隔离。

2. net namespace

   - 网络隔离是通过net namespace实现的，每个net namespace有独立的network devices, IPaddresses, IP routing tables, /proc/net 目录。
   - Docker默认采用veth的方式将container中的虚拟网卡同host上的一个docker bridge（桥接模式）: docker0连接在一起。

3. ipc namespace

   - Container中进程交互还是采用linux常见的进程间交互方法（interprocesscommunication –IPC）, 包括常见的信号量、消息队列和共享内存。
   - container 的进程间交互实际上还是host上具有相同Pidnamespace中的进程间交互，因此需要在IPC资源申请时加入namespace信息-每个IPC资源有一个唯一的32 位ID。

4. mnt namespace

   允许不同namespace的进程看到的文件结构不同，这样每个namespace 中的进程所看到的文件目录就被隔离开了。

5. uts namespace 

   UTS(“UNIX Time-sharing System”) namespace允许每个container拥有独立的hostname和domain name, 使其在网络上可以被视作一个独立的节点而非Host上的一个进程。

6. user namespace 

   每个container可以有不同的user 和group id, 也就是说可以在container内部用container内部的用户执行程序而非Host上的用户。

#### （4）、namespace 的常用操作

- 查看当前系统的`namespace：lsns –t <type>`
- 查看某进程的namespace：`ls -la /proc/<pid>/ns/`
- 进入某namespace运行命令：`nsenter -t <pid> -n ipaddr`

### 三、Cgroups

1. Cgroups（Control Groups）是Linux下用于对一个或一组进程进行资源控制和监控的机制；

2. 可以对诸如CPU使用时间、内存、磁盘I/O等进程所需的资源进行限制；
3. 不同资源的具体管理工作由相应的Cgroup子系统（Subsystem）来实现；针对不同类型的资源限制，只要将限制策略在不同的的子系统上进行关联即可；
4. Cgroups在不同的系统资源管理子系统中以层级树（Hierarchy）的方式来组织管理：每个Cgroup都可以包含其他的子Cgroup，因此子Cgroup能使用的资源除了受本Cgroup配置的资源参数限制，还受到父Cgroup设置的资源限制。

### 四、Linux调度器

内核默认提供了5个调度器，Linux内核使用struct sched_class来对调度器进行抽象：

1. Stop调度器，stop_sched_class：优先级最高的调度类，可以抢占其他所有进程，不能被其他进程抢占；
2. Deadline调度器，dl_sched_class：使用红黑树，把进程按照绝对截止期限进行排序，选择最小进程进行调度运行；
3. RT调度器，rt_sched_class：实时调度器，为每个优先级维护一个队列；
4. CFS调度器，cfs_sched_class：完全公平调度器，采用完全公平调度算法，引入虚拟运行时间概念；
5. IDLE-Task调度器，idle_sched_class：空闲调度器，每个CPU都会有一个idle线程，当没有其他进程可以调度时，调度运行idle线程。

#### （1）、CFS调度器

- CFS是Completely Fair Scheduler简称，即完全公平调度器。
- CFS 实现的主要思想是维护为任务提供处理器时间方面的平衡，这意味着应给进程分配相当数量的处理器。
- 分给某个任务的时间失去平衡时，应给失去平衡的任务分配时间，让其执行。
- CFS通过虚拟运行时间（vruntime）来实现平衡，维护提供给某个任务的时间量。
- vruntime= 实际运行时间*1024 / 进程权重
- 进程按照各自不同的速率在物理时钟节拍内前进，优先级高则权重大，其虚拟时钟比真实时钟跑得慢，但获得比较多的运行时间。

CFS调度器没有将进程维护在运行队列中，而是维护了一个以虚拟运行时间为顺序的红黑树。红黑树的主要特点有：

1. 自平衡，树上没有一条路径会比其他路径长出俩倍。
2. O(log n) 时间复杂度，能够在树上进行快速高效地插入或删除进程。

<img src="https://raw.githubusercontent.com/shiiiiyd/data/main/images/image-20220725115433088.png" alt="红黑树" style="zoom:150%;" />

#### （2）、CFS进程调度

- 在时钟周期开始时，调度器调用__schedule()函数来开始调度的运行。
- _schedule()函数调用 pick_next_task() 让进程调度器从就绪队列中选择一个最合适的进程next，即红黑树最左边的节点。
- 通过context_switch()切换到新的地址空间，从而保证next进程运行。
- 在时钟周期结束时，调度器调用entity_tick()函数来更新进程负载、进程状态以及vruntime（当前vruntime+ 该时钟周期内运行的时间）。
- 最后，将该进程的虚拟时间与就绪队列红黑树中最左边的调度实体的虚拟时间做比较，如果小于坐左边的时间，则不用触发调度，继续调度当前调度实体。

![CFS 进程管理](https://github.com/shiiiiyd/data/blob/main/images/image-20220725115705779.png?raw=true)

### 五、文件系统

#### （1）、Union FS

- 将不同目录挂载到同一个虚拟文件系统下（unite several directories into a single virtual filesystem）的文件系统
- 支持为每一个成员目录（类似GitBranch）设定readonly、readwrite和whiteout-able 权限
- 文件系统分层, 对readonly权限的branch 可以逻辑上进行修改(增量地, 不影响readonly部分的)。
- 通常Union FS 有两个用途, 一方面可以将多个disk挂到同一个目录下, 另一个更常用的就是将一个readonly的branch 和一个writeable 的branch 联合在一起。

#### （2）、Docker的文件系统

典型的Linux文件系统组成：

1. Bootfs（boot file system）•Bootloader-引导加载kernel，
   - Kernel-当kernel被加载到内存中后umountbootfs。
2. rootfs（root file system）
   - /dev，/proc，/bin，/etc等标准目录和文件。
   - 对于不同的linux发行版, bootfs基本是一致的，但rootfs会有差别。

![bootfs and rootfs](https://github.com/shiiiiyd/data/blob/main/images/image-20220725120131772.png?raw=true)



### 六、容器存储驱动

![容器存储驱动](https://github.com/shiiiiyd/data/blob/main/images/image-20220725120533988.png?raw=true)

![rongqiqudong](https://github.com/shiiiiyd/data/blob/main/images/image-20220725120554086.png?raw=true)

OverlayFS也是一种与AUFS类似的联合文件系统，同样属于文件级的存储驱动，包含了最初的Overlay和更新更稳定的overlay2。

Overlay只有两层：upper层和lower层，Lower层代表镜像层，upper层代表容器可写层。

![OverlayFS](https://github.com/shiiiiyd/data/blob/main/images/image-20220725120634242.png?raw=true)

### 参考

[1]:极客时间

