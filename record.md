1.cache中的函数一般都存在globalCacheLock锁，所以在其中调用的函数不能存在有相同的锁
2.ps -ef|grep 命令  显示所有正在运行的命令程序
3./var/lib/thci/thci.conf包含基础配置文件
4.etcd分布式存储主要采用raft算法，存在leader选举机制和日志复制，例如项目中3台cvm即是3个节点，etcd数据库中数据分布在三个节点上
5.sync.Once.Do(f func())只会执行一次,如下

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var once sync.Once

func main() {

	for i, v := range make([]string, 10) {
		once.Do(onces)
		fmt.Println("count:", v, "---", i)
	}
	for i := 0; i < 10; i++ {

		go func() {
			once.Do(onced)
			fmt.Println("213")
		}()
	}
	time.Sleep(4000)
}
func onces() {
	fmt.Println("onces")
}
func onced() {
	fmt.Println("onced")
}
```

6.使用beego.LoadAppConfig来加载配置文件和beego.AppConfig.Int()读取配置文件，详情见manager/conf/config.go中
7.交叉编译   详情百度
8.挂载命令：mount -t cifs -o username="tophc",password=123 //10.30.63.197/Projects /root/TopHC
9.打包命令：tar -zcvf tandanyu.tar.gz tandanyu
10.解压缩命令：tar -zxvf tandanyu.tar.gz或tar -zxvf tandanyu.tar.gz /root/go/tandanyu/
11.date -s @1623742368 转换时间，把int类型的时间转换
13.echo 0 > /sys/block/sdb/queue/rotational 给rotational设置0
14.ps -aux | grep usd 查看usd的进程
16.cache包中的其他文件都有接口调用metadata中的etcd接口，PutMessage（）和GetPrefix（），center中存储的内容不同于manager存储的内容，采用的加载架构也不相同	
19.history  > ~/command.txt 将历史命令重定向到文件中
20.proto.Clone(poolCachePointer).(*icenter.TmsPoolCache) 深复制，用于返回缓存
21.vim常用技巧 ’GG‘表示切换到行尾，’gg‘移动到第一行，’nG‘移动到第n行，'q!'表示强制退出，’wq!‘强制保存退出，’u{n}‘撤销一次或者n次操作，’U‘撤销当前行的所有修改，’ctrl+r‘ 撤销所有操作，普通模式下  输入  '/'然后输入需要查找的字符串回车，这个是向下查找，’?‘是向上查找，用n/N继续向下/向下查找。
22.systemctl restart monitor-agent 重启监视
23.win+R 输入msinfo32 可查看系统各种信息

## #etcd操作

```
1.etcdctl del / --prefix   删除etcd数据库中所有数据
2.etcdctl put /manager/server/mode '{"mode":"tophc","hasconvert":true}' 设置manager的服务模式
3.ETCDCTL_API=3 etcdctl get /manager/server/mode  获取etcd的内容
4.ETCDCTL_API=3 etcdctl put /manager/server/mode \{\"hasconvert\":false,\"mode\":\"tophc\"\}  插入一条键值对
5.etcdctl get --from-key ""   //查找所有的key，如果是我们的程序则加上ETCDCTL_API=3
ETCDCTL_API=3 etcdctl get /manager/vdcagent/TDT-5256ff429e0f




```

## #access操作

```
1.access agent machine  查看主机或者cvm信息
2.access agent host unregister -u 22b18ef0-7118-4549-9dcf-cc26b6887999 注销该台主机
3.access storage disk list  显示所有存储磁盘信息
4.access center usd list  显示所有磁盘信息(适用于cvm)
5.access center volume list | grep 5d65d036-1b83-4937-b1db-68b93889a8d2 显示usd下的卷详情
6.access center debug-old dump-cache > /tmp/cache  将缓存cache的重定向到文件中
7.access storage usd delete --name=data-f077e6e7-ae86-4b99-9611-90b037042b71 --force
```

查找命令：
find / -name manager.service

删除数据库操作:
rm -rf  /var/lib/thci/agent/data/data.db    删除监控的bolt数据库

cobra框架：
https://blog.csdn.net/weixin_30352645/article/details/97302219
https://blog.csdn.net/random_w/article/details/107101081
gdb安装和下载：
http://c.biancheng.net/view/8130.html
beego路由问题
https://blog.csdn.net/qq_33249452/article/details/89678436

grpc：
https://zhuanlan.zhihu.com/p/353881091
grpc官方文档：http://doc.oschina.net/grpc?t=60133
构建和启动grpc服务器：
1.使用listen,err := net.Listen("tcp",fmt.Sprintf(":%d",*port)) 指定客户端的监听端口
2.使用grpc.NewServer()创建gRPC服务器的一个实例
3.在gRPC服务器上注册我们的服务发现
4.用服务器Serve()方法以及我们的端口信息区实现阻塞等待，直到进程被杀死或者stop() 被调用

谭单雨8456

重新搭建环境流程：
1.查看cvm和宿主机的机械信息access agent machine
2.注销宿主机或者cvm： access agent host unregister -u e86bf73f-f2bf-4143-b939-fb8de272bb78
3.清除cvm的所以元数据：etcdctl del / --prefix 
4.删除盘符上的usd使得能界面手动加容量组（主机上的，先用access storage usd list查看盘符名字）      access storage usd delete --name=data-fbc28f04-bd49-4463-983d-66b0f1208812 --force

cvm宕机原因：
根据center日志和监控日志去分析，监控存在一个心跳机制去决定云管是否宕机。本次出现的是数据库问题，监控数据库采用的是bolt数据库，删除指令：rm -rf  /var/lib/thci/agent/data/data.db  

磁盘健康问题：
磁盘检测采用smartctl指令完成的，磁盘健康问题并不是从smart信息中即access agent disks，而是在usd中查看即access storage usd list

时间戳问题：linux服务器上存在两种时间：
CST时间：中国标准时间
EDT时间：美国东部夏令时间，与中国标准时间相差12小时
前端转换时间戳是按照当台PC的时区来转换的。

USD：统一存储设备  VSD：虚拟存储设备

![image-20210709102704160](C:\Users\13065\AppData\Roaming\Typora\typora-user-images\image-20210709102704160.png)

KVM

KVM(Kernel-basedVirtual Machine 基于内核的虚拟机), 狭义 KVM 指的是一个嵌入到 Linux kernel 中的虚拟化功能模块, 该模块在利用 Linux kernel 所提供的部分操作系统能力(如: 任务调度/内存管理/硬件设备交互)的基础上, 再为其加入了虚拟化能力, 使得 Linux kernel 具有了 **转化** 为 Hypervisor(虚拟化管理软件) 的条件
1.KVM内核模块本身只提供CPU和内存虚拟化
2.KVM包含一个提供给CPU的底层虚拟化可加载核心模块kvm.ko(kvm-intel.ko/kvm-AMD.ko)
3.KVM需要在具备IntelVT或AMD-V功能的x86平台上运行，所以KVM也称为硬件辅助的全虚拟化实现

QEMU

QEMU(Quick Emulator) 是一个广泛使用的开源计算机仿真器和虚拟机.QEMU 作为一个独立 Hypervisor(不同于 KVM 需要嵌入到 kernel), 能在应用程序的层面上运行虚拟机. 同时也支持兼容 Xen/KVM 模式下的虚拟化, 并且当 QEMU 运行的虚拟机架构与物理机架构相同时, 建议使用 KVM 模式下的 QEMU, 此时 QEMU 可以利用 kqemu 加速器, 为物理机和虚拟机提供更好的性能
1.当 QEMU 作为仿真器时, QEMU 通过动态转化技术(模拟)为 GuestOS 模拟出 CPU 和其他硬件资源, 让 GuestOS 认为自身直接与硬件交互. QEMU 会将这些交互指令转译给真正的物理硬件之后, 再由物理硬件执行相应的操作. 由于 GuestOS 的指令都需要经过 QEMU 的模拟, 因而相比于虚拟机来说性能较差.
2.当 QEMU 作为一个虚拟机时, QEMU 能够通过直接使用物理机的系统资源, 使虚拟机能够获得接近于物理机的性能表现.
KVM 作为 Linux 的内核模块, 需要被加载后, 才能进一步通过其他工具的辅助以实现虚拟机的创建. 但需要注意的是, KVM 作为运行于 CPU 内核态 的内核模块, 用户是无法直接对其进行操作的. 还必须提供一个运行于 CPU 用户态 的对接程序来提供给用户使用. 而这个对接程序, KVM 的开发者选择了已经成型的开源虚拟化软件 QEMU.  KVM 开发者在对 QEMU 稍加改造之后, QEMU 可以通过 KVM 对外暴露的 /dev/kvm 接口来进行调用, 官方提供的 KVM 下载有 QEMU 和 KVM 两大部分, 包含了 KVM 模块、QEMU 工具以及二者的合集 qemu-kvm 三个文件.

hypervisor：一个软件层，可以把一个物理机用不同的配置虚拟化为多个虚机的集合
domain：虚拟机的一个实例，在容器级虚拟化情况中，是一个子系统，运行在由hypervisor提供的机器上
libvirt：提供一个通用的软件层，安全高效的管理node上的domain，同时实现远程管理功能。
Node：节点，宿主机，一个node是一个单独的物理机，用于运行虚机

postman工具使用：
https://blog.csdn.net/fxbin123/article/details/80428216

linux资料
https://mirrors.edge.kernel.org/pub/

转换debug级别日志
动态开启调试模式 kill -s SIGUSR1 `pidof manager`  
manager client debug log --level=debug --endpoint=<ip地址>:9990
loglevel有四种值："error","info", "debug"

````go
const (
	LevelInfo  = LevelInformational
	LevelTrace = LevelDebug
	LevelWarn  = LevelWarning
)
````

KVM虚拟化的核心主要由以下两个模块组成

1.KVM内核模块,它属于标准Linux内核的一部分，是一个专门提供虚拟化功能的模块，主要负责CPU和内存的虚拟化，包括:客户机的创建、虚拟内存的分配、CPU执行模式的切换、vCPU寄存器的访问、vCPU的执行。
2.QEMU用户态工具，它是一个普通的Linux进程,为客户机提供设备模拟的功能，包括模拟BIOS、PCI/PCIE总线、磁盘、网卡、显卡、声卡、键盘、鼠标等。同时它通过ioctl系统调用与内核态的KVM模块进行交互。
KVM是在硬件虚拟化支持下的完全虚拟化技术，所以它能支持在相应硬件上能运行的几乎所有的操作系统，如：Linux、Windows、FreeBSD、MacOS等。KVM的基础架构。在KVM虚拟化架构下,每个客户机就是一个QEMU进程，在一个宿主机上有多少个虚拟机就会有多少个QEMU进程；客户机中的每一个虚拟CPU对应QEMU进程中的一个执行线程；一个宿主机中只有一个KVM内核模块,所有客户机都与这个内核模块进行交互。

KVM内核模块是KVM虚拟化的核心模块，它在内核中由两部分组成:一个是处理器架构无关的部分，用lsmod命令中可以看到，叫作kvm模块;另一个是处理器架构相关的部分，在Intel平台上就是kvm_intel这个内核模块。KVM的主要功能是初始化CPU硬件，打开虚拟化模式，然后将虚拟客户机运行在虚拟机模式下，并对虚拟客户机的运行提供一定的支持。

virsh命令

```go
1.virsh -c qemu+ssh://root@10.30.12.242/system    //进入和虚拟机的交互模式	
#域管理命令
1.virsh list    //显示宿主机上运行的虚拟机的ID，NAME和Status
2.domstate <ID or Name or UUID>   //获取域的运行状态
3.virsh dominfo <ID>   //获取一个域的基本信息
4.virsh domid <UUID or Name>   //根据域的名称返回域的ID
5.virsh domname <ID or UUID>   //返回域的名字
6.virsh dommemstat <ID>     //获取域的内存使用情况
7.virsh vcpuinfo  <ID>  //获取vCpu的基本信息    //对vcpu的一些设置不展开
#虚拟机的功能命令
1.virsh suspend <ID>     //暂停虚拟机
2.virsh resume <ID>		//唤醒虚拟机
3.virsh shutdown <ID>	//关机
4.virsh reboot <ID>	//重启虚拟机
5.virsh reset <ID>	//强制重启虚拟机
6.virsh destroy <ID>	//销毁虚拟机
7.virsh save <ID> <file.img> //保存一个运行的域的状态到一个文件中
8.virsh restore <file.img> //从一个被保存的文件中恢复一个域的运行
9.virsh migrate <ID> <dest_url> //将一个域迁移到另外一个目的地址
10.virsh console <ID>  //连接到一个域的控制台
#宿主机和Hypervisor的管理命令
1.virsh version			//显示libvirt和Hypervisor的版本信息
2.virsh sysinfo			//以XML格式打印宿主机系统的信息
3.virsh nodeinfo		//显示该节点的基本信息
4.virsh uri				//显示当前连接的URI
5.virsh hostname		//显示当前节点的主机名
#网络、存储卷和存储池命令略
git://git.fedorahosted.org/git/virt-manager.git

```

可参考文档https://blog.csdn.net/weixin_30332241/article/details/99389159

lsmod  |  grep  tun/bridge   //查看tun和bridge是否加载

云终端掉线情况：manager服务重启，管理员踢出，同一个用户不同地方登录

负载均衡：当一台物理服务器的负载较高时，可以将其上运行的客户机动态迁移到负载较低的宿主机服务器中，以保证客户机的服务质量(QoS)。而前面提到的,
CPU、内存的过载使用可以解决某些客户机的资源利用问题，之后当物理资源长期处于超负荷状态时，对服务器稳定性能和服务质量都有损害的，这时需要通过动态迁移来进行适当的负载均衡。





