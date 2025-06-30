#### alpine 的一些问题
###### 导致绝大部分问题来源的 C 标准库兼容性问题
alpine 的 C 标准库的实现是musl，而不是其他大多数 Linux 发行版（如Ubuntu，CentOS）使用的 glibc  
这两种实现在大多数情况下都是可替换的。这就是为什么在大多数情况下，我们可以从 Ubuntu 切换到 Alpine，而不会注意到任何不同。  
但它们并不完全兼容。大多数 Linux 上运行的软件都是针对 glibc 进行测试的，而针对 musl 的测试则非常有限。。因此，许多在其他发行版中表现正常的程序，在 Alpine 上可能会出现异常行为或构建失败。

此外，glibc 的性能普遍高于 musl。行业现状是：绝大多数 Linux 软件都以 glibc 为主流适配对象，musl 生态相对小众，兼容性和性能都存在一定劣势。

**如果你选择 Alpine 作为基础镜像，意味着：**

+ **优势：**
    - 镜像体积更小，节省存储空间
+ **劣势：**
    - 存在兼容性问题，许多软件对 musl 支持不佳
    - 性能略逊于 glibc

换句话说，后文列举的 Alpine 使用问题，几乎都可以归结为 C 标准库兼容性的问题。

###### DNS 问题：
DNS 问题的跟踪 issue 喵

[DNS because of musl/Alpine base? · Issue #138 · nicolaka/netshoot](https://github.com/nicolaka/netshoot/issues/138#issuecomment-1541172638)

（Alpine 3.18 或更高版本升级了 musl 的版本到 1.2.4 之后应该解决了这个问题  详情见上面的跟踪）

DNS 问题的简单解释和跟踪

[Why I Will Never Use Alpine Linux Ever Again](https://martinheinz.dev/blog/92)

musl's resolver did not support the use of DNS over TCP until version 1.2.4. This difference prevented the use of larger packets produced by protocols such as DNSSEC and DKIM

在 musl 1.24 版本之前，DNS 存根解析器都没有包括 TCP 回退， 这意味着任何超过 512 字节的 DNS 调用都会失败，

因为只有大数据包会丢掉，所以如果只是 在单纯的 docker 环境中，这个问题没那么容易出现

不过在Kubernetes 环境中更容易复现，因为其 DNS 查询通常字段更多。这个问题就比较容易出现

###### Golang net 问题
[Just a moment...](https://stackoverflow.com/questions/36279253/go-compiled-binary-wont-run-in-an-alpine-docker-container-on-ubuntu-host)

Golang标准库——或者更具体地说是 net/http 或 os/user 模块——依赖于 C 库，因此依赖于 glibc。然后如果应用程序需要 CGO_ENABLED=1，即使不使用这些特定的模块，使用 Alpine 显然也会遇到问题。这个也是标准 C 库导致的

###### Python Nodejs 的编译构建问题
For Python, for example, many popular libraries such as NumPy, or Cryptography rely on C code for optimizations. Luckily, at least for some of the libraries like Numpy, chances are, you will find Alpine-based compiled package and relevant dependencies. For the less popular ones however, you might have to compile them yourself and is it really worth the hassle? In my opinion... nope. Additionally, even if you manage to build an image that includes for example numpy, its size will be ~400MB, at which point using Alpine for its small size doesn't really help much.

Python 中很多库（如 NumPy、Cryptography）依赖 C 扩展，而这些扩展通常预编译的是基于 glibc 的包。在 Alpine 中你需要手动编译它们，这不仅耗时而且容易出错。此外，即使你设法构建了一个包含 numpy 等库的镜像，它的大小也会达到约 400MB，此时使用 Alpine 的小巧体积也没什么用。

Also, the build time for such an image will be atrocious, you can try for yourself, following Dockerfile takes almost 10 minutes to build:

此外，构建这样的镜像的时间会非常长，你可以自己尝试一下，下面的 Dockerfile 需要将近 10 分钟才能构建：

```plain
FROM python:3.11-alpine
RUN apk --update add gcc build-base
RUN pip install --no-cache-dir numpy
```

Obviously, similar issues can happen also in other languages. For example Node.js uses addons, which are written in C++ and compiled with node-gyp, these will depend on C libraries and therefore on glibc.

显然，类似的问题也可能发生在其他语言中。例如，Node.js 使用的插件是用 C++ 编写的，并用 node-gyp 编译，这些插件依赖于 C 库，因此也依赖于 glibc 。

###### Alpine 的线程问题
musl 默认线程栈仅 128KB，而 glibc 通常为 2MB 或更高（普遍是 2-10mb）。这会导致 Python 应用出现栈溢出崩溃或性能下降的问题。

###### 字符处理问题
[https://github.com/docker-library/php/issues/240](https://github.com/docker-library/php/issues/240)

最常用的 `iconv` 函数不能正常工作

The iconv implementation musl is very small and oriented towards being unobtrusive to static link. Its character set/encoding coverage is very strong for its size, but not comprehensive like glibc's. In particular:  
iconv 实现 musl 非常小巧，并且注重不干扰静态链接。它的字符集/编码覆盖范围就其大小而言非常强大，但不如 glibc 全面。特别是：

+ Many legacy double-byte and multi-byte East Asian encodings are supported only as the source charset, not the destination charset. JIS-based ones are supported as the destination as of version 1.1.19.  
许多传统的双字节和多字节东亚编码仅支持作为源字符集，不支持作为目标字符集。从 1.1.19 版本开始，基于 JIS 的字符集才被支持作为目标字符集。
+ Conversion to ISO-2022-JP is stateless and produces shifts in/out of nondefault states around each character.  
转换为 ISO-2022-JP 是无状态的，并且会产生每个字符周围的非默认状态的转变。
+ Transliterations (//TRANSLIT suffix) are not supported.  
不支持音译（//TRANSLIT 后缀）。
+ Converting to legacy 8-bit charsets is significantly slower than converting from them.  
转换为传统的 8 位字符集比从它们转换要慢得多。
+ Prior to version 1.1.19, conversions from plain UTF-16 or UTF-32 without an explicit endianness assumed big endian and did not honor BOM. Now they honor BOM, but BOM is never produced in output.  
在 1.1.19 版本之前，从普通 UTF-16 或 UTF-32 进行转换时，如果没有明确指定字节序，则会默认采用大端序，并且不支持 BOM。现在，它们支持 BOM，但输出中不会产生 BOM。
+ Misleading, deprecated charset aliases like UNICODE as an alias for UCS-2 are not supported. The IANA preferred MIME charset names should be used instead.  
不支持使用误导性、已弃用的字符集别名（例如将 UNICODE 作为 UCS-2 的别名）。应使用 IANA 首选的 MIME 字符集名称。
+ Contrary to POSIX, glibc iconv generates EILSEQ when a character is not representable in the destination charset. musl, in accordance with POSIX, performs an implementation-defined conversion and returns the number of such inexact conversions performed. At present, it replaces the character with an asterisk, but something akin to glibc's //TRANSLIT mode may be substituted in the future. Code written assuming the glibc semantics (error when no exact conversion is possible) may need to be tuned to work well on musl and other conforming iconv implementations.  
与 POSIX 相反，当字符无法在目标字符集中表示时，glibc iconv 会生成 EILSEQ。musl 遵循 POSIX 规范，执行实现定义的转换，并返回执行此类不精确转换的次数。目前，它会用星号替换字符，但未来可能会使用类似于 glibc 的 //TRANSLIT 模式的模式。假设 glibc 语义（无法进行精确转换时会出错）编写的代码可能需要进行调整，才能在 musl 和其他符合 POSIX 规范的 iconv 实现上正常运行。

###### Java  （Oracle  Java）的问题
Java依赖C类库，而支持Musl的Java实现，并非主流。

比如  Oracle Java 依赖于仅在 glibc 中找到的特定符号

```plain
# ldd bin/java
    /lib64/ld-linux-x86-64.so.2 (0x7f542ebb5000)
    libpthread.so.0 => /lib64/ld-linux-x86-64.so.2 (0x7f542ebb5000)
    libjli.so => bin/../lib/amd64/jli/libjli.so (0x7f542e9a0000)
    libdl.so.2 => /lib64/ld-linux-x86-64.so.2 (0x7f542ebb5000)
    libc.so.6 => /lib64/ld-linux-x86-64.so.2 (0x7f542ebb5000)
Error relocating bin/../lib/amd64/jli/libjli.so: __rawmemchr: symbol not found

```

JDK支持alpine，有两种方式来实现

+ 通过OpenJDK's project [Portola](https://openjdk.java.net/projects/portola/)项目来支持，这个是JDK中专门支持Musl的项目，当前仍未发布稳定性。
+ 在alpine上安装glibc，替换musl来实现的

或者干脆自己装一个 glibc

 GitHub 上有一个已经提供了预编译的 glibc 库，名字为alpine-pkg-glibc，装上这个库就可以完美支持 Java，同时还能够保持体积很小。

但是我个人意见还是，JDK使用alpine，是个风险很高的事。

###### 下载或者挂载的二进制运行 no such file or directory 或者 not found
该二进制是 glibc 下编译的，挂载到 musl 的 alpine 里会有这样的问题，

glibc 下编译二进制文件，所需要的动态链接库是在/lib64目录下，但是Alpine使用的是musl libc并不是标准的glibc，所以动态链接库的位置不一样，在alpine中没有lib64目录，动态链接库在lib目录下。

可以使用 glibc 的 alpine，或者安装 libc6-compat 不行再试试 gocmpat

要不然你挨个创建软链接也可以

```plain
RUN mkdir /lib64 \
    && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2
```

###### 怎么判断是不是 musl 带来的问题
[musl libc - Functional differences from glibc](https://wiki.musl-libc.org/functional-differences-from-glibc.html)

可以先看这个文档来着

###### alpine 的使用意见
+ _**如果你的服务，不依赖C，那alpine是合适的选择，否则不应当使用alpine做为基础镜像**_

**不建议使用 Alpine 的场景**

- [ ] java 应用，JVM 与 glibc 深度绑定，任何基于 musl 的方案都不太好，但是如果替换 glibc 的话，为什么不用 `gcr.io/distroless`或者 `ubuntu-slim 或者 debian-slim 呢`，更何况 java 应用大部分都是 springboot， 你何哭扣那一点点空间，一个 springboot 应用要打包多少东西进去大家都有数的。
- [ ] 依赖 CGO 的 Go 应用：如果需要 CGO_ENABLED=1 来链接一个 .so 动态库，就是在依赖 C 库。musl 和 glibc 的兼容性问题就会暴露，建议直接用 glibc 环境。
- [ ] 运行第三方或预编译的二进制文件：只要不是 Alpine 环境下自己编译的二进制，都应假定为 glibc 版本，musl 下大概率运行失败。
- [ ] python/nodejs: 涉及 C 扩展的库（如 numpy、cryptography、node-gyp 插件）在 Alpine 下构建复杂(时间还长)且体积优势消失。
- [ ] 强依赖于glibc的大型项目，比如 Tomcat、Jenkins、GitLab 等 ，人家官方就不提供基于Alpine 的 Dockerfile，你就不要自己那啥自己了

**可以用 alpine 的地方**

纯静态 Go 程序：用 CGO_ENABLED=0 编译，不依赖任何外部 C 库的。

极简 Shell/Lua/Tcl 脚本：只依赖 POSIX 标准命令行工具。比如探针啊，命令行的 job 啊 ，sidecar 容器啊这种的

完全自控的 C/C++/Rust 编译：你自己从源码开始，使用基于 musl 的工具链进行编译，能确保所有依赖都兼容。

环境极度受限：在某些 IoT 或嵌入式场景，存储空间或网络带宽极其宝贵，节省几十 MB 的空间是硬性指标。

**其他镜像使用的点**

+ _**考虑到部署的一致性，就算是容器，使用同一系的Linux基础镜像是更妥当的选择，比如全是Debian系或RHEL系等**_
+ _**优化镜像的空间非常有必要，各种建议仍然非常有价值，但不要走的太过，镜像的空间问题并没有你想像的那么严重。**_
+ _**在公司或项目级别，不要使用Docker Hub或其它公有云镜像，使用简易的registry或企业级Harbor，Nexus等搭建一个内网镜像中心，能让容器镜像的上传下载更快捷方便。**_
+ 

综上，虽然 Alpine 镜像体积小巧，但由于 musl libc 带来的兼容性问题，在实际生产环境中并不总是最佳选择。前文已详细分析了其在 DNS、Golang、Python/Nodejs、Java 等多种场景下的具体问题。这里再补充两个常见关注点：镜像大小与安全性。

#### 镜像大小优化
容器镜像虽然高效，但空间占用是实际问题。即使是简单的 Java 服务，镜像通常也有 100-200MB。CI/CD 自动化构建和多服务部署会让空间消耗快速累积。因此，很多教程和官方建议都强调"使用更小的基础镜像"，而 Alpine 是最常被推荐的。  
alpine是一个非常特别的Linux发行版本，它本来是用于嵌入式系统中的，大小才5M左右，对于嵌入式系统，非常合适。  
本来并非用于Linux服务领域，但随着容器的流行，这个才5M大小的Linux在Docker中流行起来了，因为太小了，非常节省空间。

##### Alpine 镜像体积对比
对比下同样属于Linux镜像Ubuntu与alpine的大小

```plain
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
alpine       3.16.2    9c6f07244728   7 days ago    5.54MB
ubuntu       latest    df5de72bdb3b   2 weeks ago   77.8MB
```

alpine镜像解压后才5.54MB （压缩后2.68M），而流行的Ubuntu解压后的大小是 77.8MB （压缩后是22.04M)。

##### Alpine 为什么这么小？
1. **删除了大量非必须的资源文件**
    - 例如 `/usr/share/locale`（国际化文件）、`/usr/share/doc`（帮助文档）等，这些目录在 Alpine 中压根没有。
2. **使用 musl 替代 glibc**
    - `/usr/lib/x86_64-linux-gnu` 与 `/lib/x86_64-linux-gnu` 这两个目录大多是 glibc、libc、perl 等的共享类库。Alpine 没有使用 glibc，而是使用了 musl，所以这一部分占据的大小也小了很多。
    - 通过在 musl 官网与 glibc 官网查阅它们的压缩安装包大小分别是：
        * musl：1.1MB musl-1.2.3.tar.gz
        * glibc：18M glibc-2.3.6.tar.gz
        * 123K glibc-libidn-2.3.6.tar.gz
        * 320K glibc-linuxthreads-2.3.6.tar.gz
        * 1.8M glibc-ports-2.16.0.tar.gz
    - 可以看出，光是安装包就有20倍左右的差别，安装后当然相差更大。
3. **使用了 busybox 工具集**
    - Debian 中的 `/bin` 与 `/sbin` 目录大小明显高于 Alpine 的 `/bin`，这是因为 Alpine 使用的是 busybox 工具集。
    - busybox 是一个可执行文件，集成了 300 多个常用命令，被称为 bin 命令的瑞士军刀。
    - 在 Alpine 中，像 `ls`、`pwd` 这些命令其实都是 busybox 的 alias。例如：
        * 功能与 ls 类似：`busybox ls`
        * 功能与 pwd 类似：`busybox pwd`
        * 功能与 kill 类似：`busybox kill`
    - Alpine 中最主要的一个命令文件就是 busybox，而 busybox 是一个 5M 不到大小的，包含近 300 多个命令的工具集。
4. **没有 apt 与 systemd**
    - 在 Debian/Ubuntu 中，包管理是 apt，init 系统是 systemd。
    - Alpine 用 apk 取代了 apt，没有使用 systemd，而是使用了 OpenRC，无论是 apk 还是 OpenRC，都是轻而小的实现。

#### 更小镜像的优势一定很大吗
**究竟使用alpine这种基础镜像，能节省多少空间，在稍加思考后，我发现这可能也是一个伪优势。只是假想出来的优势。**

**我在这里以alpine与ubuntu来构建同一个服务镜像，看下它们的存储大小差距吧。**

```plain
java-grpc-sample-user   ubuntu    42805f4be518   8 seconds ago    304MB
java-grpc-sample-user   alpine    d2109aa9c0d6   51 seconds ago   208MB
```

如上所示，同一个Java服务，使用alpine构建的镜像，比使用ubuntu构建镜像节省了100M左右的空间，由于镜像是要压缩才上传下载的，压缩后的大小差距可能只有50M左右了。  
于是，我们假设下，如果我们的项目要构建100个服务镜像（算是比较大的项目），这是不是意味着节省了 100 * 100 （M）空间？  
错，大错特错。  
要知道，容器镜像是分层的，基于同一基础镜像构建的层，其实会共享同一个层。并不会每个镜像都复制一份基础层。这意味着，如果你在一个服务器上有100个服务镜像，但这100镜像是共享了同一个基础层（当然你得使用同一基础镜像来构建）。  
也就是，你并不会多100 * 100（M）的空间和网络下载量，一个Docker服务上你只多了100M的空间。

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750669034202-0fd34c23-e1d1-4202-8953-50bc0dcb49fc.png)

如上图所示，使用同一基础镜像构建的不同镜像，共享了同一基础镜像层，不会每个都重复产生一个。  
这意味着，所谓的节省镜像空间，实际产生的价值并未有你想像的那么大。

###### 安全性问题
最后，再提及下安全性问题。一些博客中，在推荐使用alpine时，提及了安全性这一点。  
理由是，alpine小，代码更少，理所当然安全性更高。  
我也仔细查阅了关于这一点的相关说法，发现这一点在业界并无统一认知，并非共识。反对此观点的人很容易举出反证，更主流的镜像维护的人更多，安全上的问题修复更快，所以更安全。  
所以，关于alpine更安全，这是一个有争议的点，至少在当下，尚无足够的证据证明它。因此不能成为理由。

#### 基础镜像的选择
最后我们说一下基础镜像的选择建议：

1. **优先选择官方镜像**
    - 官方镜像优于非官方镜像。如果没有官方镜像，则尽量选择 Dockerfile 开源、社区活跃的镜像。
2. **固定版本 tag**
    - 推荐使用具体版本号（如 `python:3.11-slim`），而不是 `latest`，以保证构建的可重复性和稳定性。
3. **优先选择体积小的镜像**
    - 在满足功能需求的前提下，尽量选择 slim、alpine 等精简版镜像，减少空间占用和安全风险。

##### 选型建议
+ **如果需要快速启动并运行项目，没有空间限制，也没有时间做太多测试**，建议直接使用标准官方镜像。此时最重要的是镜像功能齐全，镜像大小不是首要考虑。
+ **如果空间有限，且只需运行特定语言的最小环境**，如 Python、Node.js 等，推荐选择 `slim` 版镜像（如 `python:3.11-slim`）。
+ **如果有极端空间限制，且有时间充分测试**，可以选择 alpine 镜像。但要注意，alpine 可能导致更长的构建时间和潜在的兼容性 bug。如果遇到容器移植或新包安装异常，优先排查是否为 alpine 兼容性问题。

##### 常用基础镜像推荐
+ [debian-slim](https://hub.docker.com/_/debian)  （如：`debian:buster-slim`）
+ [ubuntu](https://hub.docker.com/_/ubuntu)  （如：`ubuntu:18.04`、`ubuntu:20.04`）
+ [centos](https://hub.docker.com/_/centos)
+ [python-slim](https://hub.docker.com/_/python)  （如：`python:3.11-slim`）

> 选型时建议结合实际需求、团队经验和测试情况综合权衡。
>

##### 参考


[不要在生产环境中使用alpine基础镜像 -- 容器基础镜像的选择](https://ttys3.dev/blog/do-not-use-alpine-in-production-environment)



[调试 DNS 问题](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-debugging-resolution/)



[Does Alpine resolve DNS properly?](https://purplecarrot.co.uk/post/2021-09-04-does_alpine-resolve_dns_properly/)



Why I Will Never Use Alpine Linux Ever Again





[https://ttys3.dev/blog/do-not-use-alpine-in-production-environment](https://ttys3.dev/blog/do-not-use-alpine-in-production-environment)  
[https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-debugging-resolution/](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-debugging-resolution/)  
[https://purplecarrot.co.uk/post/2021-09-04-does_alpine-resolve_dns_properly/](https://purplecarrot.co.uk/post/2021-09-04-does_alpine-resolve_dns_properly/)  
[https://martinheinz.dev/blog/92](https://martinheinz.dev/blog/92)

