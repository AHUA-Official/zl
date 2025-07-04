---
title:  LinuxZL-虚拟化-VM
date: 2024-12-15 00:00:00
tags: [Linux笔记]
---
[https://github.com/yifengyou/learn-kvm](https://github.com/yifengyou/learn-kvm)    笔记

[https://github.com/xinliangnote/Go](https://github.com/xinliangnote/Go)     go入门

[https://github.com/zeromicro/go-zero](https://github.com/zeromicro/go-zero)

qemu  virsh

#### 监控 vcpu和pcpu的映射关系

 virsh list --all

![](https://cdn.nlark.com/yuque/0/2024/png/39116304/1716881407306-e6827ad4-0d5b-4f62-a37b-6bd61c36bc4f.png)

拟化原理

虚拟化技术大家想必都听说过，如果读者从事的就是运维或者在某个公有云服务商用过其服务的话那大家肯定对此并不陌生。虚拟化，简单的说就是对于某一个独立只能供单个用户使用的资源，将其虚拟成多份资源供多个用户使用。例如主机虚拟化就是将一台物理机通过虚拟化技术虚拟出多台虚拟机，以此类推存储虚拟化就是讲某个存储资源通过虚拟化技术虚拟成多个存储资源，网络虚拟化就是将某个网络资源虚拟成多个网络资源。

本节我们主要介绍服务器虚拟化的原理，因为大家接触的最多的就是服务器虚拟化，后续章节我们会陆续接触到存储虚拟化以及网络虚拟化。

在没有虚拟化的时候，应用所在的操作系统直接运行于物理硬件上，当使用虚拟化技术后，这种模型可以看成是操作系统运行在虚拟化层上，而虚拟化层直接运行在物理硬件上。这层虚拟化层有些文档称之为虚拟机监视器(Virtual Machine Monitor, VMM)，或者直接简单的称之为Hypervisor。在使用了虚拟化技术后，一般把物理机称为宿主机，而虚拟机称为客户机。

如何实现虚拟化即如何实现上面的VMM。目前的实现方式一般有两种：软件虚拟化和硬件虚拟化。

软件虚拟化的代表是QEMU，其原理比较简单，通过纯软件的方式，软件虚拟化会仿真一套其需要模拟的硬件平台的指令集，然后应用（这里的应用包括操作系统）在运行的时候，原来由硬件CPU执行的指令则被软件虚拟化软件读取，然后由其进行实际的执行。举个简单的例子，如果某个应用要执行1+1这行代码，其在X86上的汇编指令类似下面这个样子：

MOV AX 1

MOV BX 1

ADD AX BX

如果没有使用虚拟化，那么硬件CPU就会读取并执行这三行指令（当然实际读取的当然是二进制格式的数字信号）。如果使用虚拟化，假如我们用Python实现一个软件虚拟机，那么虚拟机的代码可能如下：

ax = 0

bx = 0

app = read_app_code("one_plus_one.exe")

while cmd = app.get_cmd() is not None:

    if cmd == 'MOV AX 1':

    ax = 1

    elif cmd == 'MOV BX 1':

    bx = 1

    elif cmd == 'ADD AX BX':

    ax = ax + bx

然后实际执行的时候，这段Python代码才是真正运行在物理CPU上的被执行代码，而我们的1+1的应用则是以二进制的方式被我们的软件虚拟机读取而已。

从这里的描述可以看出，软件虚拟化的性能肯定会成为一个问题。原本1+1只需要三个指令即可完成，如果用我们上面的Python软件虚拟机执行的话，实际生产的汇编指令数目估计3000都不止。那软件虚拟化的好处是什么呢？就笔者而言，笔者认为其最有用的用处是其可以在某一个硬件上运行不同指令集的应用。例如笔者以前工作的时候使用的是IBM的AIX操作系统，那个时候想要学习AIX是件很麻烦的事情，因为AIX运行在IBM的POWER芯片上，其指令集和我们日常使用的Intel的X86指令集不同，因此不能将这个操作系统安装在笔记本上来进行学习，只能在POWER服务器上进行安装。这时通过软件虚拟化，可以用软件虚拟机模拟POWER的指令集，然后将其翻译成X86的指令，运行在X86上，这样AIX操作系统就能运行在笔者的笔记本上的。当然通过软件模拟还是存在很多缺陷的，但对于日常的学习使用是足够了。软件虚拟化的还有一个好处是方便日常学习，例如笔者在学习OpenStack的时候需要三台主机进行学习，此时就可以通过软件虚拟化在笔记本上虚拟三台主机出来进行学习。

另外除了这种最简单的软件虚拟化技术外，类似于VMWare的软件虚拟化技术则使用了动态二进制翻译技术。其原理是软件虚拟机在执行被虚拟的指令时，会先扫描一遍指令，看一下这些指令直接运行在物理机上有没有问题，如果没有问题则直接运行，否则则通过上面的方式进行翻译。举个简单的例子，如果我们新建了一台虚拟机，并给它配备一个虚拟化的光驱，此时对于1+1这类指令软件虚拟机可以让其直接运行在物理机上，但对于涉及到虚拟光驱的使用，由于我们的物理设备已经不带光驱了，因此虚拟机必须把这类涉及到虚拟光驱的指令进行翻译，将原来读取光盘的指令，转为读取硬盘上某个文件的指令。通过这种方式其效率会比纯粹的软件虚拟化要高，但是由于其指令可以直接运行在物理硬件上，其也就失去了跨平台的能力。

硬件虚拟化和软件虚拟化不同，硬件虚拟化的宿主机硬件需要直接提供对虚拟化的支持。在现在云环境的大浪潮下，很多硬件厂商的硬件甚至为了虚拟化提供了额外的硬件资源。要理解硬件虚拟化的原理需要理解现有操作系统的在CPU上的运行模式。笔者这里还是举一个简单的例子：在Linux操作系统中有内核态和用户态两种概念，这两者的区别是用户态运行在CPU的ring 3级别，而内核态运行在ring 0级别。这里的ring 0和ring 3有什么区别呢？打个比方，运行在ring 0的应用可以执行关机命令，而运行在ring 3的应用则只能执行1+1这种不需要什么特权的命令。大家可以把CPU想象成住在一个四层高的小楼的一家人，楼层越低的人在家里的权限越大。四楼的人可能可以有权限决定今晚买什么菜吃，但如果要买个汽车啥的大件那么就需要一层楼的人的同意才行。一般情况下四楼的人活的自由自在，想买啥就买啥，几乎意识不到还有一楼的人管着他，但只要四楼的人一买大件那么一楼的人就会参与进来。熟悉Linux的人都知道用户态的代码在某些时候会调用到内核态代码，例如通过系统调用，或者发生trap的时候。简单的理解就是用户态的代码在执行某些指令，或者访问了不该访问的内存等情况下会切入到内核态中。用我们的例子就是四楼的人想买辆车，于是刷了家里的信用卡，然后一楼的人收到了短信提醒，这样一楼就参与到这个事情上来了，可能一楼的人设置了消费提醒，大额消费的时候银行必须电话一楼的人才能消费成功。说了这么多那么和我们的硬件虚拟化有什么关系呢？其实关系很简单，硬件虚拟化实现的最重要的途径就是这种不同ring级别的切换。硬件虚拟化的CPU在原有的ring级别上提供了额外的模式，例如Guest模式。客户端的代码现在运行在了Guest级别上而不是原来的ring 3或ring 0上。和上面说的VMWare的软件虚拟化一样，对于大部分可以直接运行的指令，客户端的代码可以直接在物理硬件上执行，但是对于少部分指令，例如虚拟内存的访问则会触发trap（也就是我们例子中的四楼人刷了家里的信用卡，这里的trap可以认为就是一楼的人收到了短信并采取操作），触发trap后，物理硬件就会接手接下来的事情，也就是把接下来的指令交由VMM进行执行（例如一楼的人和银行电话确认这笔交易的合法性），VMM在解决完这些事情后就会把执行流回到之前发生trap的地方，让原来的客户端代码继续执行（这时四楼就能去提车了）。Intel处理器中的Virtualization Technology（IntelVT）就是这样的技术。可以看到相比较软件虚拟化，硬件虚拟化由于有硬件的直接支持因此性能上会好于软件虚拟化，但相比较没有虚拟化的直接运行模式来说，性能还是有开销的。最直接的开销就是内存的寻址操作。例如应用本来要向内存地址0x1000处写入一个值，此时宿主机上虚拟了10台虚拟机，每台虚拟机上的某个应用都有这么一个请求，但实际上的物理内存就一个，因此VMM需要对这里的内存地址进行映射，例如对于第一台虚拟机，其看到的内存地址0x1000对应实际内存地址的0x2000，对于第二台虚拟机其看到的内存地址0x1000对于实际内存地址的0x3000。可以想象在代码中这里涉及内存地址的指令是非常多的，因此就这一块的开销就不可忽略。不过目前硬件虚拟化也提供了专门的解决方案处理这块开销。

大家可能会问为什么CPU要提供不同的ring级别，用户态和内核态都运行在一个级别不行吗？实际上这样的话会有很大的安全问题。例如有个木马程序想要获取你输入在浏览器中的账户密码，此时这些密码都在内存中。木马是个用户态程序，其运行在ring 3中，如果其去访问了浏览器进程的内存地址，这个时候CPU会判断下其有没有权限访问，CPU发现其处于ring 3，不是ring 0这种特权级别，因此就直接出发内存越界访问的错误。

除了硬件虚拟化和软件虚拟化外，大家可能还会听说准虚拟化和全虚拟化。由于软件虚拟化存在性能问题，一种解决性能问题的方法就是准虚拟化，其通过对客户机操作系统进行代码层面的修改，使得客户机操作系统意识到其是运行在虚拟化环境中的。此时该客户机操作系统在虚拟机中运行的时候虚拟机软件不需要事先扫描其指令集，直接在物理硬件上执行即可，因为此时的客户机操作系统对于那些需要虚拟机翻译的代码其已经事先进行了操作，其会主动通过虚拟机什么时候要其参与。在这种情况下准虚拟化的性能相比软件虚拟化会有提高，但是需要对客户机操作系统进行修改。与准虚拟化对于的就是全虚拟化了，其不需要对客户机进行修改，并提供了完整的虚拟化硬件，包括虚拟的CPU、虚拟的内存等等，总之只要这个客户机操作系统可以运行在物理硬件上，其就能不加修改直接运行在全虚拟化虚拟机上。不过相信大家在目前的实际生产环境中接触的都是硬件虚拟化了。

近几年来随着大家对容器热情的持续升温，其实按照笔者对于虚拟化的理解，容器也可以看成是一种虚拟化，只不过其虚拟的是应用的执行环境。原本一台物理机上可能只能提供一个应用的执行环境（例如一套LAMP），现在则可以提供多套环境。但容器实现的原理和上午提到的都不同，这个待我们容器的那一节再专门叙述。

虚拟化产品

目前比较常见的虚拟化产品主要有：

Citrix公司的Xen

Microsoft公司的Hyper-V

Oracle公司的Virtual Box

Redhat公司的KVM

VMWare公司的ESX/ESXi

VMWare公司的VMWare workstation

个人用户使用虚拟化产品的时候一般都是使用VMWare workstation或者是Virtual Box的居多。在笔者所见到的企业里，使用ESX Server的企业最多，其次是Hyper-V、KVM。而Xen则在国内某些大型的公有云提供商中有见过使用。另外由于KVM其在成本上面的优势，加之其是开源软件，符合我国的国家安全方向，因此国内很多有实力的厂商在自己搭建私有云环境的时候都选择以KVM为主，逐步的替换他们原先的ESX/ESXi方案。虽然最近一段时间基于Docker做私有云的企业逐渐增多，但是基于KVM、Xen做私有云也有其自己的优势，例如目前很多银行等大型机构都需要提供『云桌面』给自己的员工，『云桌面』简单的说可以看成是Windows的远程桌面，只不过公司员工通过一台低配置的PC通过网络连接到公司的服务器上远程登录进行访问。在这样的场景下企业可以通过归集效应节约IT成本，对于员工来说也可以随时随地的访问自己的工作资料，且不用担心PC坏了造成数据丢失，因此是有很大的好处的。但Docker这类适用于应用封装的容器方案就不是很适合这样的场景。

由于KVM具有开源、产品本身免费等优势，并且目前企业在做私有云规划的时候基本都是考虑基于KVM或Docker进行设计，因此下面重点介绍下KVM。

KVM

听过KVM的人一般也会听过QEMU、内核模块等相关的术语，这里就来先介绍一下KVM、QEMU和内核模块的关系。

首先来看内核模块。在目前的操作系统中有两种主流的设计模式：微内核和宏内核。微内核的代表就是Windows操作系统，在微内核中，操作系统内核的功能是通过一个一个进程实现的，而微在这里指的就是实现了最核心功能的那个进程。换句话说如果你不需要操作系统提供额外的功能，那么只要运行微内核的那个进程就行了。这种模式的好处显而易见，如果内核需要新增一个功能，那么再启动一个进程就好了，如果某个功能出现了安全隐患，那么停止掉这个进程即可。目前一般的操作系统书籍中介绍如何编写操作系统的时候都是以微内核为例进行介绍的，因为其代码的组织结构比较简单。但微内核的缺点也比较明显，那就是进程之间的调用会造成比纯粹的函数调用开销要来的大的损耗。而宏内核则没有这个问题，宏内核将所有的代码都编译在一个大的文件中，此时操作系统运行的时候就加载这个大文件，内核中不同的功能要互相调用的时候就是纯粹的函数调用，因此效率要比宏内核好一些，但缺点也很明显，那就是代码的模块化不好，如果某个组件出了问题则在修复代码后需要重新编译整个文件并重新加载这个问题才能修复。宏内核的代表就是Linux。但Linux在宏内核的基础上引入了模块化和动态模块加载的能力。可以简单的认为这里的模块就是一个动态链接库。Linux内核最核心的零件是一个独立的模块，而内核的其它功能则是一个一个独立的模块，内核可以更具需要加载特定的模块，而模块加载后并不是和微内核一样以进程的方式独立运行，而是通过动态链接的方式直接被其它模块通过地址的方式调用。此时如果有人新开发了一个功能想要把他加入到内核中，则他只需要按照模块的编写要求写一个模块，然后使用命令加载这个模块进入到内核中即可。这个过程完全可以不需要内核重启。如果某个模块发现存在问题，则同样的通过命令卸载这个模块即可。运维人员接触的最多的模块可能就是文件系统了，EXT3、EXT4等等文件系统都是以内核模块的形式实现的，当想要试验一个新的文件系统时，通过模块操作就可以让内核支持这个文件系统了。而KVM其实就是一个内核的模块。当用户加载了KVM模块后，该模块的初始化代码会的初始化相关的数据结构，同时会将操作系统至于操作系统的虚拟化模式中的根模式（参考上文的『硬件虚拟化』），接着其会创建一个/dev/kvm设备文件供用户空间调用。这里大家肯呢个会有一些疑问，例如『KVM就做这么点事情吗？那么我是怎么加载我的操作系统ISO文件并启动虚拟机的呢？难道直接和/dev/kvm打交道？但从这里的说明来看KVM最多只能保证CPU指令被硬件CPU执行，并没有看到其提供了虚拟的显示终端、鼠标等IO设备啊』。是的，KVM并没有提供这些功能，KVM只是让宿主机操作系统进入支持硬件虚拟化的层级并等待外部传来指令罢了，对于IO虚拟化这些东西则是由QEMU负责的。

QEMU本身是一个功能完备的软件虚拟化程序，不依赖KVM，其就能够独立提供完善的虚拟化功能，包括提供IO设备的虚拟化，以及模拟CPU进行指令的解析。但软件虚拟化的缺点我们上文讲过，那就是性能开销太大，因此QEMU需要一个方法提高其性能，这个方法就是KVM。

举个简单的KVM配合QEMU的例子，当我们在QEMU中启动一个操作系统的时候，首先QEMU通过虚拟的磁盘设备加载这个系统的执行文件，然后QEMU在CPU硬件的虚拟化模式的非根模式下直接在硬件CPU上执行这个执行文件(CPU之所以进入虚拟化模式是由于加载KVM模块的时候KVM模块执行了类似VMXON等指令)，对于其中一些敏感指令，CPU硬件会将这些指令传递给根模式下的内核，而内核由于加载了KVM模块，因此这些敏感的指令的执行会由内核模块KVM进行执行。通过这种方式QEMU具有了硬件虚拟化的性能，同时由于QEMU本身是一个完备的虚拟化软件，具有各种输入输出设备的模拟，因此KVM借助QEMU也就能向用户提供一个完整的虚拟化环境。

说的简单点，KVM只模拟CPU和内存，因此一个OS可以在上面跑起来，但是你看不到它，无法和他沟通。于是有人修改了QEMU的代码，把它模拟CPU、内存的代码换成KVM，而网卡、显示器啥的留着。因此QEMU+KVM就成了一个完整的虚拟化平台。

那么如何知道自己的系统是否加载了KVM模块呢？通过上面的介绍我们知道至少有两种方法可以知道这一点：

查看/dev/kvm设备文件是否存在

通过lsmod | grep -i kvm进程查看

另外我们已经知道了QEMU和KVM的关系，那么启动一个使用KVM模块的QEMU的命令的含义就比较清晰了。我们知道用户态的程序实际上是QEMU，因此命令还是用的QEMU的命令，只不过我们要让QEMU知道不要使用自己的虚拟化CPU，而使用KVM来进行硬件虚拟化，因此需要加上enable-kvm的选项。例如：

qemu-system-x86_64 -m 700 -boot order=dc -smp 1 -hda /root/kvm_demo/rhel6u5.img -cdrom /root/Desktop/rhel-server-6.5-x86_64-dvd.iso -vnc 192.168.19.105:1 -enable-kvm

如何知道一个QEMU的虚拟机是否使用了KVM呢？最简单的方法就是ps看下进程的启动命令中有没有enable-kvm，或者可以通过QEMU monitor，(ctrl-alt-2, 再 ctrl-alt-1 回到VM的显示屏)然后输入info kvm进行查看。

libvirt

提到虚拟化就不得不提libvirt，很多人可能听说过KVM，但对libvirt可能是第一次听说。简单的说，libvirt工具集是对虚拟化产品的一套管理工具，libvirt本身是个接口库，提供了一套接口供应用调用，应用调用这些接口后可以对虚拟化软件进行管理，例如调用某个接口，可以重启某个KVM虚拟机，或者是调用某个接口可以新建一台VirtualBox虚拟机等等。笔者必须指出的是类似libvirt这种将原先需要手工操作(例如手工点击鼠标建立虚拟机)的动作做成接口库的方式，是运维自动化的一个基石。无论是现有的OpenStack平台还是AWS、阿里云等公有云，亦或是Docker、Kubernetes等都无一不提供了API的方式供外界调用，且这些API基本都是Restful风格的API，使用起来非常简单。大家做自己的运维系统的时候一定要注意这一点，要记住如果你做的是平台，那么一定不能让用户只能通过键盘鼠标操作，因为一涉及到键盘鼠标那么就说明还是需要人来操作，这样就不自动化了。

继续来看libvirt，libvirt的好处除了提供一套操作虚拟化软件的应用开发接口外，它的这些接口还屏蔽了底层不同虚拟化产品的差异，欢聚话说如果你基于libvirt开发了一套管理KVM的平台，现在想去用这个平台管理Xen那么你调用的libvirt的那些代码基本上不用做太大改动(笔者没有实际使用过libvirt开发应用，但这种致力于屏蔽差异的SDK从笔者的经验来说其实很难做到完全的屏蔽)。

libvirt这套接口库实际上在对虚拟化软件进行管理的时候，其是通过libvirtd这个守护进程进行管理的，也就是说应用的代码调用libvirt提供的接口，后者再和libvirtd进行通信，而libvirtd则负责进行具体的操作。

有很多基于libvirt开发的虚拟化管理工具，例如：

virsh 用于管理虚拟化软件中的虚拟主机以及管理虚拟化软件本身的一些配置

virt-manager 图形化的虚拟软件管理工具，功能类似于virsh但提供了一个UI界面

virt-top 用于查看虚拟机中进程资源消耗的工具，类似于top命令，但其可以从宿主机调用并直接查看虚拟机内部的进程的信息

关于KVM以及libvirt、virsh的一些有用的常见操作命令可以参见本书附录。

OpenStack简介

从上面的介绍我们可以看到，在企业中虚拟化先是有了虚拟机让用户在虚拟机提供的管理界面上分配主机，然后又有了libvirt，这样用户可以调用接口封装常见的虚拟机操作，例如通过脚本自由的创建虚拟机、resize虚拟机等。这个趋势是朝着减少用户人肉工作量发展的。既然libvirt提供接口的这种方式可以让用户减少手工操作虚拟机的劳动量，那么慢慢的自然会有对存储、对网络减少用户工作量的需求，另外libvirt虽然提供了接口，但这个接口依旧太底层了，虽然相比厂家提供的原生的管理界面要提供了更自由的调用方式，但对于基于这些接口之上的功能并没有覆盖，例如虚拟机的高可用，或者是针对某个企业内部网络环境的虚拟机环境的定制化部署。这时就需要OpenStack这类私有云平台来解决这些问题了。

很多人会把OpenStack和公有云做对比，但笔者并不认为私有云和公有云是一个东西。私有云是一个具备运维自动化的运维团队早晚会去设计开发部署的一系列系统，私有云的目的是服务企业自身。如果没有公有云这个概念，那么私有云其实就叫做『运维自动化平台』。而公有云这个概念必定是产生在大型企业中的，传统的公司即便有非常强烈的运维自动化意识，但如果机器规模不够、运维研发人员不够，那么是不会产生公有云的。对于企业来说公有云是一种选择，就和选择IBM还是联想的服务器一样，但不论选择什么只要其依旧需要人肉操作，那么就一定会有运维自动化系统的需求。目前的趋势是公有云在未来是方向，但目前还需要私有云去协助企业做过渡。

再来看OpenStack， OpenStack其实就是一系列运维自动化系统的统称，就和Hadoop生态圈一样。其诞生在2010年，由RackSpace和NASA共同设计开发并开源。其版本的发布周期是每6个月发布一次。每个版本的命名方式都会由社区在邮件组里发起投票，这些版本的首字母会从A一直到Z，然后再从A开始。目前主要有下面这些版本：

Austin

Bexar

Cactus

Diablo

Essex

Folsom

Grizzly

Havana

Icehouse

Juno

Kilo

Liberty

Mitaka

根据系统功能的不同，一般会把公有云（或者说运维自动化系统这类私有云）根据功能分为下面三大类：

IaaS（基础设施即服务）。基础设施指的就是虚拟主机、虚拟网络、虚拟存储等等这类。可以类比为对应的物理硬件的资源。

PaaS（平台即服务）。平台指的就是运行应用所使用的数据库、缓存、WEB中间件等等。

SaaS（软件即服务）。软件即我们平时使用的通过WEB方式提供的服务，例如QQ邮箱、Google地图等等。注意和上面PaaS做区分。

根据这种区分方法我们很容易可以看出PaaS是处于IaaS之上的，而SaaS又需要用到PaaS的服务，因此SaaS在PaaS之上。这种PaaS叠在IaaS上，而SaaS又叠在PaaS上的结构类似于函数调用里的stack，OpenStack中的Stack也就是这个意思。但实际上在设计系统的时候IaaS和PaaS区分的不是这么明显，例如也可以不依赖虚拟机，直接在物理机上提供数据库服务。

依据这种划分，目前OpenStack的主要组件有：

IaaS：Nova（提供虚拟机这类计算资源服务），Neutron（提供网络虚拟化服务），Cinder（提供块存储虚拟化服务），Swift（提供对象存储服务），

PaaS：Trove（提供数据库服务），Magnum（提供容器服务），Sahara（提供大数据服务）

SaaS：Keystone（提供认证服务），Glance（提供镜像服务），Horizon（提供控制台）

这里列出来的并不是很全，目前OpenStack的子项目数目非常大，大家可以根据自己工作的实际需要去官网查看有没有自己需要的服务。在这些服务中，Swift、Keystone、Nova、Neutron、Cinder、Glance这六个服务称为Core Services。其余的服务称为Optional Services。这两大类的区别主要在于社区对其的重视程度，在大版本的发布，日常的邮件讨论以及会议中对Core项目会有更多的关注。因此Core项目的质量一般是会更好的。但如果某个子项目后面有大公司推动的话，子项目本身的质量也不一定比Core项目来的差。

从这里简单的介绍就可以看出OpenStack实质上就是一系列的运维自动化平台。相比较自己独立的开发一个运维自动化平台来说，使用OpenStack进行运维自动化，或者基于OpenStack进行二次开发都是不错的选择。例如对于规模较小的公司，只需要使用Nova+Keystone+Glance+Horizon就可以搭建出一套私有云（当然对于规模较小的公司，首要的目的是保证业务的稳定，因此在尝试自己搭建私有云之前需要和VMWare等公司提出的商业化方案进行权衡）。对于上一点规模的公司（在国内很多游戏公司都有这个规模，并且都使用了OpenStack）则会考虑基于OpenStack和自己已有的硬件进行二次开发，但这里的二次开发主要还是集中在Horizon这种控制台应用的二次开发。这种不对OpenStack核心组件进行二次开发的原因一般有下面几种：第一是公司规模不够，运维研发能力较弱。第二是OpenStack目前的版本已经非常成熟及稳定，适应能力强，可以开箱即用。第三则是OpenStack的核心组件代码较为复杂。笔者曾经做过Neutron的研发，对Nova、Keystone等代码也比较熟悉，这些项目的代码的量（即行数）非常的巨大，并且这些项目虽然代码的组织结构很好，但作为一个开源软件来说其什么都要涵盖，因此渐渐的代码复杂性就上来了。举个简单的例子，对于某个企业来说他只需要基于KVM+VLAN的环境即可，但Nova不仅提供了KVM的支持，还提供了VMWare、HyperV、ESXi等等的支持，同时网络上又有VxLan、VLAN、GRE，且控制器还能选择是用ML2还是ODN。第四个原因则是社区的代码迭代、变化太快，如果没有专门的团队去做这个事情那么公司的私有代码分支很快就会和社区的主干分支差别很大。这里笔者的建议是对于目前版本的OpenStack，如果公司的规模有这个需求（例如主管说明年的IT预算不能再涨了），那么可以尝试着去用一下，可以先在测试环境或开发环境搭建起来试试。

OpenStack架构

本书的后续章节会设计并开发一个基于容器的运维自动化系统，但在开发这个系统的同时，笔者会比较OpenStack、Kubernetes等系统的实现，因此这里先来看下OpenStack的架构。对于『架构』一词的定义其实并没有一个统一的定义，因此这里按照笔者自己的理解来对OpenStack的架构进行描述。

功能的划分方式

OpenStack提供了运维相关操作的各种自动化包装，因此在功能划分方面比较容易区分。例如提供计算资源的就交给Nova，提供网络资源的就交给Neutron。但在实际的软件开发过程中会遇到公共依赖的问题，例如无论Nova还是Neutron都会需要写日志，都会需要读配置，因此在OpenStack生态圈中这类公共的功能组件统一放在了一个Oslo的项目中。因此按照功能划分可以划分为两大类，一类是具体的提供实际功能的组件，如Nova、Neutron、Glance等，一类则是公共类库，后者主要就是Oslo。按照这种思路，如果读者对某个组件感兴趣那么只要学习对应的组件即可，因为OpenStack的这种功能划分架构很好的保证了源代码的独立性。

组件的解耦方式

根据功能的划分从大的方面切割了运维功能层面的系统，但对于某个独立的子系统来说其实现依旧是非常复杂的，因此这里的组件解耦就是指的某个组件内部代码设计的时候如何解耦。例如对于Nova来说，其要支持数目庞大的虚拟化软件（如需要支持KVM、支持HyperV、支持EXSi等等）。对于OpenStack来说其主要是通过stevedore这类工具解决这个问题的。stevedore其实就是一个动态加载模块的工具，熟悉面向对象的读者肯定很容易理解其功能：例如Nova提供一个接口文件：

class VMBase(object):

    def create_vm(self):

    '''创建虚拟机的接口'''

    def destroy_vm(self):

    '''销毁虚拟机的接口'''

然后对于KVM来说，其提供一个类实现这个接口：

class KVM(VMBase):

    def create_vm(self):

    '''调用libvirt实现KVM的虚拟机创建动作'''

    def destroy_vm(self):

    '''调用libvirt实现KVM的虚拟机销毁动作'''

OpenStack中大部分组件都是通过这种方式提供接口，这样各个厂商的人就能很容易的集成自己的产品到OpenStack中。除了这种解耦方式（即项目与具体实现分离）外，由于很多项目其自身功能非常复杂，因此其内部会像我们上面提到的『微内核』一样，将各个功能分离开来。和『微内核』通过进程分离类似的是，对于OpenStack的项目来说，其也是通过进程进行分离的。例如Nova就提供了nova-api进程用于提供用户的请求的服务，提供了nova-scheduler进行用于调度决策，提供了nova-compute进程用于计算资源的实际管理。这些进程（按照流行的方法，这些进程可以被称作『微服务』）之间通过基于TCP的协议交互（例如通过MQ、通过HTTP等）。从这一角度来说这也和上面的基于接口的方式提供具体实现类似，只不过这里的接口定义是由TCP之间的协议交互来确定的。因此对于很多大学的研究人员来说OpenStack是一个很好的实验他们研究理论的平台，例如有人在研究调度器，那么只要替换nova-scheduler的实现即可在OpenStack中运行他们的调度器而不用对其它组件有过多的了解。

接口的调用方式

在OpenStack中接口的调用主要分为两种，一种是RESTful风格的基于HTTP协议的API方式，还有一种是基于消息的调用方式。前者主要供用户调用，例如创建虚拟机就可以通过这类接口进行。后者则主要是组件内部调用的时候使用，例如nova-api和nova-scheduler之间的调用就会通过这种方式。这里简单的介绍一下RESTful API和基于消息的调用。

RESTful API

RESTful这个名词来自于Roy Thomas Fielding在其2000年的博士论文『Architectural Styles and the Design of Network-based Software Architectures』。REST的英文全称为『Representational State Transfer』，RESTful也就是满足REST风格的接口设计方式。想要详细了解的读者可以看下这篇文章，就笔者而言其主要是提供了一个相较于基于XML这类格式而言简单的多的一种表示方式。在REST中，所有操作的对象都认为是一种资源，例如我们的虚拟机就是一种资源，然后在REST中提供了操作这种对象的动作，映射到HTTP协议就是GET、POST、DELETE等请求方式。例如在REST中要求提供查找、创建、修改、删除等动作，在HTTP中则通过GET、POST、PUT、DELETE来对应这些动作。由于是基于资源操作的，因此每个资源需要有确定的ID，在HTTP中一般通过URL实现，例如/vm/XXX，这里的XXX就是虚拟机的ID。于是获取XXX这个虚拟机的详细信息在HTTP中就变成了GET /vm/XXX。而创建一台虚拟机在HTTP中就变成了POST /vm，并且按照REST的要求，这个POST请求需要在返回的报文中包含创建了的虚拟机的ID。通过这里简单的例子我们可以看出，HTTP是满足REST规范的一种实现的载体，因为HTTP实现了REST所要求的语义动作，使用者在设计HTTP接口（主要就是设计URL的格式、请求及相应报文的内容）的时候按照REST的要求（例如这里的POST需要返回ID号，URL需要包含资源ID等）设计HTTP接口后，这类HTTP接口就可以称为REST风格的接口，也就是RESTful的接口。当然不是一定要基于HTTP来实现REST，只是这里HTTP的协议满足了REST的要求（Roy Thomas Fielding应该是受到HTTP协议的启发而设计的REST架构）。

另外，REST还规定了这些动作的约束。例如查询这个动作就不能对系统造成修改。

那么相比较于其它的协议，这种REST的风格有什么好处呢？笔者在实际的使用过程中发现的好处主要是其接口的格式非常的简单，并且很多接口不需要去查看文档就能理解其含义。这个其实比较好理解，例如某个系统说它提供的接口是RESTful的，而你又对RESTful有先验的知识，那么你自然会很容易的理解这个接口。而如果某个自动提供的接口是它们自己定义的一种规范，你对这种规范毫无了解，那么学习该系统接口的学习曲线自然就会比较长。从这一点来说如果用RESTful提供接口的平台越多且用的越规范，那么就会很大的降低大家对于接口的学习曲线。这一点很面向对象中的封装有很大的相似性，封装的一个优点就是减少使用者的学习接口的曲线。

消息

虽然基于RESTful接口可以提供非常易用的API，但HTTP协议本身的开销以及其请求不能保证一定传递等问题其不适合组件内部之间的交互。在OpenStack中组件间的交互主要通过消息进行，或者说OpenStack的RPC主要是基于消息的。在OpenStack中消息主要通过AMQP协议进行，只要消息中间件（一般称作broker）支持AMQP协议那么就可以作为OpenStack的消息中间件使用。AMQP协议大家就认为和HTTP协议一样，是种基于TCP的协议即可。为什么OpenStack要依赖消息中间件实现RPC而不是自己直接实现呢？主要是基于消息中间件可以减少开发的复杂度，当开发关注于业务逻辑而不是非业务的消息传输。在OpenStack中对消息的使用主要有两类，一类是一对一的，及组件A调用组件B的某个接口，实现为A发送一个消息给消息中间件，且B在监听消息中间件中对应的队列，当A投递消息后队列中就有了消息，此时B就取到了消息并进行处理，处理完后返回结果也是类似的方式。还有一类是一对多的，也就是多播。基于消息的模型有一个问题就是消息中间件会成为一个瓶颈，因为要设计一个处理小块消息的组件其实现难度还是比较大的，要处理小块内存的管理，队列的维护等等，在消息量很大的时候（例如整个集群的控制器异常的时候由于控制器不响应因此各个组件会发送大量重传消息）很容易出现问题。解决的方法一般是更具zone划分区域，每个zone一个消息中间件，当然这种方式也会带来问题，因此大家在使用的时候需要根据实际的集群规模进行部署。

请求的执行时序

最后来看下请求的执行时序，一般的，在OpenStack中的请求都是按照如下的时序进行的：

用户想要创建一台虚拟机，因此会去调用nova-api的接口，但是在调用之前会先把自己的认证信息发送给keystone获取token

keystone得到用户的认证信息后如果校验无误则会返回token

用户发送RESTful的请求给nova-api，并附上token及虚拟机的相关信息

nova-api获取到请求后校验token（一般是和keystone做交互或者是基于公钥做校验），校验无误后发送消息给nova-scheduler

nova-scheduler通过调度逻辑确定这台虚拟机会被创建在哪台物理主机上，并将这个结果返回给nova-api

nova-api在获取到具体的物理主机信息后，和cinder、glance等做交互获取相关资源。这里的交互方式和上面与keystone之间做token认证的交互类似

nova-api把相关的信息写入DB，保证这个虚拟机的信息的持久化，然后发送消息给nova-compute

nova-compute收到消息后从DB中获取相关虚拟机信息，并创建虚拟机。需要说明的是在nova中和DB的交互会走一个nova-conductor的组件

其它的请求基本类似这个过程。这里除了消息之间交互及组件间合理的分工值得我们学习外，虚拟机信息在DB中的持久化时机也值得我们学习。如果虚拟机的信息不存在DB中，那么如果运行这个虚拟机的物理机宕机了那么这台虚拟机的信息可能就丢失了。

OpenStack源码及社区

对于感兴趣的读者，笔者建议可以学习下OpenStack的源码并参与一下开源社区。目前很多运维自己组建的自动化平台都是以Python语言为主的，因为对于自动化平台来说Python的性能足以满足大部分需求，且这类自动化平台的代码由于其业务逻辑较为简单，因此其代码结构上并不是非常的复杂，加上Python本身非常简洁的语法，因此对于自动化平台来说Python具有天生的优势。而学习一门语言光有书本上的知识是不够的，必须要去写代码，做项目，因此通过阅读OpenStack的代码可以了解到一个大型的Python项目是如何组织的，可以从中学习到很多东西。

如果读者要阅读OpenStack的源码，笔者建议从Keystone的源码开始学习。因为Keystone比较简单，主要就是做了认证功能，同时提供了基本的服务发现。但麻雀虽小五脏俱全，Keystone的代码中用到了很多其它组件都用到的东西，例如对Oslo库的使用，对消息的使用，对RESTful的使用等等，建议读者在学习的时候，每遇到一个Keystone使用的库就去学习一下这个库的用法及作用，例如在看Keystone代码的时候发现其import eventlet了，则建议读者去学习一下eventlet的使用。等差不多Keystone依赖的模块都了解了一遍，且Keystone的代码也基本上看完后，然后就可以根据需要去学习其他的组件的源码，例如Nova、Neutron等，这时读者就会发现这些代码基本是大同小异的，大部分的『异』主要是业务逻辑上的。另外在学习之前务必要从源码开始安装一下相关组件，例如可以从源码安装下Keystone，大部分组件的官方文档里都有如何从源码安装的说明，按照说明进行即可。当然也可以看下devstack的代码，devstack的作用就是从源码开始安装一个可以工作的OpenStack环境，且它的代码都是shell写的面向过程的代码，读起来比较容易。最后关于源码学习笔者的建议是不光关注代码本身，还要关注下组件的测试代码的实现。每个组件都有各自的单元测试、功能测试、集成测试代码，虽然笔者认为其中很多测试代码是多余的，对纯粹的基于单元测试驱动的测试驱动开发也不是非常认同，但通过测试代码的学习读者可以从中学习到一些测试框架，并能学习到如何进行功能测试、集成测试，这对今后读者编写可靠可维护的代码是非常有必要的。

最后关于OpenStack社区，大家可以从订阅邮件组开始。邮件是OpenStack社区及开发中非常重要的一个环节，一般有什么新的BUG，或者有什么使用上的问题等等都会有人先发送一个邮件到社区。因此大家可以去官网上订阅一下这个邮件组，这样所有的讨论邮件你就都能收到了。如果有人在邮件组中发了一个询问的邮件，而读者恰好也遇到过或者知道解决方法，那么就可以直接回复这个邮件帮助对方解决问题。开源社区的进步就是这样一步一步由个体的小贡献开始的。除了邮件外每个项目都会定期的在IRC中开周会，IRC大家就认为是QQ群即可，大家可以加入到这个群中参与讨论。但由于国内存在时差，且很多人可能都没加入这些群组，因此笔者还是推荐通过邮件的方式开始参与社区。当然除了邮件和IRC外，提交BUG或者提交新的功能到主干分支中也是非常重要的一环。OpenStack使用launchpad来跟踪需求及BUG，而大的新功能需求也被称为blueprint，也就是说如果读者有一个较大的新功能想贡献给社区，可以先提交一个blueprint，然后社区会讨论这个blueprint是否有价值，如果社区讨论通过了这个blueprint，那么就可以在launchpad中建立一个任务跟踪这个需求的实现进度并指派对应的实现人。这个过程中有一个概念叫做PTL，即Project Team Leader，每个大项目都基本上至少有一个PTL，同时会有很多Core Developer（一般只称作Core）。对于大家提交的BUG或者blueprint会由PTL和Core进行评审并打分，只要他们的打分通过了阀值那么相关的代码补丁或提议就会通过，此时对于代码补丁会自动进入CI流程，即会自动的尝试合并入主线分支。PTL和Core一般由社区在邮件组中选举出来，但不可否认的是基本上他们都是大公司（如RedHat、IBM）的员工，毕竟这些大公司的参与度很高，会有全职的人投入到项目中去。
