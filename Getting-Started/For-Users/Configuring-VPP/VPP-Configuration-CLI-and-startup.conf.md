## VPP配置-CLI和startup.conf

成功安装后，VPP将在/etc/vpp/目录中安装名为startup.conf的启动配置文件。可以定制此文件以使VPP根据需要运行，文件默认有典型的默认值。

以下是有关此文件及其包含的一些参数和值的更多详细信息。

### 命令行参数
在我们描述启动配置文件（startup.conf）的详细信息之前，应该提到的是，可以在没有启动配置文件的情况下启动VPP。

参数按节名称分组。当为一个节提供多个参数时，该节的所有参数都必须用花括号括起来。例如，要通过命令行使用节名称为“unix”的配置数据启动VPP：
```
$ sudo /usr/bin/vpp unix { interactive cli-listen 127.0.0.1:5002 }
```

命令行可以显示为单个字符串或多个字符串。在解析之前，命令行上给出的所有内容都用空格连接成单个字符串。VPP应用程序必须能够找到其自己的可执行镜像。确保其工作的最简单方法是在启动时通过提供其绝对路径来调用VPP应用程序。例如：‘/usr/bin/vpp <options>’，VPP应用程序首先解析它们自己的ELF部分，以创建初始化、配置和退出处理程序（exit handlers）的列表。

使用VPP开发时，在gdb中通常足以启动这样的应用程序：

```
(gdb) run unix interactive
```

### 启动配置文件(startup.conf)

将启动配置指定为VPP的更典型方法是使用启动配置文件（startup.conf）。

通过命令行将该文件的路径提供给VPP应用程序，通常配置文件位于/etc/vpp/startup.conf中，如果VPP作为软件包安装，则默认的startup.conf文件也位于此。

配置文件的格式是一个简单的文本文件，其内容与命令行相同。

一个非常简单的startup.conf文件：
```
$ cat /etc/vpp/startup.conf
unix {
  nodaemon
  log /var/log/vpp/vpp.log
  full-coredump
  cli-listen localhost:5002
}

api-trace {
  on
}

dpdk {
  dev 0000:03:00.0
}
```

VPP通过-c选项加载配置文件，例如：
```
$ sudo /usr/bin/vpp -c /etc/vpp/startup.conf
```

### 配置参数

以下是一些节名称及其相关参数的列表。这不是一个详尽的列表，但是应该能使您了解如何配置VPP。

对于所有配置参数，请在源代码中搜索VLIB_CONFIG_FUNCTION和VLIB_EARLY_CONFIG_FUNCTION的实例。

例如，调用“VLIB_CONFIG_FUNCTION（foo_config，“foo”）”将使函数“foo_config”接收名为“foo”的参数块中给出的所有参数：“foo {arg1 arg2 arg3…}”。

#### unix节

配置VPP启动和行为类型属性，以及任何基于OS的属性。

```
unix {
  nodaemon
  log /var/log/vpp/vpp.log
  full-coredump
  cli-listen /run/vpp/cli.sock
  gid vpp
}
```

##### nodaemon

不要派生/后台运行vpp进程。从进程监视器调用VPP应用程序时，通常使用这种情况。默认情况下，在默认的“startup.conf”文件中进行设置。

```
nodaemon
```

##### interactive

将CLI附加到stdin/stdout，提供一个命令行调试接口。

```
interactive
```

##### log <filename>

启动配置和后续所有CLI命令都会记录到该文件中，尤其在人们不记得或不愿意在bug报告中包含CLI命令的情况下非常有用。默认的“startup.conf”中的日志文件是“/var/log/vpp/vpp.log”文件。

在VPP 18.04中，默认日志文件位置已从“/tmp/vpp.log”移至“/var/log/vpp/vpp.log”。VPP代码与文件位置无关，但是，如果启用了SELinux，则需要新位置才能正确标记文件。请在您的系统上检查本地“startup.conf”文件的位置。

```
log /var/log/vpp/vpp-debug.log
```

##### exec | startup-config <filename>

从文件名读取启动配置。文件的内容将像在CLI上输入的那样执行。这两个关键字是同一功能的别名。如果两者都指定，则只有最后一个才会生效。

这个文件中的CLI命令可能类似于：

```
$ cat /usr/share/vpp/scripts/interface-up.txt
set interface state TenGigabitEthernet1/0/0 up
set interface state TenGigabitEthernet1/0/1 up
```

参数举例：

```
startup-config /usr/share/vpp/scripts/interface-up.txt
```

##### gid <number | name>

设置调用进程有效的组id和组名称。

```
gid vpp
```

##### full-coredump

要求Linux内核转储(dump)所有内存映射的地址区域，而不仅仅是text+data+bss。

```
full-coredump
```

##### coredump-size unlimited | <n>G | <n>M | <n>K | <n>

设置coredump文件的最大大小。输入值可以以GB，MB，KB或字节设置，也可以设置为“无限制”。

```
coredump-size unlimited
```

##### cli-listen <ipaddress:port> | <socket-path>

绑定CLI以侦听地址localhost上的TCP端口5002。接受ipaddress:port形式或文件系统路径；在后一种情况下，将打开本地Unix套接字。默认的“startup.conf”文件是打开套接字“/run/vpp/cli.sock”。

```
cli-listen localhost:5002
cli-listen /run/vpp/cli.sock
```

##### cli-line-mode

在stdin上禁用逐字符I/O。与emacs M-x gud-gdb结合使用时很有用。

```
cli-line-mode
```

##### cli-prompt <string>

配置CLI提示符字符串。

```
cli-prompt vpp-2
```

##### cli-history-limit <n>

限制历史命令行数，0表示禁用历史命令行，默认值为50。

```
cli-history-limit 100
```

##### cli-no-banner

禁止显示从stdin和telnet登录后显示的banner。

```
cli-no-banner
```

##### cli-no-pager

禁用输出页。

```
cli-no-pager
```

##### cli-pager-buffer-limit <n>

限制输出页的缓冲区行数，0表示禁用页，默认为100000。

```
cli-pager-buffer-limit 5000
```

##### runtime-dir <dir>

设置运行时目录，这是某些文件（如套接字文件）的默认位置。默认目录是基于启动VPP的用户ID，通常“root”用户，其默认为“/run/vpp/”，其他用户默认为“/run/user/<uid>/vpp/”。

```
runtime-dir /tmp/vpp
```

##### poll-sleep-usec <n>

在主循环轮询之间添加固定睡眠。默认值为0，即不休眠。

```
poll-sleep-usec 100
```

##### pidfile <filename>

在给定的文件名中写入主线程的pid。

```
pidfile /run/vpp/vpp1.pid
```

#### api-trace节

尝试了解控制平面试图要求转发平面执行的操作时，跟踪(trace)，转储(dump)和重放(replay)控制平面API跟踪的功能意义巨大。

通常，只需启用API消息跟踪方案即可：

```
api-trace {
   api-trace on
}
```

##### on | enable

从开始就启用API跟踪捕获，并在应用程序异常终止时安排API跟踪的事后转储。默认情况下，（循环）跟踪缓冲区将配置为捕获256K跟踪。默认的“startup.conf”文件启用了跟踪，除非有很强的理由，否则应保持启用状态。

```
on
```

##### nitems <n>

配置循环跟踪缓冲区以包含最近的<n>条目。默认情况下，跟踪缓冲区捕获最后收到的256K API消息。

```
nitems 524288
```

##### save-api-table <filename>

转储API消息表到/tmp/<filename>中。

```
save-api-table apiTrace-07-04.txt
```

#### api-segment节

这些值用以控制vpp二进制API接口的各个方面。

默认配置如下：

```
api-segment {
  gid vpp
}
```

##### prefix <path>

将前缀设置为用于共享内存（SHM）段的名称的前缀。默认值为空，这意味着共享内存段直接在SHM目录“/dev/shm”中创建。值得注意的是，在许多系统上，“/dev/shm”是指向文件系统中其他位置的符号链接。Ubuntu将其链接到“/run/shm”。

```
prefix /run/shm
```

##### uid <number | name>

用于设置共享内存段所有权的用户ID或名称 默认为启动VPP的用户（可能是root）。

```
uid root
```

##### gid <number | name>

用于设置共享内存段所有权的组ID或名称。默认为启动VPP的同一组，可能是root。

```
gid vpp
```

**以下参数仅应由熟悉VPP互通原理的人员设置。**

##### baseva <x>

设置SVM全局区域（global region）的基地址。如果未设置，则在AArch64上，代码将尝试确定基地址，所有其他默认值为0x30000000。

```
baseva 0x20000000
```

##### global-size <n>G | <n>M | <n>

设置全局内存大小，所有路由器实例之间共享的内存，数据包缓冲区等。如果未设置，则默认为64M。输入值可以GB，MB或字节设置。

```
global-size 2G
```

##### global-pvt-heap-size <n>M | size <n>

设置全局VM私有堆的大小。如果未设置，则默认为128k。输入值可以以MB或字节为单位设置。

```
global-pvt-heap-size size 262144
```

##### api-pvt-heap-size <n>M | size <n>

设置api私有堆的大小。如果未设置，则默认为128k。输入值可以以MB或字节为单位设置。

```
api-pvt-heap-size 1M
```

##### api-size <n>M | <n>G | <n>

设置API区域的大小。如果未设置，则默认为16M。输入值可以GB，MB或字节设置。

```
api-size 64M
```

#### socksvr节

启用处理二进制API消息的Unix域套接字。参见…/vlibmemory/socket_api.c。如果未设置此参数，则vpp不会通过套接字处理二进制API消息。

```
socksvr {
   # Explicitly name a socket file
   socket-name /run/vpp/api.sock
   or
   # Use defaults as described below
   default
}
```

“default”关键字指示vpp以root身份运行时使用/run/vpp/api.sock，否则，则使用/run/user/<uid>/api.sock。

#### cpu节

在VPP中，有一个主线程，用户可以选择创建工作线程。主线程和工作线程可以手动或自动绑定到CPU核上。

```
cpu {
   main-core 1
   corelist-workers 2-3,18-19
}
```

##### **手动绑定线程到CPU核上**

##### main-core

设置运行主线程的逻辑CPU核，如果未设置主核，则VPP将使用核1（如果有）。

```
main-core 1
```

##### corelist-workers

设置工作线程使用的逻辑CPU核。

```
corelist-workers 2-3,18-19
```

##### **自动绑定线程到CPU核上**

##### skip-cores number

设置要跳过的CPU核数（1…N-1），跳过的CPU核不用于绑定主线程和工作线程。

主线程自动绑定到第一个可用的CPU核上，工作线程固定在分配给主线程的核心之后的下一个空闲CPU核。

```
skip-cores 4
```

##### workers number

指定要创建的工作线程数，并将工作线程绑定到N个连续的CPU核上，同时跳过“skip-cores”个CPU核和主线程的CPU核。

```
workers 2
```

##### scheduler-policy other | batch | idle | fifo | rr

设置主线程和工作线程的调度策略和优先级。

调度策略选项包括：其他（SCHED_OTHER），批量（SCHED_BATCH），空闲（SCHED_IDLE），FIFO（SCHED_FIFO），rr（SCHED_RR）。

```
scheduler-policy fifo
```

##### schedule-priority number

调度优先级仅用于“实时策略（fifo和rr）”，并且必须在特定策略支持的优先级范围内。

```
schedule-priority 50
```

#### buffers节

```
buffers {
   buffers-per-numa 128000
   default data-size 2048
}
```

##### buffers-per-numa number

增加分配的缓冲区数，只有在具有大量接口和辅助线程的情况下才需要。值是针对每个numa节点的，默认值为16384（如果未特权运行则为8192）。

```
buffers-per-numa 128000
```

##### default data-sze number

缓冲数据区域的大小，默认为2048。

```
default data-size 2048
```

#### dpdk节

```
dpdk {
   dev default {
      num-rx-desc 512
      num-tx-desc 512
   }

   dev 0000:02:00.1 {
      num-rx-queues 2
      name eth0
   }
}
```

##### dev <pci-dev> | default { .. }

将特定的PCI设备列入白名单（例如，尝试驱动）。PCI-dev是形式为“DDDD:BB:SS.F”的字符串，其中：

DDDD = 域（Domain）
BB = 总线编号（Bus Number）
SS = 插槽号（Slot Number）
F = 功能（Function）

如果使用了default关键字，则default中的值将应用于所有设备。

此格式与Linux sysfs树（即/sys/bus/pci/devices）中用于PCI设备目录名称的格式相同。

```
dpdk {
   dev default {
      num-rx-desc 512
      num-tx-desc 512
   }
```

##### dev <pci-dev> { .. }

通过指定PCI地址将特定接口列入白名单。通过指定PCI地址将特定接口列入白名单时，还可以指定其他自定义参数。有效选项包括：

```
dev 0000:02:00.0
dev 0000:03:00.0
```

##### blacklist <pci-dev>

通过指定PCI供应商将特定设备类型列入黑名单：设备白名单条目优先。

```
blacklist 8086:10fb
```

##### name interface-name

设置接口名称。

```
dev 0000:02:00.1 {
   name eth0
}
```

##### num-rx-queues <n>

接收队列数。配置后，同时了启用RSS。预设值为1。

```
dev 0000:02:00.1 {
   num-tx-queues <n>
}
```

##### num-tx-queues <n>

传输队列数。默认值等于工作线程数，如果没有工作线程，则默认为1。

```
dev 000:02:00.1 {
   num-tx-queues <n>
}
```

##### num-rx-desc <n>

接收环中描述符的数量。增加或减少数量可能会影响性能。默认值为1024。

```
dev 000:02:00.1 {
   num-rx-desc <n>
}
```

##### vlan-strip-offload on | off

接口的VLAN Strip卸载模式。默认情况下，除了使用ENIC驱动程序的VIC外，所有NIC的VLAN剥离均关闭，ENIC驱动程序默认情况下启用了VLAN剥离。

```
dev 000:02:00.1 {
   vlan-strip-offload on|off
}
```

##### uio-driver driver-name

更改VPP使用的UIO驱动程序，可选项有：igb_uio，vfio-pci，uio_pci_generic或auto（默认）。

```
uio-driver vfio-pci
```

##### no-multi-seg

禁用多段缓冲区，提高性能，但也同时禁用了巨型（Jumbo）MTU的支持。

```
no-multi-seg
```

##### socket-mem <n>

更改每个CPU插槽的大页分配，仅在需要大量mbuf时才需要。每个检测到的CPU插槽的默认值为256M。

```
socket-mem 2048,2048
```

##### no-tx-checksum-offload

禁用UDP/TCP TX校验和卸载。通常需要使用更快的矢量PMD（和no-multi-seg配合）。

```
no-tx-checksum-offload
```

##### enable-tcp-udp-checksum

启用UDP/TCP TX校验和卸载，这是“no-tx-checksum-offload”的反向选项。

```
enable-tcp-udp-checksum
```

#### plugins节

配置vpp插件：

```
plugins {
   path /ws/vpp/build-root/install-vpp-native/vpp/lib/vpp_plugins
   plugin dpdk_plugin.so enable
}
```

##### path pathname

根据vpp插件存放目录调整插件路径。

```
path /ws/vpp/build-root/install-vpp-native/vpp/lib/vpp_plugins
```

##### plugin plugin-name | default enable | disable

默认禁止所有的插件，然后选择启用的插件。

```
plugin default disable
plugin dpdk_plugin.so enable
plugin acl_plugin.so enable
```

模式启用所有的插件，然后选择禁用的插件。

```
plugin dpdk_plugin.so disable
plugin acl_plugin.so disable
```

#### statseg节

```
statseg {
   per-node-counters on
 }
```

##### socket-name <filename>

统计段的默认套接字“/run/vpp/stats.sock”。

```
socket-name /run/vpp/stats.sock
```

##### size <nnn>[KMG]

统计段的大小，默认32MB。

```
size 1024M
```

##### per-node-counters on | off

默认值为none。

```
per-node-counters on
```

##### update-interval <f64-seconds>

设置统计段搜集/更新的时间间隔。

```
update-interval 300
```

### 一些高级配置参数
#### acl-plugin节
#### api-queue节
#### cj节
#### dns节
#### heapsize节
#### ip节
#### ip6节
#### l2learn节
#### l2tp节
#### logging节
#### mactime节

### "map"参数
#### nat节 
#### oam节
#### physmem节
#### tapcli节
#### tcp节
#### tls节
#### tuntap节
#### vhost-user节
#### vlib节