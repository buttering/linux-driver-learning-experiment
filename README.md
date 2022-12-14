# LINUX 设备驱动程序学习测试代码

- [LINUX 设备驱动程序学习测试代码](#linux-设备驱动程序学习测试代码)
  - [项目说明](#项目说明)
  - [项目结构](#项目结构)
  - [实验运行方法](#实验运行方法)
    - [1.hello\_driver.c](#1hello_driverc)
    - [2. proc\_interface.c](#2-proc_interfacec)
    - [3. wait\_wakeup\_ioctl.c](#3-wait_wakeup_ioctlc)
    - [4. poll\_select.c](#4-poll_selectc)
    - [5. async.c](#5-asyncc)
  - [实验环境](#实验环境)

## 项目说明

本项目展示作者研读《LINUX设备驱动程序》所编写的所有测试代码。
已实现驱动功能：

1. 驱动程序的挂载、卸载
2. 驱动程序的注册
3. 对驱动程序的基本系统调用
4. 在/proc下挂载驱动程序
5. 阻塞操作
6. ioctl调用
7. poll调用
8. async异步通知

## 项目结构

每个文件夹是一个独立demo，分别完成对一个或多个功能的实验。

文件夹下文件作用如下：

1. {foldername}.c
   驱动程序，是项目的核心文件。
2. test_xxx.c
   测试文件，必要时编译为用户态应用程序以对驱动进行测试。
3. Makefile
   用于控制编译驱动程序。
4. test.sh
   测试脚本，完整完成驱动程序的挂载、调用，测试文件的编译和调用。
5. clean.sh
   清理脚本，完成驱动程序的卸载和编译文件的清理。

## 实验运行方法

### 1.hello_driver.c

本文件完成驱动程序基本的注册、挂载和卸载，并测试file_operations结构体的使用(读取、写入操作）。

1)进入目录

```shell
cd 1.hello_driver
chmod +x test.sh
chmod +x clean.sh
```

2)编译驱动

```shell
make
```

3)运行测试文件

```shell
sh test.sh hello_driver
```

4)运行结果

```shell
❯ sh test.sh hello_driver
module: hello_driver
device: hello_dev
[ 2315.228068] Goodbye, World!
---OLD DRIVER CLEARED---

kernal: 5.15.0-50-generic
load driver: 506 hello_dev
load device node: crw-r--r-- 1 root root 506, 0 10月 25 14:50 /dev/hello_dev
device node change group: crw-rw-r-- 1 root wang 506, 0 10月 25 14:50 /dev/hello_dev
[ 2315.331563] Hello, world
[ 2315.331571] The process is "insmod" (pid 10133)
[ 2315.331575] Hello, Mom
[ 2315.331576] Hello, Mom
[ 2315.331578] Hello, Mom
[ 2315.331581] dev major = 506, minor = 0
向驱动读取数据
App read data:This is the kernel data
[ 2315.557830] dev file opened
[ 2315.557842] [read task]count=50
[ 2315.557845] [read data]output string: This is the kernel data
[ 2315.557968] dev file closed
向驱动写入数据
[ 2315.576305] dev file opened
[ 2315.576322] [write data]input string: This is user data!
[ 2315.576328] dev file closed
```

5)卸载和清理驱动

```shell
sh ./clean.sh hello_driver
make clean
```

### 2. proc_interface.c

测试proc接口的挂载操作，并尝试从/proc接口中读取信息。

1)进入目录并授权

```makefile
cd 2.proc_interface
chmod +x test.sh
chmod +x clean.sh
```

2)编译驱动并运行测试脚本

```shell
make
```

3)运行测试脚本

```shell
sh test.sh proc_interface
```

4)运行结果

```shell
❯ sh test.sh proc_interface
module: proc_interface
device: hello_dev
proc: hello_proc
rm: 无法删除 '/dev/hello_dev': 没有那个文件或目录
rmmod: ERROR: Module proc_interface is not currently loaded
---OLD DRIVER CLEARED---

kernal: 5.15.0-50-generic
load driver: 506 hello_dev
load device node: crw-r--r-- 1 root root 506, 0 10月 25 19:33 /dev/hello_dev
proc information is: This is a /proc interface output.
[ 1708.649623] unregister_chrdev_region
[ 1934.897420] Hello, world
[ 1934.897425] The process is "insmod" (pid 10126)
[ 1934.897428] dev major = 506, minor = 0
[ 1934.926393] proc interface is called

```

5)卸载和清理驱动

```shell
sh ./clean.sh proc_interface
make clean
```

### 3. wait_wakeup_ioctl.c

测试linux内核的阻塞与唤醒操作和原子化操作，以及测试ioctl调用

1)进入目录并授权测试脚本

```makefile
cd 3.wait_wakeup_ioctl
chmod +x test.sh
chmod +x clean.sh
```

2)编译驱动

```shell
make
```

3)运行测试脚本

```shell
sh test.sh wait_wakeup_ioctl
```

4)运行结果

```shell
❯ sh test.sh wait_wakeup_ioctl
module: wait_wakeup_ioctl
device: wwdev
testfile: test_ioctl
rm: 无法删除 '/dev/wwdev': 没有那个文件或目录
rmmod: ERROR: Module wait_wakeup_ioctl is not currently loaded
rm: 无法删除 'test_ioctl': 没有那个文件或目录
---OLD DRIVER CLEARED---

kernal: 5.15.0-50-generic
load driver: 506 wwdev
load device node: crw-r--r-- 1 root root 506, 0 10月 25 20:13 /dev/wwdev
device node change group: crw-rw-r-- 1 root wang 506, 0 10月 25 20:13 /dev/wwdev

sub process read operation to wait... 
write operation to wakeup sub process... 
[ 3734.409463] [writer]process 14072 (sh) awakening the readers...
[ 3734.410665] [reader]process 14103 (cat) going to sleep
[ 3734.410674] [reader]awoken 14103 (cat)

传入指针由ioctl修改值
ioctl read result: 55
[ 3734.505092] ioctl called!
[ 3734.505095] [ioctl read] output number to ptr: 55
直接接受ioctl调用返回值
ioctl read result: 55
[ 3734.521584] ioctl called!
[ 3734.521588] [ioctl read] output number by return: 55
```

5)继续尝试和写入读取操作

通过读取操作实现阻塞，可开启多个终端运行

```shell
cat /dev/wwdev
```

另开一个终端，通过写入操作，实现进程的唤醒，每次调用唤醒一个被阻塞的进程

```shell
echo 1 > /dev/wwdev
dmesg
```

6)卸载和清理驱动

```shell
sh clean.sh wait_wakeup_ioctl
make clean
```

### 4. poll_select.c

实验了poll系统调用的各种情况，使用class_create完成自动节点注册和多节点挂载

1)进入目录，提供权限

```shell
cd poll_select
chmod +x test.sh
chmod +x clean.sh
```

2)编译驱动

```shell
make
```

3)运行测试脚本

```shell
sh test.sh poll_select
```

4)运行结果

```shell
❯ sh test.sh poll_select
module: poll_select
device: dev
[ 3757.121941] this is dev_module_cleanup
---OLD DRIVER CLEARED---

kernal: 5.15.0-50-generic
load driver: poll_select            16384  0
load device node: 总用量 0
crw------- 1 root root 506, 0 10月 24 21:40 dev0
crw------- 1 root root 506, 1 10月 24 21:40 dev1
crw------- 1 root root 506, 2 10月 24 21:40 dev2
crw------- 1 root root 506, 3 10月 24 21:40 dev3
device node change group: 总用量 0
crw-rw-r-- 1 root wang 506, 0 10月 24 21:40 dev0
crw-rw-r-- 1 root wang 506, 1 10月 24 21:40 dev1
crw-rw-r-- 1 root wang 506, 2 10月 24 21:40 dev2
crw-rw-r-- 1 root wang 506, 3 10月 24 21:40 dev3
[ 3757.169212] Test call poll
[ 3757.169214] The process is "insmod" (pid 10607)
[ 3757.169217] dev major = 506, minor = 0, count = 4

poll调用前资源已就绪，只调用poll一次
[ 3757.254376] [write]resources are parpared (switch flag to 1)
get data: This is the kernal data!
[ 3757.261365] poll interface has been called
[ 3757.261367] resource are ready

不进行唤醒，且规定时间内(5s)资源未就绪，在调用时和超时时各调用一次
[ 3757.268024] [write]resources are scarded (switch flag to 0)
not fds get ready, timeout!
[ 3757.274508] poll interface has been called
[ 3757.274510] resource are not ready
[ 3762.277128] poll interface has been called
[ 3762.277138] resource are not ready

主动进行唤醒，且唤醒时资源已就绪 , 在调用时和被唤醒时各调用一次
[ 3762.304176] [write]resources are scarded (switch flag to 0)
get data: This is the kernal data!
[ 3762.318506] poll interface has been called
[ 3762.318509] resource are not ready
[ 3762.328979] [write]resources are parpared (switch flag to 1) and arouse initiatively
[ 3762.329023] poll interface has been called
[ 3762.329025] resource are ready

主动进行唤醒，但唤醒时资源未就绪，在调用时和被唤醒时各调用一次，会保持阻塞状态直到超时或再次被唤醒
[ 3762.336905] [write]resources are scarded (switch flag to 0)
[ 3762.345125] poll interface has been called
[ 3762.345127] resource are not ready
[ 3762.355501] [write]resources are scarded (switch flag to 0) and arouse initiatively
[ 3762.355545] poll interface has been called
[ 3762.355547] resource are not ready
not fds get ready, timeout!
```

5)清理环境

```shell
sh clean.sh poll_select
make clean
```

### 5. async.c

1)进入目录，提供权限

```shell
cd async
chmod +x test.sh
chmod +x clean.sh
```

2)编译驱动

```shell
make
```

3)运行测试脚本

```shell
sh test.sh async
```

4)运行结果

```shell
❯ sh test.sh async
module: async
device: asyncdevice
[  745.014143] rockchip_canfd fe580000.can can0: rockchip_canfd_get_berr_counter RX_ERR_CNT=0x00000000, TX_ERR_CNT=0x00000000
[  745.106335] rockchip_canfd fe580000.can can0: rockchip_canfd_get_berr_counter RX_ERR_CNT=0x00000000, TX_ERR_CNT=0x00000000
[  745.159886] [exit]module clean up!
---OLD DRIVER CLEARED---

kernal: 4.19.193-47-rockchip-g0f91d24e217c
load driver: async                  16384  0
load device node: crw------- 1 root root 238, 0 Nov 29 10:46 /dev/asyncdevice
device node change group: crw-rw-r-- 1 root wang 238, 0 Nov 29 10:46 /dev/asyncdevice
[  745.328000] rockchip_canfd fe580000.can can0: rockchip_canfd_get_berr_counter RX_ERR_CNT=0x00000000, TX_ERR_CNT=0x00000000
[  745.387800] [init]Test async message
[  745.387824] [init]The process is "insmod" (pid 4147)
[  745.387834] [init]dev major = 238, minor = 0
[  745.450891] rockchip_canfd fe580000.can can0: rockchip_canfd_get_berr_counter RX_ERR_CNT=0x00000000, TX_ERR_CNT=0x00000000
[  745.533144] rockchip_canfd fe580000.can can0: rockchip_canfd_get_berr_counter RX_ERR_CNT=0x00000000, TX_ERR_CNT=0x00000000
----驱动加载成功!----
/proc/device field: 238 asyncdev
/sys/class node: drwxr-xr-x 2 root root 0 Nov 29 10:46 asyncclass
/dev node: crw-rw-r--  1 root wang    238,   0 Nov 29 10:46 asyncdevice
可用的kill信号: 
 1 HUP      2 INT      3 QUIT     4 ILL      5 TRAP     6 ABRT     7 BUS
 8 FPE      9 KILL    10 USR1    11 SEGV    12 USR2    13 PIPE    14 ALRM
15 TERM    16 STKFLT  17 CHLD    18 CONT    19 STOP    20 TSTP    21 TTIN
22 TTOU    23 URG     24 XCPU    25 XFSZ    26 VTALRM  27 PROF    28 WINCH
29 POLL    30 PWR     31 SYS     

运行用户态程序，监听SIGIO信号。
[app]Pid: 4209
[app]Driver file open successed: 3
[app]Listion signal: SIGIO(29)
[app]Listion signal: SIGUSR1(10)
[  746.172878] [fasync]regist async

向驱动写入信息，驱动向用户程序发送SIGIO信号。
[app]receive signal: 29
[  746.445004] [write]Get data: User msg
[  746.445029] [write]Send async signal: 29

使用kill向用户程序发送SIGUSR1信号
[app]receive signal: 10
```

5)清理环境

```shell
sh clean.sh poll_select
make clean
```

## 实验环境

操作系统：
实验1-4：Linux Li-Jiancheng-Ubuntu 5.15.0-50-generic #56~20.04.1-Ubuntu SMP Tue Sep 27 15:51:29 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
实验5：4.19.193-47-rockchip-g0f91d24e217c

IDE：VSCode
