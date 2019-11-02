---
title: Kubernetes排错指南
date: 2019-11-02 08:12:50
top:
categories: 持续集成
tags:
- docker
- k8s
---

> 转载自：https://github.com/imroc/kubernetes-troubleshooting-guide

## 分析 ExitCode 定位 Pod 异常退出原因

使用 `kubectl describe pod <pod name>` 查看异常 pod 的状态:

```
Containers:
  kubedns:
    Container ID:  docker://5fb8adf9ee62afc6d3f6f3d9590041818750b392dff015d7091eaaf99cf1c945
    Image:         ccr.ccs.tencentyun.com/library/kubedns-amd64:1.14.4
    Image ID:      docker-pullable://ccr.ccs.tencentyun.com/library/kubedns-amd64@sha256:40790881bbe9ef4ae4ff7fe8b892498eecb7fe6dcc22661402f271e03f7de344
    Ports:         10053/UDP, 10053/TCP, 10055/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      --domain=cluster.local.
      --dns-port=10053
      --config-dir=/kube-dns-config
      --v=2
    State:          Running
      Started:      Tue, 27 Aug 2019 10:58:49 +0800
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Tue, 27 Aug 2019 10:40:42 +0800
      Finished:     Tue, 27 Aug 2019 10:58:27 +0800
    Ready:          True
    Restart Count:  1
```

在容器列表里看 `Last State` 字段，其中 `ExitCode` 即程序上次退出时的状态码，如果不为 0，表示异常退出，我们可以分析下原因。

### 退出状态码的区间

- 必须在 0-255 之间
- 0 表示正常退出
- 外界中断将程序退出的时候状态码区间在 129-255，(操作系统给程序发送中断信号，比如 `kill -9` 是 `SIGKILL`，`ctrl+c` 是 `SIGINT`)
- 一般程序自身原因导致的异常退出状态区间在 1-128 (这只是一般约定，程序如果一定要用129-255的状态码也是可以的)

假如写代码指定的退出状态码时不在 0-255 之间，例如: `exit(-1)`，这时会自动做一个转换，最终呈现的状态码还是会在 0-255 之间。我们把状态码记为 `code`

- 当指定的退出时状态码为负数，那么转换公式如下:

```
256 - (|code| % 256)
```

- 当指定的退出时状态码为正数，那么转换公式如下:

```
code % 256
```

### 常见异常状态码

- 137 (被  `SIGKILL` 中断信号杀死)

  - 此状态码一般是因为 pod 中容器内存达到了它的资源限制(`resources.limits`)，一般是内存溢出(OOM)，CPU达到限制只需要不分时间片给程序就可以。因为限制资源是通过 linux 的 cgroup 实现的，所以 cgroup 会将此容器强制杀掉，类似于 `kill -9`，此时在 `describe pod` 中可以看到 Reason 是 `OOMKilled`

  - 还可能是宿主机本身资源不够用了(OOM)，内核会选取一些进程杀掉来释放内存

  - 不管是 cgroup 限制杀掉进程还是因为节点机器本身资源不够导致进程死掉，都可以从系统日志中找到记录:

    > ubuntu 的系统日志在 `/var/log/syslog`，centos 的系统日志在 `/var/log/messages`，都可以用 `journalctl -k` 来查看系统日志

  - 也可能是 livenessProbe (存活检查) 失败，kubelet 杀死的 pod

  - 还可能是被恶意木马进程杀死

- 1 和 255

  - 这种可能是一般错误，具体错误原因只能看容器日志，因为很多程序员写异常退出时习惯用 `exit(1)` 或 `exit(-1)`，-1 会根据转换规则转成 255

### 状态码参考

这里罗列了一些状态码的含义：[Appendix E. Exit Codes With Special Meanings](http://tldp.org/LDP/abs/html/exitcodes.html)

### Linux 标准中断信号

Linux 程序被外界中断时会发送中断信号，程序退出时的状态码就是中断信号值加上 128 得到的，比如 `SIGKILL` 的中断信号值为 9，那么程序退出状态码就为 9+128=137。以下是标准信号值参考：

```
Signal     Value     Action   Comment
──────────────────────────────────────────────────────────────────────
SIGHUP        1       Term    Hangup detected on controlling terminal
                                     or death of controlling process
SIGINT        2       Term    Interrupt from keyboard
SIGQUIT       3       Core    Quit from keyboard
SIGILL        4       Core    Illegal Instruction
SIGABRT       6       Core    Abort signal from abort(3)
SIGFPE        8       Core    Floating-point exception
SIGKILL       9       Term    Kill signal
SIGSEGV      11       Core    Invalid memory reference
SIGPIPE      13       Term    Broken pipe: write to pipe with no
                                     readers; see pipe(7)
SIGALRM      14       Term    Timer signal from alarm(2)
SIGTERM      15       Term    Termination signal
SIGUSR1   30,10,16    Term    User-defined signal 1
SIGUSR2   31,12,17    Term    User-defined signal 2
SIGCHLD   20,17,18    Ign     Child stopped or terminated
SIGCONT   19,18,25    Cont    Continue if stopped
SIGSTOP   17,19,23    Stop    Stop process
SIGTSTP   18,20,24    Stop    Stop typed at terminal
SIGTTIN   21,21,26    Stop    Terminal input for background process
SIGTTOU   22,22,27    Stop    Terminal output for background process
```

### C/C++ 退出状态码

`/usr/include/sysexits.h` 试图将退出状态码标准化(仅限 C/C++):

```
#define EX_OK           0       /* successful termination */

#define EX__BASE        64      /* base value for error messages */

#define EX_USAGE        64      /* command line usage error */
#define EX_DATAERR      65      /* data format error */
#define EX_NOINPUT      66      /* cannot open input */
#define EX_NOUSER       67      /* addressee unknown */
#define EX_NOHOST       68      /* host name unknown */
#define EX_UNAVAILABLE  69      /* service unavailable */
#define EX_SOFTWARE     70      /* internal software error */
#define EX_OSERR        71      /* system error (e.g., can't fork) */
#define EX_OSFILE       72      /* critical OS file missing */
#define EX_CANTCREAT    73      /* can't create (user) output file */
#define EX_IOERR        74      /* input/output error */
#define EX_TEMPFAIL     75      /* temp failure; user is invited to retry */
#define EX_PROTOCOL     76      /* remote error in protocol */
#define EX_NOPERM       77      /* permission denied */
#define EX_CONFIG       78      /* configuration error */

#define EX__MAX 78      /* maximum listed value */
```





## 使用 systemtap 定位疑难杂症

### Ubuntu

安装 systemtap:

```
apt install -y systemtap
```

运行 `stap-prep` 检查还有什么需要安装:

```
$ stap-prep
Please install linux-headers-4.4.0-104-generic
You need package linux-image-4.4.0-104-generic-dbgsym but it does not seem to be available
 Ubuntu -dbgsym packages are typically in a separate repository
 Follow https://wiki.ubuntu.com/DebuggingProgramCrash to add this repository

apt install -y linux-headers-4.4.0-104-generic
```

提示需要 dbgsym 包但当前已有软件源中并不包含，需要使用第三方软件源安装，下面是 dbgsym 安装方法(参考官方wiki: <https://wiki.ubuntu.com/Kernel/Systemtap>):

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C8CAB6595FDFF622

codename=$(lsb_release -c | awk  '{print $2}')
sudo tee /etc/apt/sources.list.d/ddebs.list << EOF
deb http://ddebs.ubuntu.com/ ${codename}      main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-security main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-updates  main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-proposed main restricted universe multiverse
EOF

sudo apt-get update
```

配置好源后再运行下 `stap-prep`:

```
$ stap-prep
Please install linux-headers-4.4.0-104-generic
Please install linux-image-4.4.0-104-generic-dbgsym
```

提示需要装这两个包，我们安装一下:

```
apt install -y linux-image-4.4.0-104-generic-dbgsym
apt install -y linux-headers-4.4.0-104-generic
```

### CentOS

安装 systemtap:

```
yum install -y systemtap
```

默认没装 `debuginfo`，我们需要装一下，添加软件源 `/etc/yum.repos.d/CentOS-Debug.repo`:

```
[debuginfo]
name=CentOS-$releasever - DebugInfo
baseurl=http://debuginfo.centos.org/$releasever/$basearch/
gpgcheck=0
enabled=1
protect=1
priority=1
```

执行 `stap-prep` (会安装 `kernel-debuginfo`)

最后检查确保 `kernel-debuginfo` 和 `kernel-devel` 均已安装并且版本跟当前内核版本相同，如果有多个版本，就删除跟当前内核版本不同的包(通过`uname -r`查看当前内核版本)。

重点检查是否有多个版本的 `kernel-devel`:

```
$ rpm -qa | grep kernel-devel
kernel-devel-3.10.0-327.el7.x86_64
kernel-devel-3.10.0-514.26.2.el7.x86_64
kernel-devel-3.10.0-862.9.1.el7.x86_64
```

如果存在多个，保证只留跟当前内核版本相同的那个，假设当前内核版本是 `3.10.0-862.9.1.el7.x86_64`，那么使用 rpm 删除多余的版本:

```
rpm -e kernel-devel-3.10.0-327.el7.x86_64 kernel-devel-3.10.0-514.26.2.el7.x86_64
```

### 使用 systemtap 揪出杀死容器的真凶

Pod 莫名其妙被杀死? 可以使用 systemtap 来监视进程的信号发送，原理是 systemtap 将脚本翻译成 C 代码然后调用 gcc 编译成 linux 内核模块，再通过 `modprobe` 加载到内核，根据脚本内容在内核做各种 hook，在这里我们就 hook 一下信号的发送，找出是谁 kill 掉了容器进程。

首先，找到被杀死的 pod 又自动重启的容器的当前 pid，describe 一下 pod:

```
    ......
    Container ID:  docker://5fb8adf9ee62afc6d3f6f3d9590041818750b392dff015d7091eaaf99cf1c945
    ......
    Last State:     Terminated
      Reason:       Error
      Exit Code:    137
      Started:      Thu, 05 Sep 2019 19:22:30 +0800
      Finished:     Thu, 05 Sep 2019 19:33:44 +0800
```

拿到容器 id 反查容器的主进程 pid:

```
$ docker inspect -f "{{.State.Pid}}" 5fb8adf9ee62afc6d3f6f3d9590041818750b392dff015d7091eaaf99cf1c945
7942
```

通过 `Exit Code` 可以看出容器上次退出的状态码，如果进程是被外界中断信号杀死的，退出状态码将在 129-255 之间，137 表示进程是被 SIGKILL 信号杀死的，但我们从这里并不能看出是被谁杀死的。

如果问题可以复现，我们可以使用下面的 systemtap 脚本来监视容器是被谁杀死的(保存为`sg.stp`):

```
global target_pid = 7942
probe signal.send{
  if (sig_pid == target_pid) {
    printf("%s(%d) send %s to %s(%d)\n", execname(), pid(), sig_name, pid_name, sig_pid);
    printf("parent of sender: %s(%d)\n", pexecname(), ppid())
    printf("task_ancestry:%s\n", task_ancestry(pid2task(pid()), 1));
  }
}
```

- 变量 `pid` 的值替换为查到的容器主进程 pid

运行脚本:

```
stap sg.stp
```

当容器进程被杀死时，脚本捕捉到事件，执行输出:

```
pkill(23549) send SIGKILL to server(7942)
parent of sender: bash(23495)
task_ancestry:swapper/0(0m0.000000000s)=>systemd(0m0.080000000s)=>vGhyM0(19491m2.579563677s)=>sh(33473m38.074571885s)=>bash(33473m38.077072025s)=>bash(33473m38.081028267s)=>bash(33475m4.817798337s)=>pkill(33475m5.202486630s)
```

通过观察 `task_ancestry` 可以看到杀死进程的所有父进程，在这里可以看到有个叫 `vGhyM0` 的奇怪进程名，通常是中了木马，需要安全专家介入继续排查。



## 非root用户执行docker命令时提示permission denied

## 原因

摘自docker mannual上的一段话

```
Manage Docker as a non-root user

The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user root and other users can only access it using sudo. The docker daemon always runs as the root user.

If you don’t want to use sudo when you use the docker command, create a Unix group called docker and add users to it. When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the docker group.
```

大概的意思就是：docker进程使用Unix Socket而不是TCP端口。而默认情况下，Unix socket属于root用户，需要root权限才能访问。

### 解决方法1

使用sudo获取管理员权限，运行docker命令

### 解决方法2

docker守护进程启动的时候，会默认赋予名字为docker的用户组读写Unix socket的权限，因此只要创建docker用户组，并将当前用户加入到docker用户组中，那么当前用户就有权限访问Unix socket了，进而也就可以执行docker相关命令

```
sudo groupadd docker     #添加docker用户组
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
docker ps    #测试docker命令是否可以使用sudo正常使用
```