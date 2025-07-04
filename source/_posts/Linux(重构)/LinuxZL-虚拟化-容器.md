---
title:  LinuxZL-虚拟化-容器
date: 2024-12-15 00:00:00
tags: [Linux笔记]
---
本章我们来看下目前流行的容器的相关产品及原理。其实在目前这个时间点，如果有人说他们在生产系统里使用『容器』，那么十有八九就是基于docker的。不过其实在docker诞生之前就有很多公司使用了类似『容器』的概念在管理他们的系统，例如Google、阿里巴巴等。那么在没有docker的时候这些公司是如何提供『容器』的呢？答案很简单，docker的技术栈是基于namespace和cgroup的，namespace在很早的时候就在linux kernel中存在了（当然在其它的OS中也有这个功能），在2008年的时候RedHat的工程师提交了关于user namespace的一个重要补丁并合并入kernel主线分支，而cgroup这个功能则由Google的工程师在2006年开始开发并在2008年的时候提交并合并入kernel主线分支。基于namespace和cgroup这两项技术在2008年的时候IBM的工程师开发了LXC，这个可以看做是docker的前身，后来在2013年的时候Docker公司公布了他们内部使用的容器管理工具docker，最开始的版本是基于LXC的，后来逐渐的减少了对LXC的依赖，并随着libcontainer的出现慢慢的变的成熟，并形成了目前这样一个热门的产品。如果要比较LXC和docker的区别，其实从历史就可以看到，最开始的docker是基于LXC并提供了更多的功能的，例如Dockerfile、Docker hub等等。

本节笔者会介绍一下namespace的使用及其在内核中的实现。

### namespace简介及例子

namespace这个概念在很多编程语言中都有，用处是防止函数名之类的符号冲突。在内核中的namespace的作用则是用于对某个进程或一组进程提供一个隔离的内核上下文资源环境。处于同一个namespace下的进程看到的全局资源对于他们来说是全局的，但是对于处于namespace之外的进程来说则看不到(但namespace的隔离并不仅仅针对进程，这个我们下面会看到具体的针对网卡的例子)。

我们来看一个例子（需要注意namespace对内核的版本有要求，不过从读者看本书的时间点来说读者使用的内核版本应该都满足条件。另外下面例子中对ip命令的版本有要求，建议使用较新的版本）:

```plain
[root@dev ~]# ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 08:00:27:89:4a:1e brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 08:00:27:77:2b:ca brd ff:ff:ff:ff:ff:ff
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 08:00:27:66:66:88 brd ff:ff:ff:ff:ff:ff
5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 08:00:27:5e:84:f9 brd ff:ff:ff:ff:ff:ff
6: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT
    link/ether 52:54:00:27:e8:bc brd ff:ff:ff:ff:ff:ff
7: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN mode DEFAULT qlen 500
    link/ether 52:54:00:27:e8:bc brd ff:ff:ff:ff:ff:ff
[root@dev ~]# ip netns list
[root@dev ~]# ip netns add ns01
[root@dev ~]# ip netns list
ns01
[root@dev ~]# ip netns exec ns01 ip l
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

ip l 这个命令的作用是查看当前的网卡设备。ip netns list这个命令的作用是查看当前有哪些网络namespace，目前内核一共支持七种namespace，分别是：

| Namespace类型 | 说明                                  | 编程常量        |
| ------------- | ------------------------------------- | --------------- |
| Cgroup        | 用于隔离CGroup的根目录                | CLONE_NEWCGROUP |
| Network       | 用于隔离网络设备，如协议栈，端口等    | CLONE_NEWNET    |
| IPC           | 用于隔离System V IPC以及POSIX消息队列 | CLONE_NEWIPC    |
| Mount         | 用于隔离挂载点                        | CLONE_NEWNS     |
| PID           | 用于隔离进程的PID                     | CLONE_NEWPID    |
| User          | 用于隔离用户ID和组ID                  | CLONE_NEWUSER   |
| UTS           | 用于隔离主机名和NIS域名               | CLONE_NEWUTS    |

这里我们用到的就是Network这个类型的namespace。ip netns add ns01这个命令的含义是新建一个名字为ns01的namespace，而ip netns exec ns01 ip l这个命令则是在ns01这个namespace中执行ip l命令。可以看到在ns01中只有lo这个本地回环虚拟网卡，并且其中看不到主机上直接执行ip l命令能看到的那些网卡。后面我们看内核实现的时候就会看到，直接执行ip l命令其实是在一个默认的namespace中查看网络设备，这个默认的namespace在内核启动的时候自动创建。换句话说我们的内核其实一直都是跑在一个默认的namespace下面的。

现在笔者再来做些操作来让大家熟悉下namespace的直观体验。在OpenStack或者是Docker的网络中会看到一个veth的东西，他的作用是建立两个虚拟网卡，例如veth01和veth02，这两个虚拟网卡有一个特点就是发送给veth01的数据流量会直接放到veth02的接收队列中，环境话说如果你发送数据包给veth01，那么在veth02这个网络上读取就能读到这个数据包。这就好比是用一根网线的两头直接插在这两个网卡上一样。我们这里做个操作，建立一对veth设备，然后将其中的一个设备放到nsa这个namespace中，另一个放到nsb这个namespace中，然后在nsa中走veth设备发送数据，并在nsb中监听veth，看下是否有数据包出现：

```plain
[root@dev ~]# ip netns add nsa
[root@dev ~]# ip netns add nsb
[root@dev ~]# ip link add veth0 type veth peer name veth1
[root@dev ~]# ip link set veth0 netns nsa
[root@dev ~]# ip link set veth1 netns nsb
[root@dev ~]# ip netns exec nsa ip l
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
9: veth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT qlen 1000
    link/ether be:b5:28:f3:c5:d9 brd ff:ff:ff:ff:ff:ff
[root@dev ~]# ip netns exec nsb ip l
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
8: veth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT qlen 1000
    link/ether 26:47:03:7d:b9:95 brd ff:ff:ff:ff:ff:ff
[root@dev ~]# ip netns exec nsa ip addr add 10.99.0.1 dev veth0
[root@dev ~]# ip netns exec nsb ip addr add 10.99.0.2 dev veth1
[root@dev ~]# ip netns exec nsa ip link set veth0 up
[root@dev ~]# ip netns exec nsb ip link set veth1 up
[root@dev ~]# ip netns exec nsa ip route add default via 10.99.0.1
[root@dev ~]# ip netns exec nsb ip route add default via 10.99.0.2
```

然后我们开两个窗口，其中一个用于在nsa中发数据包到nsb，另一个则在nsb中抓包看是否有数据过来：

```plain
# 窗口A
[root@dev ~]# ip netns exec nsa ping 10.99.0.2 -c 10000
```

```plain
# 窗口B
[root@dev ~]# ip netns exec nsb tcpdump -ni veth1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on veth1, link-type EN10MB (Ethernet), capture size 65535 bytes
22:04:42.029883 IP 10.99.0.1 > 10.99.0.2: ICMP echo request, id 10339, seq 209, length 64
22:04:42.029950 IP 10.99.0.2 > 10.99.0.1: ICMP echo reply, id 10339, seq 209, length 64
22:04:43.029938 IP 10.99.0.1 > 10.99.0.2: ICMP echo request, id 10339, seq 210, length 64
22:04:43.030007 IP 10.99.0.2 > 10.99.0.1: ICMP echo reply, id 10339, seq 210, length 64
22:04:44.030363 IP 10.99.0.1 > 10.99.0.2: ICMP echo request, id 10339, seq 211, length 64
22:04:44.030447 IP 10.99.0.2 > 10.99.0.1: ICMP echo reply, id 10339, seq 211, length 64
```

而此时我们在主机的默认namespace下是看不到这两个veth虚拟网卡的：

```plain
[root@dev ~]# ip l | grep -i veth | wc -l
0
```

从这个例子我们可以看到，通过namespace，我们的主机从网络这个角度来说就好像有了三台主机：宿主机本身一台主机，nsa一台主机，nsb一台主机。并且nsa和nsb分别有自己的网卡。另外我们也可以看到，namespace的隔离并不仅仅是针对进程的，这里的网络设备同样存在namespace的隔离。可以想象基于内核目前提供的七种namespace，我们的隔离可以做的更加彻底，例如nsa的/etc目录可以和nsb的/etc目录完全不一样（基于Mount类型），nsa中ps命令看到的进程数目可以和nsb中ps命令看到的不一样（基于PID类型）等等。从资源的『name』隔离这个角度来说，基于namespace可以实现非常类似虚拟机隔离的效果。这也是容器能流行的一个原因，如果读者自己动手执行了上面例子中的命令就会发现创建一个namespace是非常迅速的，而创建一台虚拟机则花费的时间则要多许多。

### namespace在内核中的实现

这里我们还是以网络的namespace为例介绍一下namespace在内核中的实现。下面这个例子可以帮助大家在Python伪代码层面理解namespace的实现。在没有引入namespace之前，获取当前系统上网卡名字的接口可能是类似于下面的实现：

```plain
interfaces = ["eth0", "eth1"]

def get_interfaces():
	return interfaces

# 获取网卡列表
get_interfaces()
```

在有了namespace后，实现变为：

```plain
class NS(object):

	def __init__():
		self.interfaces = []

	def add_interface(ifname):
		self.interfaces.append(ifname)

	def get_interfaces():
		return self.interfaces

def get_interfaces(ns):
	return ns.get_interfaces()

ns_apple = NS()
ns_apple.add_interface("eth0")
ns_apple.add_interface("eth1")

ns_pear = NS()
ns_pear.add_interface("eth0")

# apple namespace下的网卡列表
apple_ifs = get_interfaces(ns_apple)
# pear namespace下的网卡列表
pear_ifs = get_interfaces(ns_pear)
```

这里的关键就是为namespace建立一个单独的数据结构NS，将对namespace有『意识』的内核资源（如网卡）关联到这个数据结构下，同时对所有获取这些内核资源的内核接口都做一个改造，将原先直接获取列表的逻辑变为从对应的NS数据结构中获取。当然具体能看到哪个namespace是由进程决定的。是不是很简单？但实际上内核开发者们的工作还是巨大的，因为对于所有涉及到的内核资源和相关的接口都需要增加这么一个指向NS结构体的指针或类似的东西。

我们来看下网络类型的namespace的实现。上面的例子中一个NS对象就是一个namespace，对于内核来说，一个进程如何知道自己在哪个namespace下操作是依赖进程task_struct的nsproxy属性来判断的。task_struct这个结构体大家可以认为在内核中就是代表一个task（即进程）：

```plain
/* namespaces */
    struct nsproxy *nsproxy;

/*
 * A structure to contain pointers to all per-process
 * namespaces - fs (mount), uts, network, sysvipc, etc.
 *
 * The pid namespace is an exception -- it's accessed using
 * task_active_pid_ns.  The pid namespace here is the
 * namespace that children will use.
 *
 * 'count' is the number of tasks holding a reference.
 * The count for each namespace, then, will be the number
 * of nsproxies pointing to it, not the number of tasks.
 *
 * The nsproxy is shared by tasks which share all namespaces.
 * As soon as a single namespace is cloned or unshared, the
 * nsproxy is copied.
 */
struct nsproxy {
    atomic_t count;
    struct uts_namespace *uts_ns;
    struct ipc_namespace *ipc_ns;
    struct mnt_namespace *mnt_ns;
    struct pid_namespace *pid_ns_for_children;
    struct net       *net_ns;
};
extern struct nsproxy init_nsproxy;
```

我们可以把nsproxy简单的类比成我们上面举例的NS类。只不过内核对nsproxy做了细分，分成了uts、ipc、mnt、pid以及net这几个namespace，我们的网络namespace就是这里的net。也就是说原来的代码一个进程如果想要查看当前环境下的网卡列表是通过调用get_interfaces查看的获取的话，此时则是通过get_interfaces(task_struct->nsproxy->net_ns)来获取了。

来看下net_ns。这个结构体在include/net/net_namespace.h文件中。下面是其一些属性：

```plain
struct net {
    ...
    //namespace的链表
    struct list_head    list;
    //用来串net_device的链表  
    struct list_head    dev_base_head;
    struct hlist_head   *dev_name_head;
    struct hlist_head   *dev_index_head;
    //每个namespace自己的lo链表
    struct net_device       *loopback_dev;
    ...
};
```

这里的dev_XXX以及loopback_dev会关联所有这个namespace的网络设备。

前面的例子通过ip命令对网络类型的namespace进行操作，下面我们通过ip命令的源码来看下网络namespace的一些实现。当敲下ip netns add XXX后，代码会进入到：

```plain
if (matches(*argv, "add") == 0)
    return netns_add(argc-1, argv+1);
```

这里netns_add负责创建一个网络namespace，实现为：

```plain
static int netns_add(int argc, char **argv)
{
    /* This function creates a new network namespace and
     * a new mount namespace and bind them into a well known
     * location in the filesystem based on the name provided.
     *
     * The mount namespace is created so that any necessary
     * userspace tweaks like remounting /sys, or bind mounting
     * a new /etc/resolv.conf can be shared between uers.
     */
    char netns_path[MAXPATHLEN];
    const char *name;
    int fd;
    int made_netns_run_dir_mount = 0;

    if (argc < 1) {
        fprintf(stderr, "No netns name specified\n");
        return -1;
    }  
    name = argv[0];

    snprintf(netns_path, sizeof(netns_path), "%s/%s", NETNS_RUN_DIR, name);

    if (create_netns_dir())
        return -1;
```

首先这里会在NETNS_RUN_DIR下建立一个目录，NETNS_RUN_DIR的定义为：

#define NETNS_RUN_DIR "/var/run/netns"

比如这个例子，我们可以看到ip命令为我们建立了目录：

```plain
[root@dev ~]# ip netns add X
[root@dev ~]# cd /var/run/netns/
[root@dev netns]# ll
总用量 0
-r--r--r--. 1 root root 0 7月   4 21:11 X
```

此时netns_add还没有为我们真正建立namespace，所以我们继续分析其实现（这里会跳过mnt部分）：

```plain
/* Create the filesystem state */
fd = open(netns_path, O_RDONLY|O_CREAT|O_EXCL, 0);
if (fd < 0) {
    fprintf(stderr, "Cannot create namespace file \"%s\": %s\n",
        netns_path, strerror(errno));
    return -1;
}
close(fd);
if (unshare(CLONE_NEWNET) < 0) {
    fprintf(stderr, "Failed to create a new network namespace \"%s\": %s\n",
        name, strerror(errno));
    goto out_delete;
}
```

注意这里的unshare以及传给它的CLONE_NEWNET，unshare由内核提供，其会建立一个新的网络namespace并将当前进程的task_struct->nsproxy->net_ns指向它。因此从此刻开始这个进程所执行的所有涉及网络相关的命令就都在这个新的namespace中了。至于unshare建立新的namespace会做些什么操作我们下面会看到。

另外这里读者可能会有个疑问，如果unshare会让task_struct->nsproxy->net_ns指向一个新的namespace，那么原来task_struct->nsproxy->net_ns指向的是什么呢？笔者前面说过内核中有一个默认的namespace，如果原来task_struct->nsproxy->net_ns指向的是这个默认的namespace，比如namespace A，那么为什么ip netns list命令看不到这个namespace而只能看到新创建的namespace呢？其实这里是被ip命令欺骗了，其list命令的实现为：

```plain
static int netns_list(int argc, char **argv)
{
    struct dirent *entry;
    DIR *dir;
    int id;

    dir = opendir(NETNS_RUN_DIR);
    if (!dir)
        return 0;

    while ((entry = readdir(dir)) != NULL) {
        if (strcmp(entry->d_name, ".") == 0)
            continue;
        if (strcmp(entry->d_name, "..") == 0)
            continue;
        printf("%s", entry->d_name);
        if (ipnetns_have_nsid()) {
            id = get_netnsid_from_name(entry->d_name);
            if (id >= 0)
                printf(" (id: %d)", id);
        }
        printf("\n");
    }
    closedir(dir);
    return 0;
}
```

可以看到，ip netns list只会列出NETNS_RUN_DIR下的那些namespace，而不会列出内核中所有的namespace。其实内核在启动的时候会建立一个默认的网络namespace DEFAULT_NETNS，如果没有特别操作的话，进程一般都是使用这个DEFAULT_NETNS作为网络的namespace的。也就是task_struct->nsproxy->net_ns一般都是指向DEFAULT_NETNS。

现在我们知道新建立一个网络namespace可以通过unshare来实现。我们来看下一个新的网络namespace的建立代码。查看unshare代码，可以看到关键的切入点为：

```plain
int copy_namespaces(unsigned long flags, struct task_struct *tsk)
{  
    struct nsproxy *old_ns = tsk->nsproxy;
    struct user_namespace *user_ns = task_cred_xxx(tsk, user_ns);
    struct nsproxy *new_ns;

    if (likely(!(flags & (CLONE_NEWNS | CLONE_NEWUTS | CLONE_NEWIPC |
                  CLONE_NEWPID | CLONE_NEWNET)))) {
        get_nsproxy(old_ns);
        return 0;
    }

    if (!ns_capable(user_ns, CAP_SYS_ADMIN))
        return -EPERM;

    /*
     * CLONE_NEWIPC must detach from the undolist: after switching
     * to a new ipc namespace, the semaphore arrays from the old
     * namespace are unreachable.  In clone parlance, CLONE_SYSVSEM
     * means share undolist with parent, so we must forbid using
     * it along with CLONE_NEWIPC.
     */
    if ((flags & (CLONE_NEWIPC | CLONE_SYSVSEM)) ==
        (CLONE_NEWIPC | CLONE_SYSVSEM))
        return -EINVAL;

    new_ns = create_new_namespaces(flags, tsk, user_ns, tsk->fs);
    if (IS_ERR(new_ns))
        return  PTR_ERR(new_ns);

    tsk->nsproxy = new_ns;
    return 0;
}
```

create_new_namespaces建立了新的namespace。来看下涉及网络的部分：

```plain
new_nsp->net_ns = copy_net_ns(flags, user_ns, tsk->nsproxy->net_ns);
if (IS_ERR(new_nsp->net_ns)) {
    err = PTR_ERR(new_nsp->net_ns);
    goto out_net;
}

struct net *copy_net_ns(unsigned long flags,
            struct user_namespace *user_ns, struct net *old_net)
{  
    struct net *net;
    int rv;

    if (!(flags & CLONE_NEWNET))
        return get_net(old_net);

    net = net_alloc();
    if (!net)
        return ERR_PTR(-ENOMEM);

    get_user_ns(user_ns);

    mutex_lock(&net_mutex);
    rv = setup_net(net, user_ns);
    if (rv == 0) {
        rtnl_lock();
        list_add_tail_rcu(&net->list, &net_namespace_list);
        rtnl_unlock();
    }
    mutex_unlock(&net_mutex);
    if (rv < 0) {
        put_user_ns(user_ns);
        net_drop_ns(net);
        return ERR_PTR(rv);
    }
    return net;
}
```

如果CLONE_NEWNET没有设置，那么就用老的网络的namespace（也就是新的进程继承老的进程的namespace，你老的能看到几个网卡我新的同样能看到，因为我们都是指向同一个结构体）。否则则是先调用net_alloc获取个新的net结构体，然后通过setup_net初始化后将它放到内核全局的net_namespace_list链表下。net_namespace_list定义为：

```plain
LIST_HEAD(net_namespace_list);
EXPORT_SYMBOL_GPL(net_namespace_list);
```

对于一个新的网络的namespace，初始化工作主要由setup_net实现。其代码为：

```plain
/*
 * setup_net runs the initializers for the network namespace object.
 */
static __net_init int setup_net(struct net *net, struct user_namespace *user_ns)
{
    /* Must be called with net_mutex held */
    const struct pernet_operations *ops, *saved_ops;
    int error = 0;
    LIST_HEAD(net_exit_list);

    atomic_set(&net->count, 1);
    atomic_set(&net->passive, 1);
    net->dev_base_seq = 1;
    net->user_ns = user_ns;
    idr_init(&net->netns_ids);

    list_for_each_entry(ops, &pernet_list, list) {
        error = ops_init(ops, net);
        if (error < 0)
            goto out_undo;
    }
out:
    return error;

out_undo:
    /* Walk through the list backwards calling the exit functions
     * for the pernet modules whose init functions did not fail.
     */
    list_add(&net->exit_list, &net_exit_list);
    saved_ops = ops;
    list_for_each_entry_continue_reverse(ops, &pernet_list, list)
        ops_exit_list(ops, &net_exit_list);

    ops = saved_ops;
    list_for_each_entry_continue_reverse(ops, &pernet_list, list)
        ops_free_list(ops, &net_exit_list);

    rcu_barrier();
    goto out;
}
```

这里主要是调用了pernet_list的ops来执行各种初始化。内核中一些子系统、模块可以通过register_pernet_device注册一些需要在一个新的namespace建立时执行的函数放在parent_list上，此时一个新的网络namespace被建立的时候就能调用这些函数。比如在net_dev_init这里可以看到如下代码：

```plain
/* The loopback device is special if any other network devices
 * is present in a network namespace the loopback device must
 * be present. Since we now dynamically allocate and free the
 * loopback device ensure this invariant is maintained by
 * keeping the loopback device as the first device on the
 * list of network devices.  Ensuring the loopback devices
 * is the first device that appears and the last network device
 * that disappears.
 */
if (register_pernet_device(&loopback_net_ops))
    goto out;
```

loopback_net_ops会在新的namespace中建立一个lo的net_device，所以我们的namespace中就都能看到lo设备的存在。

需要说明的是虽然上面的分析都是通过进程的task_struct来引到网络的namespace结构体上，但是实际上这个结构体是独立存在在内核中的。比如net_device有一个nd_net指针指向了某个网络的namespace结构体，表示这个net_device属于哪个namespace。

网络namespace的基本实现原理就是上面这些。最后有一个需要注意的就是目前还不是所有网络相关的代码都能感知到namespace的存在。也就是说一些代码目前还没有来得及改成本节开始举例中有ns参数的形式。笔者曾经遇到过一个物理机的ip_forward系统变量设置为1但是namespace中依然为0造成的故障，现在我们知道这个故障的原因是对于ip_forward这个系统变量在网络的namespace中已经支持造成的。如果网络的namespace还不能支持系统变量，那么类似于get_system_var这种函数就还不能传递ns参数，于是所有namespace取到的系统参数应该都是同一个，也就不会存在ip_forward不一致的情况了。那么如何快速简单粗略的定位目前内核的网络namespace代码被哪些模块感知了呢？可以查看网络namespace的net这个结构体的属性，比如net结构体里有sysctl相关的结构体：

```plain
struct netns_ipv4   ipv4;

struct netns_ipv4 {
#ifdef CONFIG_SYSCTL
    struct ctl_table_header *forw_hdr;
    struct ctl_table_header *frags_hdr;
    struct ctl_table_header *ipv4_hdr;
    struct ctl_table_header *route_hdr;
    struct ctl_table_header *xfrm4_hdr;
#endif
```

通过这个我们能快速确定哪些系统变量目前能够感知到网络namespace的存在。

## cgroup

### cgroup简介及例子

在知道namespace是个什么东西后，我们开始来看cgroup。cgroup有什么用呢？简单的说就四个字：限制资源。在前面namespace的网络例子中我们看到通过namespace我们可以做出类似虚拟机的隔离动作，但这种隔离只是对于『name』的隔离，相较于虚拟机的隔离是有一个很大的区别的，这个区别就是资源使用量上的隔离。我们可以指定虚拟机用多少内存多少CPU，但基于namespace的话则搞不定这个。于是乎cgroup就出现了。关于cgroup比较好的参考资料是Redhat的《Red Hat Enterprise Linux 6 Resource Management Guide》。

目前内核支持如下类型的隔离子系统：

| 子系统   | 说明                                                                               |
| -------- | ---------------------------------------------------------------------------------- |
| blkio    | 用于对块设备的IO操作进行速率限制                                                   |
| cpu      | 用于对进程能使用的cpu时间片进行限制                                                |
| cpuacct  | 这个子系统是用于收集cpu的统计信息的                                                |
| cpuset   | 用于CPU及内存绑定，例如限制某个进程使用哪个物理核及哪个物理核的内存                |
| devices  | 用于限制对device的访问                                                             |
| freezer  | 用于启动或停止cgroup中的某个进程                                                   |
| memory   | 用于内存资源的限制，同时负责内存资源统计信息的收集                                 |
| net_cls  | 用于给网络报文打上tag，这个tag会被tc使用用于整流。tc在我们后面的网络章节会进行介绍 |
| net_prio | 用于直接在网络设备上给网络流量设置优先级                                           |
| ns       | 即namespace子系统                                                                  |

下来我们来看下如何使用cgroup进行限制，在使用之前，我们需要了解下cgroup的一些概念：

+ task： 在cgroup就是指的进程或线程，这点和内核是一样的
+ subsystem： 就是上面表格中列出的几类子系统，每种系统各自负责各自的工作，例如cpu子系统负责限制cpu
+ hierarchy： 这个概念比较难理解，大家就认为是个目录结构即可，例如/mem_256m_cg、/cpu_50_cg这类目录，或者是带层次的，例如/mem_256m_cg/mem_128m_cg这类。我们可以对这种hierarchy上加上子系统的限制，例如可以对/mem_256_cg上加上一个内存最多256m的限制。实际使用的时候对于subsystem如何加到hierarchy上会有一些限制，例如同一个子系统不能加载到几个已经加载了其它子系统的hierarchy上。当我们创建一个hierarchy并加载了一个subsystem在这个hierarchy上后，内核中所有的task都会受到这个hierarchy上的subsystem的限制，并且这些task通过fork调用生成的新的task会继承他们parent task的限制，也就是说这些新的task会和他们的父进程处于同一个hierarchy上。

这几个概念中hierarchy的比较难理解，不过不用担心，看了下面的例子就能明白了。类似于我们在namespace中使用的ip命令，内核只是提供了cgroup的编程操作接口，实际上为了方便使用我们还是要借助一些工具命令，例如libcgroup，它自带的cgconfig可以通过配置文件的方式维护hierarchy。但这里我们使用命令的方式直接进行操作。通过mount命令加上echo我们可以创建上面的hierarchy并进行限制。

我们来看个限制cpu的例子（如果读者运行docker的话，docker会自动加载这些子系统，因此建议读者先umount下）：

```plain
[root@dev ~]# mkdir /cgroup/cpu
[root@dev ~]# mount -t cgroup -o cpu cpu /cgroup/cpu
[root@dev ~]# cd /cgroup/cpu/
[root@dev cpu]# ls
cgroup.clone_children  cgroup.procs          cpu.cfs_period_us  cpu.rt_period_us   cpu.shares  notify_on_release  tasks
cgroup.event_control   cgroup.sane_behavior  cpu.cfs_quota_us   cpu.rt_runtime_us  cpu.stat    release_agent
[root@dev cpu]# wc -l tasks
232 tasks
[root@dev cpu]# tail tasks
2468
2470
2487
2489
2517
2519
2521
2524
2525
2532
```

这里我们建立了一个目录/cgroup/cpu，这个目录就是我们的第一个hierarchy。接着通过mount给这个目录（hierarchy）加上了一个cpu子系统，然后进入该目录我们可以看到很多的文件，其中的tasks文件中的每一行代表一个task，表示这些task的cpu资源使用受这个hierarchy上的cpu subsystem限制。

```plain
[root@dev cpu]# mkdir cpu_limited
[root@dev cpu]# cd cpu_limited/
[root@dev cpu_limited]# ls
cgroup.clone_children  cgroup.event_control  cgroup.procs  cpu.cfs_period_us  cpu.cfs_quota_us  cpu.rt_period_us  cpu.rt_runtime_us  cpu.shares  cpu.stat  notify_on_release  tasks
[root@dev cpu_limited]# pwd
/cgroup/cpu/cpu_limited
```

可以看到，如果在hierarchy下建立新的目录，那么这些目录下面会自动的多出很多文件。因此这里的/cgroup/cpu/cpu_limited其实也是一个hierarchy，换句话说在hierarchy通过建立目录的方式我们可以得到子hierarchy。这些子hierarchy默认的资源限制是继承父hierarchy的，即这里的/cgroup/cpu/cpu_limited是继承/cgroup/cpu的，而我们还没有对/cgroup/cpu进行限制，因此/cgroup/cpu/cpu_limited同样没有资源上的限制。

```plain
[root@dev cpu_limited]# mkdir cpu_limited_a
[root@dev cpu_limited]# mkdir cpu_limited_b
[root@dev tmp]# echo 10000 > /cgroup/cpu/cpu_limited/cpu_limited_a/cpu.cfs_quota_us
[root@dev tmp]# echo 100000 > /cgroup/cpu/cpu_limited/cpu_limited_a/cpu.cfs_period_us
[root@dev tmp]# echo 10000 > /cgroup/cpu/cpu_limited/cpu_limited_b/cpu.cfs_quota_us
[root@dev tmp]# echo 20000 > /cgroup/cpu/cpu_limited/cpu_limited_b/cpu.cfs_period_us
```

在/cgroup/cpu/cpu_limited这个hierarchy下我们再建立两个目录，此时我们开始对CPU资源进行限制。这里的限制用于限制某个cgroup下面的task所能使用的cpu时间片的百分比，例如这里的cpu.cfs_quota_us/cpu.cfs_period_us=0.1/0.5则表示可以使用百分之十/百分之五十的CPU。下面我们来做个测试，假如运行下面这个消耗百分百CPU的程序：

```plain
[root@dev tmp]# cat cpu.py
while True:
    pass
[root@dev tmp]# python cpu.py &
[1] 2997
[root@dev tmp]# python cpu.py &
[2] 2998
[root@dev tmp]# top
top - 22:19:43 up 35 min,  2 users,  load average: 0.63, 0.16, 0.09
Tasks: 150 total,   3 running, 147 sleeping,   0 stopped,   0 zombie
%Cpu(s): 47.7 us,  0.2 sy,  0.0 ni, 52.0 id,  0.1 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem :   992568 total,    75196 free,   344020 used,   573352 buff/cache
KiB Swap:  2097148 total,  2096260 free,      888 used.   248316 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                  
 2997 root      20   0  123520   4608   1960 R 100.0  0.5   0:20.66 python                   
 2998 root      20   0  123520   4612   1960 R 100.0  0.5   0:18.50 python
```

我们可以看到这里两个CPU的使用率都是100%，如何通过cpu_limited_a和cpu_limited_b对他们做限制呢？很简单，我们上面讲多hierarchy下的tasks文件代表哪些task受这个hierarchy管理，因此这里我们只要把这两个进程的PID写入到对应目录的tasks文件即可：

```plain
[root@dev tmp]# echo 2997 > /cgroup/cpu/cpu_limited/cpu_limited_a/tasks
[root@dev tmp]# echo 2998 > /cgroup/cpu/cpu_limited/cpu_limited_b/tasks
[root@dev tmp]# top
top - 22:25:45 up 41 min,  2 users,  load average: 1.44, 1.59, 0.79
Tasks: 151 total,   4 running, 147 sleeping,   0 stopped,   0 zombie
%Cpu(s): 10.5 us,  0.2 sy,  0.0 ni, 89.2 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   992568 total,    73844 free,   345188 used,   573536 buff/cache
KiB Swap:  2097148 total,  2096272 free,      876 used.   247000 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                  
 2998 root      20   0  123520   4612   1960 R  49.8  0.5   5:30.58 python                   
 2997 root      20   0  123520   4608   1960 R  10.0  0.5   4:31.71 python
```

可以看到CPU限制生效了，并且满足我们希望的限制要求。对于其它子系统的使用方法也类似，读者感兴趣的话可以参考下文档。

实际的使用过程中如果不使用docker的话对于这里的hierarchy的建立方法会有一些不同的思路，例如是按照每个子系统分目录还是按照组合的方式分目录等等。另外实际使用的时候会存在要被限制的进程异常退出或重启等情况，例如某个mysqld进程是我们要去限制的，但由于一些意外这个进程被kill了，然后又被HA组件启动了，新启动的进程可能不在我们的tasks文件中。这些都需要读者在设计系统的时候考虑到。

## 联合文件系统和chroot

现在读者应该已经知道namespace和cgroup是什么以及大概能做什么，知道基于这两项技术已经可以在物理机上将提供不同服务的进程放到不同的namespace下，并通过cgroup加以限制了。从某种程度上讲这已经类似于虚拟机了，但是在目前的docker的实现中，还有两个关键的技术：联合文件系统和chroot。

虚拟机有个概念叫做快照。快照的意思就是对当前磁盘（也可以包括内存）做一个只读化的操作，这个操作之后所有新写入磁盘（或内存）的数据都会写入到另一个地方。当需要读取这些数据的时候，虚拟机保证你会读取到最新的数据。如果有一天用户希望丢弃这部分新的数据，将整个虚拟机回滚到做快照时候的状态，那么虚拟机就会丢弃那些新的数据，并将磁盘（或内存）的状态变为当初做快照的样子。快照这个概念其实很多地方都用到，例如linux内核fork操作的写时复制、商业存储中的卷复制、版本控制工具（如git）的版本管理等等。一般做快照的目的是为了有一个备份，例如在高端存储及数据库的支持下对于几十个T的数据库进行备份会通过快照的方式。但在虚拟机中快照还有一个常见的作用就是做镜像。例如笔者曾经在虚拟机里装了一套学习TSM带库管理软件的环境，包括虚拟带库等等都装好了，这些安装步骤非常繁琐，因此笔者装好后就做了个快照。后来有同事也想学习TSM，但搭建环境实在太容易出问题（尤其是国内的网络环境实在是不稳定），那么笔者只要把这个虚拟机导出，然后笔者的同事导入虚拟机，并从快照运行虚拟机即可获取到一个可以直接使用的环境。这样即省去了安装操作系统的时间，也省去了下载安装包及安装相关依赖的时间。这种为了某个目的安装好相关环境的可以被其它虚拟机导入直接使用的快照可以称之为镜像。读者如果现在去国内的公有云上申请云主机会发现他们提供了很多镜像给你选择，并且现在还有镜像市场的概念，镜像市场中的镜像和普通的镜像相比前者除了操作系统环境安装完成后还会安装特定的中间件，例如安装了LAMP、Wordpress等等，而后者一般只安装操作系统，并且处于安全的考虑后者会尽量少安装其它软件。如果读者用过镜像并比较下没有镜像直接手工安装操作系统然后安装软件这两个过程的话，读者就会发现使用镜像是非常方便的。

假如目前我们启动了一个进程，并让这个进程运行在独立的namespace中并加了cgroup限制，那么如何让这个进程看到的文件系统和宿主机不同呢？这个可以通过chroot命令实现。chroot顾名思义就是『改变根目录』的意思，它是Linux很早就提供的一个功能，用于修改某个进程所看到的根文件系统路径。比如在进程A的执行代码中执行类似chroot /tmp/a的命令，则在此之后A执行ls /命令看到的就是/tmp/a下面的内容了。类比我们上面说的虚拟机镜像，对于仅针对磁盘的镜像来说我们可以通过chroot来做磁盘镜像。方法很简单：首先我们启动一个拥有独立namespace的进程并运行bash，chroot它的根目录到某个路径，例如/tmp/base，然后我们将常见的工具复制进来，并在其中安装我们需要的软件。这样在宿主机的/tmp/bash下就有了我们完整的磁盘镜像文件。当我们启动一个其他进程想要使用这个磁盘镜像文件时，我们只需要复制一份该目录，例如复制到/tmp/processa，然后启动一个新的进程，将其chroot到/tmp/processa即可。此时对于这个新启动的进程其看到的文件系统上的文件就和我们之前做base的进程看到的一样。也就是说通过chroot我们就能实现类似虚拟机镜像的功能，但这种方式有一些问题，例如如果按照这种方式，对于一百个进程就需要复制一百份/tmp/base，但我们知道这个目录中的大部分文件是只读的，例如base安装了mysql，那么mysql的执行文件其实是只读的，它对于这一百个进程都是一样的，如果复制了一百份的话占用磁盘空间不说，还会影响进程启动的速度，因为进程启动前需要拷贝这些文件。为了解决这个问题就需要用到『联合文件系统』，如果读者类比上面的虚拟机的镜像的话大概就能猜测出来，『联合文件系统』需要实现的功能就是上面『快照』的『写时复制』功能。目前常见的『联合文件系统』主要有AUFS和Device Mapper，前者有Ubuntu社区在推动，后者有Redhat推动，但其功能远不止『联合文件系统』一项。除此之外还有OverlayFS等。基于『联合文件系统』，我们启动进程的时候不需要复制一份/tmp/base，而是只要以/tmp/base文件只读的根目录，在其之上建立『一层』可写层即可。当进程没有在文件系统中写入数据的时候物理机上的磁盘空间占用就是一个/tmp/base，即便有一百个chroot的进程也同样如此。当进程要写入数据的时候，『联合文件系统』会将数据写入到可写层中，保证这部分新的数据不会影响到其它使用/tmp/base的进程。

在docker中，基于『联合文件系统』这种启动迅速又省空间的技术，就有了docker自己的镜像。同时docker公司还提供了一个『Docker hub』的网站，类似于github，用户可以把自己的镜像上传到这个网站上（这里的镜像其实就类似于上面的/tmp/base打一个tar包），然后其他人可以基于别人的镜像启动自己的容器。并且『联合文件系统』是可以分多层的，例如A用户基于B用户的MySQL镜像(例如/tmp/mysql)启动了容器，然后在这个容器中安装了Apache并打包了一个镜像(例如/tmp/apache--based-on-->/tmp/mysql )并上传到了docker hub中，此后C用户可以下载这个安装了apache和mysql的镜像并在里边安装wordpress(即/tmp/wordpress--based-on-->/tmp/apache--based-on-->/tmp/mysql)。通过这种方式使用docker的用户可以很快的基于别人的模板镜像运行自己的服务。但这里也有一些问题，例如别人的镜像是否安全，会不会装有后门等等，另外随着镜像层次的增加性能上的额外开销是否可以承受等等都是需要考虑的问题，尤其是对于生产环境的系统更是如此。

## docker

我们来看下docker，这个如今最流行的容器产品。docker所依赖的主要技术笔者在前面都已经介绍了，因此本节主要是介绍一下docker中的概念及主要的一些使用命令。

首先看下docker中的几个概念：

+ 镜像：镜像就是我们上面讲的基于『联合文件系统』实现的类似虚拟机镜像或快照一样的东西。本质上镜像就是一个tar包，tar包包含的内容主要是使用『联合文件系统』的实现安装好某些应用及工具的目录，同时还包括了一些元数据信息用于docker在实际运行的时候能解析这个tar包并获得足够的信息。
+ 容器：容器即实际上是一个运行时环境。读者已经知道镜像是包含了各种应用及工具的一些文件及目录，例如包含了mysql的可执行文件，但我们实际上使用的时候需要的是一个运行着的mysql，于是按照之前namespace中讲的那样，我们启动一个进程，将这个进程加入到某个namespace中，并chroot我们的镜像文件目录作为这个进程的根目录，然后加上一些cgroup限制，此时这个进程及这个进程所处的运行时环境就称之为容器。关于容器我们有启动、停止、杀死、删除之类的说法。由于我们知道这个容器主要就是一个进程，因此对于启动、停止、杀死都比较好理解，就是对进程的启动、停止、杀死。那么删除是什么意思呢？这里的删除主要还是指的元数据，docker会保存一些关于其运行的容器的信息，例如这个容器挂载了哪些目录，启动的命令是什么之类的，当我们杀死了容器的进程后，这些容器的信息依旧保存着，因此删除一个容器即删除这些信息。
+ 仓库：即存放镜像的地方。仓库分为共有仓库和私有仓库，可以类比共有的github及私有的gitlab。对于共有的仓库主要是hub.docker.com这个网站，大家可以上传自己的镜像供别人下载使用，也可以下载别人上传的镜像。对于私有仓库则是一般企业内部处于安全或标准化的原因建立的内部仓库。
+ Dockerfile：当我们下载了别人的镜像后，我们可能会在这个镜像的基础上在安装我们需要的工具，例如下载了一个mysql的镜像，接着我们以这个镜像启动一个容器，并在这个容器中运行yum install nginx安装nginx，然后对nginx进行各种配置。但如果我们意外的删除了这个容器，这些东西就都丢了。此时我们主要有两种方法可以避免这种情况的出现， 一种是导出我们的容器到一个镜像，简单的说就是将容器运行后的叠加在『联合文件系统』上的写入的东西作为一层『联合文件系统』的数据并生成一个新的镜像，之后从这个镜像启动容器即可。另一种则是基于Dockerfile。大家可以认为Dockerfile就是一个启动脚本，类似于linux中init.d中的那些文件，它的作用简单的理解就是当启动一个容器的时候，先启动一个进程并chroot我们的镜像，然后执行Dockerfile中的命令，在这里Dockerfile中就得写入yum install nginx等命令用于安装和配置nginx。基于Docker的好处有很多，例如我们这里的例子，如果我们将我们的安装了nginx的容器导出的镜像上传到hub.docker.com中想分享给大家，其他人虽然很想用，但他们可能担心你有没有装什么后门。虽然你的基础镜像mysql是受认证的，但你基于这个镜像做的操作他们并不清楚。但如果提供Dockerfile的话他们一看就知道你基于这个受认证的mysql镜像做了什么，因此很容易就能接受你的镜像。除此之外基于Dockerfile我们可以很方便的搭建基于容器的开发及部署方案，这一点在我们后续的章节会进行介绍。
+ daemon：docker的守护进程，即docker的主服务进程。在docker中主服务进程和向docker发送命令的『client』都是通过『docker』这个可执行文件进行的，区别在于如果是要启动daemon，则需要用-d参数，否则默认是以『client』的角色向daemon发送命令。一般是通过unix本地套接字发送命令的，当然也可以主动的让daemon监听tcp端口走tcp协议进行交互。值得注意的是docker daemon与client之间是走的http协议（即便是基于unix套接字，应用层依旧是http协议），因此我们可以通过curl等方式与daemon进行交互，进行启停容器等操作。提供HTTP的接口这个是在公有云产生后（尤其是AWS出现后）的软件与这之前的软件（如nginx、mysql）的巨大区别之一。而且大部分提供HTTP的软件都遵循我们上面讲过的RESTful风格。

概念讲完了，我们来看下docker的常见用法吧。这里我们以一个启动centos容器的例子为例：

```plain
[root@dev ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@dev ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
centos              6                   3bbbf0aca359        8 months ago        190.6 MB
[root@dev ~]# docker run -i -t --name centos-6-bingotree centos:6 /bin/bash
[root@2bb73d6a1507 /]#
[root@2bb73d6a1507 /]# hostname
2bb73d6a1507
[root@2bb73d6a1507 /]# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:01  
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:1/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 b)  TX bytes:738 (738.0 b)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
[root@2bb73d6a1507 /]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.42.1     0.0.0.0         UG    0      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 eth0
# 从容器中detach可以通过：ctrl+p ctrl+q实现
[root@dev ~]#
[root@dev ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
2bb73d6a1507        centos:6            "/bin/bash"         About a minute ago   Up About a minute                       centos-6-bingotree   
[root@dev ~]# docker stop 2bb73d6a1507
2bb73d6a1507
[root@dev ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
2bb73d6a1507        centos:6            "/bin/bash"         2 minutes ago       Exited (137) 5 seconds ago                       centos-6-bingotree    
[root@dev ~]# docker rm 2bb73d6a1507
2bb73d6a1507
[root@dev ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
[root@dev ~]# docker run -i -t --name centos-6-bingotree centos:6 /bin/bash
[root@b3477f79d8f1 /]# [root@dev ~]#
[root@dev ~]# docker exec -i -t centos-6-bingotree /bin/bash
[root@b3477f79d8f1 /]#
```

上面的例子就是常见的容器操作。读者可以注意下这里的exec，在排错的时候经常会用到这个命令，它的原理很简单：执行了docker exec后会再次启动一个进程，然后把这个进程放到和我们的目标容器相同的namespace等运行时环境中，这样我们的新进程就好像和我们的目标容器出在同一个环境中了。对于Dockerfile及镜像的生成、上传、下载等操作我们在后面讲持续集成/持续部署的时候会给大家看相关的例子。

## kubernetes

在以docker为代表的容器出现后，企业对软件部署的方式有了一种新的选择，尤其是对规模很大的公司更是如此。以往部署应用会有很多问题，例如依赖问题、语言问题等等，但基于容器的话这些问题能有很大的改善。例如有些公司可能内部有java应用、go应用、python应用等等，运维团队要维护这些应用是很困难的，可能主机上缺少了这个jar包那个jar包，也可能主机上应用依赖的rpm包版本存在冲突。对于上规模的企业来说一般的解决方法就是把应用部署在虚拟机中减少依赖包带来的影响，但虚拟机的启动速度、资源开销都令人头疼，而容器则相比较虚拟机来说既可以解决这些问题，同时也能做到快速启动及消耗较少的额外资源。但正如对虚拟机的管理一样，当容器的个数到达一定规模的时候，依靠容器自身提供的命令行工具是很难对容器集群进行管理的。举个简单的例子，如果企业需要启动1000个容器用于提供web service服务，那么通过docker的命令行接口手工去启动这些容器是不现实的，必然要依靠脚本启动。虽然通过脚本启动解决了『启动』这个问题，但实际情况中物理机的物理故障发生概率并不低，那么当物理机宕机了它上面的容器如何做迁移？当业务代码迭代版本要进行更新的时候如何进行灰度的更新？通过脚本很难处理这些问题。脚本可以解决『量』的问题，但无法提供『质』的功能，因为如果要提供『质』的功能，脚本必然非常复杂，而复杂的脚本在维护性、可读性等方面不如专门的高级语言编写的管理平台来的好。而kubernetes就是这类高级语言编写的管理平台中的一个。

kubernetes由google研发并对外开发源码，其设计思想参考了google内部使用了很久的borg集群管理系统(见论文[Large-scale cluster management at Google with Borg](https://research.google.com/pubs/pub43438.html))。通俗的讲它就是一个容器的管理平台，我们可以通过kubernetes管理我们的容器集群，并且kubernetes会自动的帮助我们保证容器服务的『实际状态』满足我们的『期望状态』。『期望状态』打个比方就是我们期望在我们的集群中有1000个提供认证服务的容器，并且这些容器分布在不同的主机上，则『实际状态』就是目前实际上在我们的物理主机集群中这些容器是否达到了我们期望的数目1000，以及实际上这些运行着的容器是否按照我们的期望运行在不同的物理主机上。让『实际状态』和『期望状态』一致是kubernetes的核心功能，除此之外它还支持集群中容器应用的健康检查、回滚升级、负载均衡、资源监控、横向扩展等功能。

kubernetes中主要有如下的概念：

+ Cluster:运行kubernetes软件的物理机集群（也可以是虚拟机集群），kubernetes管理的容器应用就跑在Cluster中
+ Node:集群中的物理机或者虚拟机。Node是调度器调度容器运行可以在哪里运行的目标
+ Pod:一组存在逻辑相关的容器及资源的集合。例如一个基于LAMP的应用就可以看做一个Pod，这个Pod可以包含一个MySQL容器和一个运行PHP+Apache的容器。
+ Label:Label就是KV对(键值对)形式的一种说明，例如我们可以给一个Pod打上一个Label，内容可以是priority:100，用于说明这个Pod的优先级。Label在调度的时候有重要的作用
+ Selector:用于匹配Label的一种表达式，例如一个selector可以类似为priority=100这种新式，即匹配含有优先级为100的Label的Pod
+ Replication Controller:用于维护我们的『目标状态』和『期望状态』一致的一个服务
+ Service:用于对外提供访问接口，例如给我们的Pod提供一个外网IP或者是DNS供外界访问
+ Volume:即存放数据的卷
+ Secret:用于存放敏感信息
+ Name:用于标示资源
+ Namespace:用于让不同的team共享一个集群而不会出现name冲突之类的情况，简单的说就是做了名字隔离
+ Annotation:类似于Label，但Label是由人提供的，并且对人是可以阅读的，而Annotation则是由工具或系统扩展提供的

从构成的组件来说，kubernetes主要有下面几种组件：

+ kube-apiserver:提供RESTful的接口，用户通过这些接口管理集群
+ kube-proxy:用于提供网络服务
+ kube-scheduler:用于提供调度
+ kubelete:运行在Node上，用于将『期望状态』同步到本Node上
+ kube-controller-manager:用于提供管控服务，例如提供Replication Controller

在实际的使用中，大部分功能主要通过kubectl这个命令行工具实现，但目前kubernetes也提供了官网的UI页面供集群管理。

在本身后面的章节中笔者会分析每一章对应的功能在kubernetes中的实现，到时候读者会对上面的『概念』及『组件』有更深入的理解。

现在我们来看一个例子，这个例子使用了基于OpenVSwitch的多节点网络环境，搭建方法可以参考本书的附录。为了使用kubernetes，我们先要让集群知道它有哪些Node可以被使用，因此我们先定义Nodes：

```plain
[root@k18s01 ~]# cat node01.json
{
    "apiVersion": "v1",
    "kind": "Node",
    "metadata": {
        "name": "k18s02",
        "labels":{ "name": "kub-node-label"}
    },
    "spec": {
        "externalID": "k18s02"
    }
}
[root@k18s01 ~]# cat node02.json
{
    "apiVersion": "v1",
    "kind": "Node",
    "metadata": {
        "name": "k18s03",
        "labels":{ "name": "kub-node-label"}
    },
    "spec": {
        "externalID": "k18s03"
    }
}
[root@k18s01 ~]# kubectl create -f ./node01.json
node "k18s02" created
[root@k18s01 ~]# kubectl create -f ./node02.json
node "k18s03" created
[root@k18s01 ~]# kubectl get nodes
NAME      LABELS                          STATUS
k18s01    kubernetes.io/hostname=k18s01   Ready
k18s02    name=kub-node-label             Ready
k18s03    name=kub-node-label             Ready
```

接着我们建立rc及service，rc即Replication Controller，用于说明我们的Pod存在几份副本及说明Pod的内容，而service则用于说明我们的Pod如何对外提供服务映射：

```plain
[root@k18s01 ~]# cat nginx-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-controller
spec:
  replicas: 1
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
[root@k18s01 ~]# cat nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
  - port: 8000
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
[root@k18s01 ~]# kubectl create -f ./nginx-rc.yaml
replicationcontroller "nginx-controller" created
[root@k18s01 ~]# kubectl get rc
CONTROLLER         CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS
nginx-controller   nginx          nginx      app=nginx   1
[root@k18s01 ~]# kubectl get po
NAME                     READY     STATUS    RESTARTS   AGE
nginx-controller-zfd64   1/1       Running   0          25s
[root@k18s01 ~]# kubectl create -f ./nginx-service.yaml
service "nginx-service" created
[root@k18s01 ~]# kubectl get service
NAME            LABELS                                    SELECTOR    IP(S)           PORT(S)
kubernetes      component=apiserver,provider=kubernetes   <none>      10.254.0.1      443/TCP
nginx-service   <none>                                    app=nginx   10.254.85.157   8000/TCP
# k18s02上
[root@k18s02 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@k18s02 ~]#

# k18s03上
[root@k18s03 ~]# docker ps
CONTAINER ID        IMAGE                                  COMMAND                CREATED             STATUS              PORTS               NAMES
9ea0c512a8cb        nginx                                  "nginx -g 'daemon of   4 minutes ago       Up 4 minutes                            k8s_nginx.6420169f_nginx-controller-zfd64_default_eab9db09-40ca-11e5-b237-fa163ee101cd_f4ab7b51  
e4fb6592bfee        gcr.io/google_containers/pause:0.8.0   "/pause"               4 minutes ago       Up 4 minutes                            k8s_POD.ef28e851_nginx-controller-zfd64_default_eab9db09-40ca-11e5-b237-fa163ee101cd_19ed5eed
```

## 容器与虚拟化

这里的虚拟化主要指虚拟机。笔者有幸经历了容器的从无到有的过程。在笔者刚工作的时候（2012年左右）大部分的公司还是用着IOE的解决方案，也就是使用IBM、Oracle、EMC、HP等公司的软硬件产品。这些产品中并没有容器这么一个概念，有的只是虚拟化。那时的虚拟化主要是以EMC旗下的VMware公司的EXSi方案为主，也有少部分企业会上KVM的方案，而银行等财大气粗的企业则会使用IBM的Power虚拟化技术。后来在2013年底2014年初的时候OpenStack开始火了起来，出现了大量的创业公司，原因从笔者的实际经历来讲一方面是当时红帽对OpenStack下了堵住，押宝其能和linux一样成为其下一个公司的主要关注对象，而其他公司也希望能有一个新的东西支撑其股价，另一方面则是OpenStack在那时渐渐的开始成熟，并在一定程度上能替代商业化的ESXi方案，为企业节省不少的成本，加上那时淘宝的去IOE运动让开源软件进入了很多企业CTO的视线，除此之外AWS的成功让大家都将注意力转向了『云』这个概念，而作为『事实上私有云标准』的OpenStack则自然的因为『云』概念的风靡而名声鹊起。但从实际的情况来看到目前为止OpenStack的表现只能说是中规中矩，这一点和SDN很像。但在那时无论是AWS、OpenStack还是ESXi，他们主要的管理对象是虚拟机，但在这之后在2014年底2015年初的时候docker就非常火爆了，docker的优点在前面已经讲过了，事实上确实基于docker可以在某些情况的业务场景中取代虚拟机，因此那时有很多说虚拟机已死的文章在网上出现。那容器和虚拟化对于企业来说到底应该如何取舍呢？

笔者认为，无论是容器还是虚拟化，它们都只是工具，具体如何使用需要根据业务场景来判断。这句话有些不负责任，因为这并没有告诉大家怎么去判断业务场景。就笔者目前的经验来说，如果业务上需要的是一台『机器』，那么就使用虚拟化，如果业务上需要的是一个『服务』，那么就用容器。实际上对于生产环境基本上需要的都是『容器』，因为生产环境是对外提供服务用的，没人会登陆到具体的生产服务器上去再安装个啥啥啥软件或开个vi写点代码啥的。但对于测试环境、开发环境等一般需要的都是虚拟机，因为开发人员和测试人员会经常登陆上上去，安装一些他们需要的调试软件，或者有些开发想要尝尝新，例如想试着学下函数式编程，那么对于开发来说自己从头安装一下开发环境则是个很好的选择。除了从业务角度出发外，虚拟机相比较于容器来说其出现的时间较长，也较为稳定（简单的说就是bug少），而容器的技术栈依旧在高速的迭代更新中，因此不可避免的在实际使用的时候会出现一些坑，因此如果团队的运维能力较弱，那么使用虚拟机比较可取。另外一些软件目前很难通过容器做成服务，例如关系型数据库，虽然可以使用mysql的镜像启动一个数据库服务，但这个服务没有高可用，并且对于企业来说数据库服务一般为了保证性能是独占一台极高配置的主机的，从这一点来说容器并不合适，甚至虚拟机也不合适。所以在实际情况中具体是使用容器还是虚拟化需要考虑这些实际情况，不能一概而论。另外目前很多业务场景下单独使用虚拟化或者单独使用容器都不合适，因为容器虽然启动停止很方便，但隔离性不好，且一般企业的CI系统、CMDB系统不支持容器，而虚拟化虽然在企业中有很多工具衔接良好，且隔离性强，但却有点『重』，因此在虚拟机中运行容器是个很常见的用法。比如多个不同的业务使用一个集群，为了保证这些业务不互相影响，可以给各个业务分配虚拟机，而后业务可以在各自的虚拟机集群中建立容器集群。因此无论是容器还是虚拟化，它们只是业务前进道路上的垫脚石。就如本书开头说的，对于运维来说，保证业务的持续健康运行是才根本目标。
