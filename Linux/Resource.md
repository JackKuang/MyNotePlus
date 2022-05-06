# Linux查看资源占用

## 一、内存

### 1.1 free

```sh
[root@Jack ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1006624      761268       65012         712      180344      107324
Swap:             0           0           0
# 以m作为单位
[root@Jack ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:            983         743          63           0         176         104
Swap:             0           0           0
# 单位最简化
[root@Jack ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           983M        743M         63M        712K        176M        104M
Swap:            0B          0B          0B
```

* total

> 总共物理内存

* used

> 已使用内存

* free

> 剩余内存

* shared

> 多个进程共享的内存

* buff/cache

> 磁盘缓存的大小

* available

> 可用内存
>
> available是可以被应用程序使用的*内存*大小,available = free + buffer + cache

* Swap

> Swap内存

### 1.2 top

```sh
[root@Jack ~]# top
Tasks:  77 total,   2 running,  51 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.4 us,  0.0 sy,  0.0 ni, 96.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1006624 total,    63932 free,   764024 used,   178668 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   104372 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND           
11882 root      10 -10  140396  12836   3064 S  3.3  1.3 129:35.86 AliYunDun         
    1 root      20   0  125392   2748   1428 S  0.0  0.3   2:44.91 systemd           
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.36 kthreadd          
    3 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 rcu_gp            
    4 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 rcu_par_gp        
    6 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 kworker/0:0H-kb   
    8 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 mm_percpu_wq      
    9 root      20   0       0      0      0 S  0.0  0.0   0:42.20 ksoftirqd/0
```

* 与free命令类似，可以查看到整体内存使用情况，以及各个应用的内存消耗情况
* 输入==**M**==：进程将会按照内存消耗大小降序排序
* 输入==**e**==：内存单位切换显示，每次切换单位为KB、MB、GB、TB、PB。

top的内容展示：CentOS

![image-20220402155439630](http://img.hurenjieee.com/uPic/image-20220402155439630.png)



> 第一行是任务队列信息：
> top - 15:55:07 up 304 days, 23:19,  1 user,  load average: 0.02, 0.05, 0.02
>
> * top - 15:55:07 当前时间
> * up 304 days, 23:19 系统运行时间
> * 1 user 当前登陆用户数
> * load average: 0.02, 0.05, 0.02 系统负载（任务队列的平均长度），分别表示过去1、5、15分钟的平均值，若果这个数除以逻辑CPU，结果高于5时表示系统超负荷运载。

> 第二行是任务信息：
>
> Tasks: 107 total,   2 running,  74 sleeping,   0 stopped,   0 zombie
>
> * total 进程总数
> * running 正在运行的进程数
> * sleeping 睡眠的进程数
> * stopped 停止的进程数
> * zombie 僵尸进程数

> 第三行是CPU信息
>
> %Cpu(s):  0.2 us,  0.3 sy,  0.0 ni, 99.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
>
> * 0.2 us, 用户空间占用CPU百分比
> *  0.3 sy, 内核空间占用CPU百分比
> *  0.0 ni, 用户进程空间内改变过优先级的进程占用CPU百分比
> * 99.5 id,空闲CPU百分比
> * 0.0 wa,等待输入输出的CPU时间百分比
> * 0.0 hi,硬中断（Hardware IRQ，硬件设备）占用CPU的百分比
> * 0.0 si,软中断（Software Interrupts，中断指令）占用CPU的百分比
> * 0.0 st

> 第四行是内存信息：
> KiB Mem :  3693920 total,   126068 free,  3261364 used,   306488 buff/cache
>
> * 3693920 total,   物理内存总量
> * 126068 free,  空闲内存总量
> * 3261364 used,   使用的物理内存总量
> * 306488 buff/cache 用户内核缓存的内存总量

> 第五行是交换区内存信息：
>
> KiB Swap:        0 total,        0 free,        0 used.   204532 avail Mem 
>
> *  0 total,        交换区内存总量
> * 0 free,        空闲交换区总量
> * 0 used.   使用的交换区总量
> * 204532 avail Mem 可用内存（包含物理内存和交换区内存）

> 进程信息：
>
>   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
>
> | 列名    | 含义                                                         |
> | :------ | :----------------------------------------------------------- |
> | PID     | 进程id                                                       |
> | PPID    | 父进程id                                                     |
> | RUSER   | Real user name                                               |
> | UID     | 进程所有者的用户id                                           |
> | USER    | 进程所有者的用户名                                           |
> | GROUP   | 进程所有者的组名                                             |
> | TTY     | 启动进程的终端名。不是从终端启动的进程则显示为 ?             |
> | PR      | 优先级（rt字样，表示是实时任务）                             |
> | NI      | nice值。负值表示高优先级，正值表示低优先级。                 |
> | P       | 最后使用的CPU，仅在多CPU环境下有意义                         |
> | %CPU    | 上次更新到现在的CPU时间占用百分比                            |
> | TIME    | 进程使用的CPU时间总计，单位秒                                |
> | TIME+   | 进程使用的CPU时间总计，单位到毫秒                            |
> | %MEM    | 进程使用的物理内存百分比                                     |
> | VIRT    | 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES                |
> | SWAP    | 进程使用的虚拟内存中，被换出的大小，单位kb                   |
> | RES     | 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA    |
> | CODE    | 可执行代码占用的物理内存大小，单位kb                         |
> | DATA    | 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb    |
> | SHR     | 共享内存大小，单位kb                                         |
> | nFLT    | 页面错误次数                                                 |
> | nDRT    | 最后一次写入到现在，被修改过的页面数                         |
> | S       | 进程状态（D-不可中断的睡眠状态，R-运行，S-睡眠，T-跟踪/停止，Z-僵尸） |
> | COMMAND | 命令名/命令行                                                |
> | WCHAN   | 若该进程在睡眠，则显示睡眠中的系统函数名                     |
> | Flags   | 任务标记                                                     |



### 1.3 cat  /proc/meminfo

```sh
[root@Jack ~]# cat  /proc/meminfo
MemTotal:        1006624 kB
MemFree:           61748 kB
MemAvailable:     104736 kB
Buffers:            5244 kB
Cached:           156220 kB
SwapCached:            0 kB
Active:           839408 kB
Inactive:          38436 kB
Active(anon):     716864 kB
Inactive(anon):      236 kB
Active(file):     122544 kB
Inactive(file):    38200 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                36 kB
Writeback:             0 kB
AnonPages:        710044 kB
Mapped:            34756 kB
Shmem:               712 kB
Slab:              38388 kB
SReclaimable:      19796 kB
SUnreclaim:        18592 kB
KernelStack:        3676 kB
PageTables:         6112 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:      503312 kB
Committed_AS:    2036808 kB
VmallocTotal:   34359738367 kB
VmallocUsed:           0 kB
VmallocChunk:          0 kB
Percpu:              284 kB
HardwareCorrupted:     0 kB
AnonHugePages:    507904 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:       75648 kB
DirectMap2M:      972800 kB
DirectMap1G:           0 kB
```

* 显示内容与之前的类似。

## 二、CPU

### 2.1 cat /proc/cpuinfo

```sh
[root@Jack ~]# cat /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 79
model name	: Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz
stepping	: 1
microcode	: 0x1
cpu MHz		: 2494.224
cache size	: 40960 KB
physical id	: 0
siblings	: 1
core id		: 0
cpu cores	: 1
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl cpuid tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single pti ibrs ibpb stibp fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm rdseed adx smap xsaveopt
bugs		: cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs taa itlb_multihit
bogomips	: 4988.44
clflush size	: 64
cache_alignment	: 64
address sizes	: 46 bits physical, 48 bits virtual
power management:
```

### 2.2 top

**top命令，w命令，uptime命令， cat /proc/loadavg命令**都可以查看系统负载。

```
[root@Jack ~]# top
top - 11:18:09 up 104 days, 52 min,  1 user,  load average: 0.02, 0.01, 0.00
Tasks:  77 total,   1 running,  51 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1006624 total,    65388 free,   763996 used,   177240 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   104216 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND           
11882 root      10 -10  140396  14824   5052 S  1.0  1.5 130:02.15 AliYunDun         
22179 root      20   0  825100   6852   1604 S  0.7  0.7   1292:11 staragent-core    
30323 root      20   0  161900   4536   3916 R  0.3  0.5   0:00.02 top               
    1 root      20   0  125392   2748   1428 S  0.0  0.3   2:44.93 systemd           
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.36 kthreadd          
    3 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 rcu_gp            
    4 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 rcu_par_gp        
    6 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 kworker/0:0H-kb   
    8 root       0 -20       0      0      0 I  0.0  0.0   0:00.00 mm_percpu_wq      
    9 root      20   0       0      0      0 S  0.0  0.0   0:42.21 ksoftirqd/0
```

* 输入==**P**==：进程将会按照CPU消耗大小降序排序

* CPU面板配置：

  ![image-20200902112020572](http://img.hurenjieee.com/image-20200902112020572.png)

  1. **load average:**

     > **系统负载（System Load）**是系统CPU繁忙程度的度量，即有多少进程在等待被CPU调度（进程等待队列的长度）。
     >
     > **平均负载（Load Average）**是一段时间内系统的平均负载，这个一段时间一般取1分钟、5分钟、15分钟。
     >
     > **平均负载含义**：理解为CPU的繁忙程度
     >
     > * 单核情况下，Load等于1为资源跑满，小于1为空闲，大于1为过盲。
     > * n核情况下，Load等于n为资源跑满，小于n为空闲，大于n为过盲。

  2. **Tasks**：

     > * total：进程总数
     > * running：正在运行的进程数
     > * sleeping：睡眠的进程数
     > * stopped：停止的进程数
     > * zombie：僵尸的进程数

  3. **%Cpu(s)**

     > * us：用户空间占用CPU百分比
     > * sy：内核空间占用CPU百分比
     > * ni：用户进程空间内改变优先级的进程占用CPU百分比
     > * id：空间CPU百分比
     > * wa：等待输入输出的CPU百分比
     > * hi：硬中断占用CPU的百分比
     > * si：软中断占用CPU的百分比
     > * st：虚拟机占用百分比

## 三、端口

### 3.1 lsof -i:端口号

```sh
[root@Jack ~]# lsof -i:80
COMMAND     PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
docker-pr  3520 root    4u  IPv6    39094      0t0  TCP *:http (LISTEN)
```

面板内容：

1. COMMAND

   > 进程名称

2. POD

   > 进程PID

3. USER

   > 进程所有者

4. FD

   > 文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等。

5. TYPE

   > 文件类型，如DIR、REG等。

6. DEVICE

   > 指定磁盘名称

7. SIZE

   > 文件的大小

8. NODE

   > 索引节点（文件仔磁盘上的表示）

9. NAME

   > 打开文件的确切名称

### 3.2 netstat -tunlp | grep 端口号

```sh
[root@Jack ~]# netstat -tunlp | grep 80
tcp6       0      0 :::8080                 :::*                    LISTEN      18016/docker-proxy  
tcp6       0      0 :::80                   :::*                    LISTEN      3520/docker-proxy
```

参数：

* -t（tcp）显示tcp相关选项
* -u（udp）显示udp相关选项
* -n 拒绝显示别名，能显示数字全部转化为数字
* -l 显示仔Listen（监听）的服务状态
* -p 显示建立相关链接的程序名

## 四、磁盘

### 4.1 df -h

```sh
[root@Jack data]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        482M     0  482M   0% /dev
tmpfs           492M     0  492M   0% /dev/shm
tmpfs           492M  696K  491M   1% /run
tmpfs           492M     0  492M   0% /sys/fs/cgroup
/dev/vda1        40G  4.9G   33G  14% /
overlay          40G  4.9G   33G  14% /var/lib/docker/overlay2/50aff1a5c919353fd83ac50f5273014a272e8916cf08f760de1b69e43a9d74a2/merged
overlay          40G  4.9G   33G  14% /var/lib/docker/overlay2/c4f3131c55840e82065e5a1b7a3c4c3d877606b85936a0d1ed55f81992a3e4f6/merged
tmpfs            99M     0   99M   0% /run/user/0
```

* 查看系统上的文件系统的磁盘使用情况统计。

### 4.2 du -h

```sh
[root@Jack data]# du -h --max-depth=1 /data
12K     /data/sh
320M    /data/jenkins
15M     /data/common
334M    /data
[root@Jack data]# du -h --max-depth=1
12K     ./sh
320M    ./jenkins
15M     ./common
334M    .
```

* 查看目录或者文件夹大小

### 4.3 iostat

```sh
[root@Jack ~]# iostat
Linux 4.19.91-18.al7.x86_64 (Jack)      09/02/2020      _x86_64_        (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.01    0.00    1.14    0.09    0.00   97.76

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               1.02        12.26         7.04  109388004   62765092
# 定时显示所有信息，每隔2秒刷新显示，刷新3次
[root@Jack ~]# iostat 2 3
# 显示指定磁盘的信息
[root@Jack ~]# iostat -d /dev/sda
```

**cpu属性值说明**：

> %user:CPU处在用户模式下的时间百分比
>
> %nice:CPU处于带NICE值的用户模式下的百分比
>
> %system:CPU在系统模式下的百分比
>
> %iowait:CPU等待输入输出完成时间的百分比
>
> %steal:管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比
>
> %idle:CPU空闲时间百分比

**备注内容**：

> 如果%iowait的值过高，表示硬盘存在I/O瓶颈
>
> 如果%idle值高，表示CPU空闲
>
> 如果%idle值高但是系统响应满，可能CPU等待分配内存，应该加大内存。
>
> 如果%idle值持续低于10，表明CPU处理能力相对较低，系统中最需要的解决的资源是CPU

**CPU属性值说明**：

> tps：该设备每秒的传输次数
>
> kB_read/s：每秒从设备（drive expressed）读取的数据量
>
> kB_wrtn/s：每秒从设备（drive expressed）写入的数据量
>
> kB_read：读取的总数据量
>
> kB_wrtn：写入的总数据量

```sh
[root@Jack ~]# iostat -d -x -k 1 1
Linux 4.19.91-18.al7.x86_64 (Jack)      09/03/2020      _x86_64_        (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.09     0.64    0.18    0.85    12.28     7.04    37.71     0.01    6.23   18.90    3.59   0.96   0.10
```

说明：

> rrqm/s：每秒进行merge的读操作数目，即delta(rmerge)/s
>
> wrqm/s：每秒进行merge的写操作数目，即delta(merge)/s
>
> %util：一秒中有百分之多少的时间用于I/O，如果接近于100%，说明产生I/O请求太多，I/O系统已经满负荷，
>
> idle小于70%，IO就比较大时，一般读取数据就会有较多wait。

## 五、网络

### 5.1 iftop

* iftop类似于top的实时流量监控工具，可以用来监控网卡的实时流量（可以指定网段）、反向解析IP、显示端口信息等。
  * 增加-P选项会在iftop的数据结果中开启端口显示。

```sh
              1.91Mb        3.81Mb        5.72Mb        7.63Mb  9.54Mb
└─────────────┴─────────────┴─────────────┴─────────────┴─────────────
Jack                  => 115.206.126.229       1.53Kb  1.15Kb  0.98Kb
                      <=                        416b    384b    289b
Jack                  => 100.100.45.100        1.44Kb   672b    451b
                      <=                        160b    147b    123b
Jack                  => 100.100.30.26            0b      0b      9b
                      <=                          0b      0b     18b


──────────────────────────────────────────────────────────────────────
TX:             cum:   2.00MB   peak: rates:Kb 2.97Kb  1.81Kb  1.43Kb
RX:                    65.3KB           1.12Kb  576b    531b    430b
TOTAL:                 2.06MB           3.97Kb 3.53Kb  2.33Kb  1.85Kb
```

面板内容：

> * 横向
>   * TX：发送流量
>   * RX：接收流量
>   * TOTAL：总流量
> * 纵向
>   * cum：运行iftop到目前累计时间的总流量
>   * peak：流量峰值
>   * rates：分别在过去2s 10s 40s的平均流量

### 5.2 nload

* 可以用来监控入站流量和出栈流量，额外提供了绘制图表的功能

  ![image-20200903212409629](http://img.hurenjieee.com/uPic/image-20200903212409629.png)

## 六、Java

### 6.1 jps（进程监控）

```sh
jenkins@b3574b7a2f27:/$ jps
24482 Jps
6 jenkins.war
jenkins@b3574b7a2f27:/$ jps -l
6 /usr/share/jenkins/jenkins.war
24492 sun.tools.jps.Jps
jenkins@b3574b7a2f27:/$ jps -v
6 jenkins.war -Duser.home=/var/jenkins_home -Djenkins.model.Jenkins.slaveAgentPort=50000
24503 Jps -Dapplication.home=/usr/local/openjdk-8 -Xms8m
```

* java程序启动后，会在目录/tmp/hsperfdata_{userName}/下生成几个文件，文件名就是java进程的pid，因此jps列出进程id就是把这个目录下的文件明列一下而已，至于系统参数，则是读取文件内容。

* 查看Java进程

```sh
jack@Jacks-MacBook-Pro Linux % jps
19534 Jps
```

* jps -l

  > 输出应用程序main.class的完整package名或者应用程序jar文件完成路径名

* jps -v

  > 输出传递给JVM的参数

### 6.2 jstack（栈监控）

```sh
jenkins@b3574b7a2f27:/$ jstack 6
2020-09-03 22:47:06
Full thread dump OpenJDK 64-Bit Server VM (25.242-b08 mixed mode):

"Attach Listener" #54947 daemon prio=9 os_prio=0 tid=0x00007f90dc02f800 nid=0x5fcb waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Jetty (winstone)-47140" #47140 prio=5 os_prio=0 tid=0x00007f90ec546800 nid=0x3fe8 waiting on condition [0x00007f90c7856000]
   java.lang.Thread.State: TIMED_WAITING (parking)
```

* 主要用于生成指定进程当前时刻的线程快照，线程快照时当前java虚拟机每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源时间过长等待。

### 6.3 jmap（内存监控）

```sh
jenkins@b3574b7a2f27:/$ jmap 6
Attaching to process ID 6, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.242-b08
0x0000000000400000      8K      /usr/local/openjdk-8/bin/java
0x00007f90c9382000      82K     /lib/x86_64-linux-gnu/libresolv-2.24.so
0x00007f90c9599000      22K     /lib/x86_64-linux-gnu/libnss_dns-2.24.so
0x00007f90cdfff000      201K    /usr/lib/x86_64-linux-gnu/libpng16.so.16.28.0
0x00007f90ce232000      102K    /lib/x86_64-linux-gnu/libz.so.1.2.8
0x00007f90ce44c000      699K    /usr/lib/x86_64-linux-gnu/libfreetype.so.6.12.3
0x00007f90ce6fb000      485K    /usr/local/openjdk-8/jre/lib/amd64/libfontmanager.so
0x00007f90ce96f000      38K     /usr/local/openjdk-8/jre/lib/amd64/libawt_headless.so
0x00007f90ceb77000      757K    /usr/local/openjdk-8/jre/lib/amd64/libawt.so
0x00007f90cf250000      90K     /lib/x86_64-linux-gnu/libgcc_s.so.1
0x00007f90cf46b000      260K    /usr/local/openjdk-8/jre/lib/amd64/libsunec.so
0x00007f90d87c9000      51K     /usr/local/openjdk-8/jre/lib/amd64/libmanagement.so
0x00007f90db9d4000      114K    /usr/local/openjdk-8/jre/lib/amd64/libnet.so
0x00007f90dbbec000      92K     /usr/local/openjdk-8/jre/lib/amd64/libnio.so
0x00007f910472b000      123K    /usr/local/openjdk-8/jre/lib/amd64/libzip.so
0x00007f9104947000      46K     /lib/x86_64-linux-gnu/libnss_files-2.24.so
0x00007f9104b59000      46K     /lib/x86_64-linux-gnu/libnss_nis-2.24.so
0x00007f9104d65000      86K     /lib/x86_64-linux-gnu/libnsl-2.24.so
0x00007f9104f7d000      30K     /lib/x86_64-linux-gnu/libnss_compat-2.24.so
0x00007f9105185000      202K    /usr/local/openjdk-8/jre/lib/amd64/libjava.so
0x00007f91053af000      68K     /usr/local/openjdk-8/jre/lib/amd64/libverify.so
0x00007f91055bf000      31K     /lib/x86_64-linux-gnu/librt-2.24.so
0x00007f91057c7000      1038K   /lib/x86_64-linux-gnu/libm-2.24.so
0x00007f9105acb000      15981K  /usr/local/openjdk-8/jre/lib/amd64/server/libjvm.so
0x00007f9106a38000      1649K   /lib/x86_64-linux-gnu/libc-2.24.so
0x00007f9106dd7000      14K     /lib/x86_64-linux-gnu/libdl-2.24.so
0x00007f9106fdb000      334K    /usr/local/openjdk-8/lib/amd64/jli/libjli.so
0x00007f91071f2000      132K    /lib/x86_64-linux-gnu/libpthread-2.24.so
0x00007f910740f000      149K    /lib/x86_64-linux-gnu/ld-2.24.so
```

* 主要用于打印指定java进程的共享对象内存映射或者堆内存细节。

* jmap pid

  > 打印的信息分别为：共享对象的起始地址、映射大小、共享对象路径的全程。

* jmap -heap pid

  ```sh
  jenkins@b3574b7a2f27:/$ jmap -heap 6
  Attaching to process ID 6, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 25.242-b08
  
  using thread-local object allocation.
  Mark Sweep Compact GC
  
  Heap Configuration:
     MinHeapFreeRatio         = 40
     MaxHeapFreeRatio         = 70
     MaxHeapSize              = 257949696 (246.0MB)
     NewSize                  = 5570560 (5.3125MB)
     MaxNewSize               = 85983232 (82.0MB)
     OldSize                  = 11206656 (10.6875MB)
     NewRatio                 = 2
     SurvivorRatio            = 8
     MetaspaceSize            = 21807104 (20.796875MB)
     CompressedClassSpaceSize = 1073741824 (1024.0MB)
     MaxMetaspaceSize         = 17592186044415 MB
     G1HeapRegionSize         = 0 (0.0MB)
  
  Heap Usage:
  New Generation (Eden + 1 Survivor Space):
     capacity = 68222976 (65.0625MB)
     used     = 51248472 (48.874351501464844MB)
     free     = 16974504 (16.188148498535156MB)
     75.11908011752521% used
  Eden Space:
     capacity = 60686336 (57.875MB)
     used     = 50443896 (48.10704803466797MB)
     free     = 10242440 (9.767951965332031MB)
     83.12232921756885% used
  From Space:
     capacity = 7536640 (7.1875MB)
     used     = 804576 (0.767303466796875MB)
     free     = 6732064 (6.420196533203125MB)
     10.675526494565217% used
  To Space:
     capacity = 7536640 (7.1875MB)
     used     = 0 (0.0MB)
     free     = 7536640 (7.1875MB)
     0.0% used
  tenured generation:
     capacity = 151359488 (144.34765625MB)
     used     = 114464528 (109.16188049316406MB)
     free     = 36894960 (35.18577575683594MB)
     75.62428329567288% used
  
  38119 interned Strings occupying 3450392 bytes.
  ```

  > 查看堆使用情况

* jmap -histo pid

  ```sh
  jenkins@b3574b7a2f27:/$ jmap -histo 6
  
   num     #instances         #bytes  class name
  ----------------------------------------------
     1:          5691       51126512  [F
     2:        367105       31891392  [C
     3:        366530        8796720  java.lang.String
     4:          8354        8578080  [I
     5:        208379        6668128  java.util.HashMap$Node
  ```

  

  > 查看堆中对象数量和大小
  >
  > 打印的信息分别是序列号，Class实例的数量，内存的占用，类限定名。
  >
  > 如果是那部类，类名的开头会加上*，如果加上live子参数的话，如jmap -histo:live pod，这个命令会触发一次FULL GC，只统计存货对象。

* jmap -dump:format=b,file=heapdump pid

  > 将内存使用的详细情况输出到文件
  >
  > 然后使用jhat命令查看该文件：jhat -port 4000 文件名，在浏览器中访问 http://localhost:4000/

* 总结：

  > 该命令适用的场景是程序内存不足或者GC频繁，这时候很可能是内存泄漏。通过以上命令查看堆使用情况、大量对象被持续引用等情况。

### 6.4 jstat（资源性能监控）

* 主要是对java应用程序的资源和性能进行实时的命令行监控，包含对heap size和垃圾回收状况的监控。

> jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
>
> option: 选项有gc、gcutil
>
> vmid：java进程id
>
> interval：时间间隔
>
> count：打印次数

* gc

  ```sh
  jenkins@b3574b7a2f27:/$ jstat -gc 6 5000 2
   S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
  7360.0 7360.0 805.4   0.0   59264.0   9490.5   147812.0   57928.8   103768.0 90905.3 12928.0 9930.8  15562   93.627  20      5.592   99.219
  7360.0 7360.0 805.4   0.0   59264.0   9726.7   147812.0   57928.8   103768.0 90905.3 12928.0 9930.8  15562   93.627  20      5.592   99.219
  ```

  面板内容：

  > S0C：年轻代第一个survivor大小
  >
  > S1C：年轻代第二个survivor已使用
  >
  > S0U：年轻代第一个survivor大小
  >
  > S1U：年轻代第二个survivor已使用
  >
  > EC：年轻代中Eden大小
  >
  > EU：年轻代中Eden已使用
  >
  > OC：老年代大小
  >
  > OU：老年代已使用
  >
  > MC：方法区大小
  >
  > MU：方法区已使用
  >
  > CCSC：压缩类空间大小
  >
  > CCSU：压缩类空间已使用
  >
  > YGC：年轻代垃圾回收次数
  >
  > YGCT：年轻代垃圾回收消耗时间（s）
  >
  > FGC：老年代垃圾回收时间
  >
  > FGCT：老年代垃圾回收小号时间（s）
  >
  > GCT：垃圾回收总时间

* gcutil

  ```sh
  jenkins@b3574b7a2f27:/$ jstat -gcutil 6 5000 2
    S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
   10.94   0.00  77.49  39.19  87.60  76.82  15562   93.627    20    5.592   99.219
   10.94   0.00  77.55  39.19  87.60  76.82  15562   93.627    20    5.592   99.219
  ```

  面板内容：

  > S0：年轻代第一个Survivor使用占比
  >
  > S1：年轻代第二个Survivor使用占比
  >
  > E：年轻代Eden去使用占比
  >
  > O：老年代中已使用占比
  >
  > M：方法区已使用占比
  >
  > CCS：压缩使用占比
  >
  > YGC：年轻代垃圾回收次数
  >
  > YGCT：年轻代垃圾回收消耗时间（s）
  >
  > FGC：老年代垃圾回收时间
  >
  > FGCT：老年代垃圾回收小号时间（s）
  >
  > GCT：垃圾回收总时间

### 6.5 jhat（dump文件转化web服务）

* 生成dump文件的前面已经介绍过了（jmap -dump:format=b,file=heapdump pid）。jhat主要用来解析java堆dump并启动一个web服务器，然后就可以在浏览器中查看堆的dump文件了。

```sh
jenkins@ac1a28c8678d:~$ jmap -dump:format=b,file=heapdump 6
Dumping heap to /var/jenkins_home/heapdump ...
File exists
jenkins@ac1a28c8678d:~$ jhat heapdump
Reading from heapdump...
Dump file created Sat Sep 05 17:06:59 CST 2020
Snapshot read, resolving...
Resolving 1295479 objects...
Chasing references, expect 259 dots...................................................................................................................................................................................................................................................................
Eliminating duplicate references...................................................................................................................................................................................................................................................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
# 指定一个自定义端口
jhat -port 4000 heapdump
```

* 这个命令将heapdump文件转换成html格式，并且启动一个http服务，默认端口为7000。

### 6.6 jinfo（查询/修改扩展参数）

* jinfo 可以用来看正在运行的java运行程序的扩展参数，甚至支持在运行时动态地更改部分参数。

```sh
jenkins@ac1a28c8678d:~$ jinfo 6
Attaching to process ID 6, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.242-b08
Java System Properties:

java.runtime.name = OpenJDK Runtime Environment
jna.platform.library.path = /usr/lib/x86_64-linux-gnu:/lib/x86_64-linux-gnu:/lib64:/usr/lib:/lib:/usr/local/openjdk-8/lib/amd64/jli:/usr/local/openjdk-8/lib/amd64
java.vm.version = 25.242-b08
sun.boot.library.path = /usr/local/openjdk-8/jre/lib/amd64
mail.smtp.sendpartial = true
java.vendor.url = http://java.oracle.com/
java.vm.vendor = Oracle Corporation
path.separator = :
file.encoding.pkg = sun.io
java.vm.name = OpenJDK 64-Bit Server VM
jna.loaded = true
sun.os.patch.level = unknown
sun.java.launcher = SUN_STANDARD
user.dir = /
java.vm.specification.name = Java Virtual Machine Specification
java.runtime.version = 1.8.0_242-b08
java.awt.graphicsenv = sun.awt.X11GraphicsEnvironment
os.arch = amd64
java.endorsed.dirs = /usr/local/openjdk-8/jre/lib/endorsed
line.separator = 

java.io.tmpdir = /tmp
java.vm.specification.vendor = Oracle Corporation
os.name = Linux
mail.smtps.sendpartial = true
sun.jnu.encoding = UTF-8
svnkit.http.methods = Digest,Basic,NTLM,Negotiate
jnidispatch.path = /var/jenkins_home/.cache/JNA/temp/jna3155463234972793452.tmp
jetty.git.hash = 271836e4c1f4612f12b7bb13ef5a92a927634b0d
java.library.path = /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
java.specification.name = Java Platform API Specification
java.class.version = 52.0
sun.management.compiler = HotSpot 64-Bit Tiered Compilers
os.version = 4.19.91-18.al7.x86_64
jenkins.model.Jenkins.slaveAgentPort = 50000
user.home = /var/jenkins_home
user.timezone = Asia/Shanghai
java.awt.printerjob = sun.print.PSPrinterJob
file.encoding = UTF-8
java.specification.version = 1.8
user.name = jenkins
java.class.path = /usr/share/jenkins/jenkins.war
java.vm.specification.version = 1.8
sun.arch.data.model = 64
svnkit.ssh2.persistent = false
sun.java.command = /usr/share/jenkins/jenkins.war
java.home = /usr/local/openjdk-8/jre
user.language = en
java.specification.vendor = Oracle Corporation
awt.toolkit = sun.awt.X11.XToolkit
java.vm.info = mixed mode
java.version = 1.8.0_242
java.ext.dirs = /usr/local/openjdk-8/jre/lib/ext:/usr/java/packages/lib/ext
sun.boot.class.path = /usr/local/openjdk-8/jre/lib/resources.jar:/usr/local/openjdk-8/jre/lib/rt.jar:/usr/local/openjdk-8/jre/lib/sunrsasign.jar:/usr/local/openjdk-8/jre/lib/jsse.jar:/usr/local/openjdk-8/jre/lib/jce.jar:/usr/local/openjdk-8/jre/lib/charsets.jar:/usr/local/openjdk-8/jre/lib/jfr.jar:/usr/local/openjdk-8/jre/classes
java.awt.headless = true
java.vendor = Oracle Corporation
file.separator = /
java.vendor.url.bug = http://bugreport.sun.com/bugreport/
sun.io.unicode.encoding = UnicodeLittle
sun.font.fontmanager = sun.awt.X11FontManager
sun.cpu.endian = little
executable-war = /usr/share/jenkins/jenkins.war
sun.cpu.isalist = 

VM Flags:
Non-default VM flags: -XX:CICompilerCount=2 -XX:InitialHeapSize=16777216 -XX:MaxHeapSize=257949696 -XX:MaxNewSize=85983232 -XX:MinHeapDeltaBytes=196608 -XX:NewSize=5570560 -XX:OldSize=11206656 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops 
Command line:  -Duser.home=/var/jenkins_home -Djenkins.model.Jenkins.slaveAgentPort=50000
```

```sh
# 显示新生代对象晋升到老年代对象的最大年龄。在运行程序运行时并没有指定这个参数，但是可以通过jinfo查看这个参数设置的值
jenkins@ac1a28c8678d:~$ jinfo -flag MaxTenuringThreshold 6
-XX:MaxTenuringThreshold=15
# 显示是否打印GC详细信息
jenkins@ac1a28c8678d:~$ jinfo -flag PrintGCDetails 6
-XX:-PrintGCDetails
# 打开打印GC
jenkins@ac1a28c8678d:~$ jinfo -flag +PrintGCDetails 6
# 显示是否打印GC详细信息
jenkins@ac1a28c8678d:~$ jinfo -flag PrintGCDetails 6
-XX:+PrintGCDetails
# jinfo只支持部分的参数动态修改
```

### 6.7 jcmd（多功能命令行）

* jdk1.7之后提供的多功能命令行工具，可以用来导出堆、查看java进程、导出线程信息，执行GC等。jcmd拥有jmap的大部分功能，Oracle建议使用jcmd代替jmap。

```sh
jenkins@ac1a28c8678d:~$ jcmd -l
6 /usr/share/jenkins/jenkins.war
520 sun.tools.jcmd.JCmd -l
jenkins@ac1a28c8678d:~$ jcmd 6 help
6:
The following commands are available:
# help可以替换以下命令
VM.native_memory # 原生内存
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
VM.classloader_stats
GC.rotate_log
Thread.print	# 打印线程
GC.class_stats
GC.class_histogram	# 系统中类统计信息
GC.heap_dump	# 导出堆内存信息
GC.finalizer_info
GC.heap_info	# 堆信息
GC.run_finalization	# GC.run_finalization 
GC.run	# 触发GC
VM.uptime	# VM启动时间
VM.dynlibs
VM.flags
VM.system_properties	# 获取系统Properties
VM.command_line	# 启动命令参数
VM.version
help

For more information about a specific command use 'help <command>'.
jenkins@ac1a28c8678d:~$ jcmd 6 VM.native_memory
6:
Native memory tracking is not enabled

jenkins@ac1a28c8678d:~$ jcmd 6 VM.flags
6:
-XX:CICompilerCount=2 -XX:InitialHeapSize=16777216 -XX:MaxHeapSize=257949696 -XX:MaxNewSize=85983232 -XX:MinHeapDeltaBytes=196608 -XX:NewSize=5570560 -XX:OldSize=11206656 -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops
```

### 6.8 JVisualVM（可视化工具）

自带监控工具

![image-20200905182005538](http://img.hurenjieee.com/uPic/image-20200905182005538.png)

### 6.9 Arthas

Alibaba开源的Java诊断工具：[Arthas](https://github.com/alibaba/arthas/blob/master/README_CN.md)

## 七、其他

* 暂无

