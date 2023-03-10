
# Linux中的Audit审计工具
## 概述
> Auditd工具可以帮助运维人员审计Linux，分析发生在系统中的发生的事情。Linux 内核有用日志记录事件的能力，包括记录系统调用和文件访问。管理员可以检查这些日志，确定是否存在安全漏洞（如多次失败的登录尝试，或者用户对系统文件不成功的访问）。

audit工作原理

![preview](_media/v2-380baacbb0769d87889c33062bb46de9_r.jpg)

内核audit模块定义了user，task，exit钩子，exclude用于auditd取audit log时进行过滤，过滤掉不感兴趣的event

## 安装
 在centosi中默认已经安装了Audit
```sh
[rpm -aq |grep audit](<[root@bd1 ~]# rpm -aq |grep audit
audit-2.8.5-4.el7.x86_64
audit-libs-2.8.5-4.el7.x86_64>)
```

使用命令systemctrl status auditd可以查看审计是否启动
```sh
[root@bd1 ~]# systemctl status auditd
● auditd.service - Security Auditing Service
   Loaded: loaded (/usr/lib/systemd/system/auditd.service; enabled; vendor prese                                                  t: enabled)
   Active: active (running) since 三 2022-03-23 15:19:48 CST; 6min ago
     Docs: man:auditd(8)
           https://github.com/linux-audit/audit-documentation
  Process: 681 ExecStartPost=/sbin/augenrules --load (code=exited, status=0/SUCC                                                  ESS)
  Process: 676 ExecStart=/sbin/auditd (code=exited, status=0/SUCCESS)
 Main PID: 677 (auditd)
   CGroup: /system.slice/auditd.service
           └─677 /sbin/auditd

3月 23 15:19:48 bd1 augenrules[681]: lost 0
3月 23 15:19:48 bd1 augenrules[681]: backlog 1
3月 23 15:19:48 bd1 augenrules[681]: enabled 1
3月 23 15:19:48 bd1 augenrules[681]: failure 1
3月 23 15:19:48 bd1 augenrules[681]: pid 677
3月 23 15:19:48 bd1 augenrules[681]: rate_limit 0
3月 23 15:19:48 bd1 augenrules[681]: backlog_limit 8192
3月 23 15:19:48 bd1 augenrules[681]: lost 0
3月 23 15:19:48 bd1 augenrules[681]: backlog 1
3月 23 15:19:48 bd1 systemd[1]: Started Security Auditing Service.
```

## 配置文件
配置文件在/etc/audit路径下
```sh
[root@bd1 ~]# ls /etc/audit
auditd.conf  audit.rules  audit-stop.rules  rules.d
```

- auditd.conf文件 ----审计工具的配置文件
- audit.rules文件 ----审计的规则，该文件由/etc/audit/rules.d产生，所以我们的规则一般放置在rules.d文件夹下
- audit-stop.rules文件 ----审计停止规则，该文件定义了停止检测
- rules.d目录 ----定义需要审计的规则，写到文件之后会永久有效

如下是auditd.conf，审计工具配置文件。在该文件中，用中文增加了对应的参数值解释。详细解释，可查阅官方解释：

[audit参数文档](https://linux.die.net/man/8/auditd.conf)

```sh
#
# This file controls the configuration of the audit daemon
#
local_events = yes
write_logs = yes
## 审计日志文件存放路径
log_file = /var/log/audit/audit.log
##审计日志所属用户组
log_group = root
## 写日志时使用的格式。RAW数据以内核中检索到的格式写到日志文件中；NOLOG数据不会写到日志文件中
log_format = RAW
## 多长时间写向日志文件中写一次数据，INCREMENTAL:
flush = INCREMENTAL_ASYNC
# 如果flush为INCREMENTAL 审计守护进程在写到日志文件中前从内核接收的记录数
freq = 50
#单位M，最大日志容量，到达这个容量，时，会执行max_log_file_action指定的动作
max_log_file = 100
## backlog设置值，以便留出日志循环时间，如果没有设置num_logs值，默认为0，表示不循环日志文件
num_logs = 5
## 审计守护进程的优先级，必须是非负数，0表示没有变化
priority_boost = 4
## 调度程序与审计守护进程之间的通信类型，一般不修改
disp_qos = lossy
##当启动这个守护进程时，由守护进程自动启动程序
dispatcher = /sbin/audispd
# 计算机节点名如何插入到审计数据流中
name_format = NONE
##name = mydomain
## 当达到max_log_file的日志文件大小时采取的动作
max_log_file_action = ROTATE
##单位M，当达到这个数值时，会采用space_left_action参数中的动作
space_left = 75
## 当磁盘空间达到space_left中的值时，采取的动作
space_left_action = SYSLOG
verify_email = yes
## 负责维护审计守护进程和日志的管理员电子邮箱地址
action_mail_acct = root
## 单位M，表示磁盘空间的数量
admin_space_left = 50
## 当自由磁盘空间达到admin_space_left的值时，采取的动作
admin_space_left_action = SUSPEND
## 如果还有审计文件的分区已满，则采取这个动作
disk_full_action = SUSPEND
## 如果在写审计日志或循环日志文件时，检测到错误时采取的动作
disk_error_action = SUSPEND
use_libwrap = yes
##tcp_listen_port = 60
tcp_listen_queue = 5
tcp_max_per_addr = 1
##tcp_client_ports = 1024-65535
tcp_client_max_idle = 0
enable_krb5 = no
krb5_principal = auditd
##krb5_key_file = /etc/audit/audit.key
distribute_network = no
```

## audit的配置规则

audit的规则类型分为3类：

1. 控制规则：控制aurid系统的规则，可以理解为audit的相关配置

2. 文件系统规则：文件监控规则，可以用于监控一个特定的文件或者一个路径，比如对于/etc/passwd的监控，/etc/hosts文件的监控等

3. 系统调用规则：记录特定程序的系统调用规则

### 控制规则

-b 设置在内核中audit缓冲空间的最大值。

-f  这个选项来决定内核如何处理critical erros：0=silent 1=printk 2=panic.默认值为1。

-e 设置使能标志，设置为0，为关闭了audit，设置为1，则开启audit；当设置为2时，表示锁定，一般在设置完其他规则后最后设置，防止其他人修改规则；任何修改规则的行为都会被拒绝，并且记录审计日志，只有当重启系统后，这个使能标志才可以被修改。

```shell
[root@bd1 ~]# vi /etc/audit/audit.rules
## This file is automatically generated from /etc/audit/rules.d
-D
-b 8192
-f 1
## 锁定标识
-e 2 
```

### 文件系统规则

可以通过auditctl 命令设置，也可以定义在rules的文件中

命令：auditctl -w path -p permissions -k key_name

-w : 目录或者文件路径

-p： 描述文件系统监视将触发的权限访问类型，r=读取，w=写入，x=执行，a=属性更改。

-k: 设置审核规则的筛选关键字

例子：

```shell
## 设置策略
[root@bd1 ~]# auditctl -w /etc/passwd -p rwxa -k identity
## 查看策略是否设置成功
[root@bd1 ~]# auditctl -l
-w /etc/passwd -p rwxa -k identity
## 访问审核文件
[root@bd1 ~]# cat /etc/passwd
...
##audit日志文件显示
type=SYSCALL msg=audit(1648025135.905:211): arch=c000003e syscall=2 success=yes exit=3 a0=7ffe0bee07b9 a1=0 a2=1fffffffffff0000 a3=7ffe0bedde60 items=1 ppid=1775 pid=1889 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts1 ses=4 comm="cat" exe="/usr/bin/cat" key="identity"
type=CWD msg=audit(1648025135.905:211):  cwd="/root"
type=PATH msg=audit(1648025135.905:211): item=0 name="/etc/passwd" inode=38525463 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
type=PROCTITLE msg=audit(1648025135.905:211): proctitle=636174002F6574632F706173737764

```

### 系统调用规则

可以通过auditctl 命令设置，也可以定义在rules的文件中

命令：auditctl -a [list,action|action,list] -S [Syscall name or number|all] -F field=value -k key_name

-a: action和list 明确一个事件被记录。action可以为always或者never，list明确出对应的匹配过滤，list可以为：task,exit,user,exclude,filesystem。

-S**:** system_call 明确出系统调用的名字，几个系统调用可以写在一个规则里，如-S xxx -S xxx。系统调用的名字可以在/usr/include/asm/unistd_64.h文件中找到。

-F: field=value 作为附加选项，修改规则以匹配特定架构、GroupID，ProcessID等的事件。具体有哪些字段，可以参考man linux https://linux.die.net/man/8/auditctl

-k: 设置审核规则的筛选关键字

## auditd工具

- auditctl：auditd默认操作命令
- aureport：生成和查看审计规则的文件
- ausearch：是一个搜索各种事件的工具
- autrce：用于跟踪进程的命令



## 常用策略示例

### 记录系统的日期和时间的修改

```shell
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change
-a always,exit -F arch=b32 -S adjtimex -S settimeofday -S stime -k timechange
-a always,exit -F arch=b64 -S clock_settime -k time-change
-a always,exit -F arch=b32 -S clock_settime -k time-change
-w /etc/localtime -p wa -k time-change
```

### 记录用户和组的修改的事件

```shell
-w /etc/group -p wa -k identity
-w /etc/passwd -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/security/opasswd -p wa -k identity
```

### 记录网络环境修改时间

```shell
-a always,exit -F arch=b64 -S sethostname -S setdomainname -k system-locale
-a always,exit -F arch=b32 -S sethostname -S setdomainname -k system-locale
-w /etc/issue -p wa -k system-locale
-w /etc/issue.net -p wa -k system-locale
-w /etc/hosts -p wa -k system-locale
-w /etc/sysconfig/network -p wa -k system-locale
-w /etc/sysconfig/network-scripts/ -p wa -k system-locale
```

### 记录登录和登出事件

```shell
-w /var/log/lastlog -p wa -k logins
-w /var/run/faillock/ -p wa -k logins
```

### 记录会话启动事件

文件/var/run/utmp跟踪当前登录的所有用户。所有审计记录都将用标识符“session”标记， 可以用who命令读取

文件/var/log/wtmp文件跟踪登录、注销、关机和重新启动事件。

文件/var/log/btmp跟踪失败的登录尝试，可以通过输入命令 ‘/usr/bin/last -f /var/log/btmp’ 读取。所有审核记录都将被标记为标识符“logins”

```shell
-w /var/run/utmp -p wa -k session
-w /var/log/wtmp -p wa -k logins
-w /var/log/btmp -p wa -k logins
```

### 监视对文件权限、属性、所有权和组的更改

在这里有几个附加选项

arch，其中arch可以通过uname -m命令获取具体的arch信息，b32|b64

auid：用户id审计条件，具体可以通过id $用户名 进行查询

在所有情况下，审核记录将只为非系统用户id（auid>=1000）并将忽略守护进程事件（auid=4294967295）。

所有审计记录用标识符“perm_mod”标记

```shell

-a always,exit -F arch=b64 -S chmod -S fchmod -S fchmodat -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b32 -S chmod -S fchmod -S fchmodat -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b64 -S chown -S fchown -S fchownat -S lchown -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b32 -S chown -S fchown -S fchownat -S lchown -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b64 -S setxattr -S lsetxattr -S fsetxattr -S removexattr -S lremovexattr -S fremovexattr -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b32 -S setxattr -S lsetxattr -S fsetxattr -S removexattr -S lremovexattr -S fremovexattr -F auid>=1000 -F auid!=4294967295 -k perm_mod
```

### 记录未授权文件访问尝试

```shell
-a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -S ftruncate -F exit=-EACCES -F auid>=1000 -F auid!=4294967295 -k access
-a always,exit -F arch=b32 -S creat -S open -S openat -S truncate -S ftruncate -F exit=-EACCES -F auid>=1000 -F auid!=4294967295 -k access
-a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -S ftruncate -F exit=-EPERM -F auid>=1000 -F auid!=4294967295 -k access
-a always,exit -F arch=b32 -S creat -S open -S openat -S truncate -S ftruncate -F exit=-EPERM -F auid>=1000 -F auid!=4294967295 -k access
```

### 确保收集使用特权命令

监视特权程序（那些在执行时设置了setuid和/或setgid位的程序）以确定没有权限的用户是否正在运行这些命令。

通过下面命令生成策略

```shell
[root@bd1 ~]# find / -xdev \( -perm -4000 -o -perm -2000 \) -type f | awk '{print "-a always,exit -F path=" $1 " -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged" }'
-a always,exit -F path=/usr/bin/wall -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/fusermount -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/chfn -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/chage -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/chsh -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/gpasswd -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/newgrp -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/mount -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/su -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/sudo -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/umount -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/write -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/crontab -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/pkexec -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/ssh-agent -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/bin/passwd -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/sbin/unix_chkpwd -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/sbin/pam_timestamp_check -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/sbin/netreport -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/sbin/usernetctl -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/sbin/postdrop -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/sbin/postqueue -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/lib/polkit-1/polkit-agent-helper-1 -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/libexec/utempter/utempter -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/libexec/dbus-1/dbus-daemon-launch-helper -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
-a always,exit -F path=/usr/libexec/openssh/ssh-keysign -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
```

### 收集成功挂载磁盘事件

```shell
-a always,exit -F arch=b64 -S mount -F auid>=1000 -F auid!=4294967295 -k mounts
-a always,exit -F arch=b32 -S mount -F auid>=1000 -F auid!=4294967295 -k mount
```

### 确保收集用户的文件删除事件

```shell
-a always,exit -F arch=b64 -S unlink -S unlinkat -S rename -S renameat -F auid>=1000 -F auid!=4294967295 -k delete
-a always,exit -F arch=b32 -S unlink -S unlinkat -S rename -S renameat -F auid>=1000 -F auid!=4294967295 -k delete
```

### 确保收集对系统管理范围（sudoers）的更改

```shell
-w /etc/sudoers -p wa -k scope
-w /etc/sudoers.d/ -p wa -k scope
```

### 监视sudo日志文件

如果系统已正确配置为禁用su命令并强制所有管理员必须先登录，然后使用sudo执行特权命令，

然后所有管理员命令将被记录到/var/log/sudo.log文件.

每当执行命令时，审核事件将被触发为/var/log/sudo.log文件将打开文件进行写入，并执行管理命令将写入日志。

```shell
-w /var/log/sudo.log -p wa -k actions
```

### 确保收集内核模块加载和卸载

```shell
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules
-a always,exit -F arch=b64 -S init_module -S delete_module -k modules
```

### 确保审核配置是不可变的

审核规则不能使用auditctl修改。设置标志“-e2“强制将审核置于不可变模式。进行审核更改只能对系统重新启动。

```shell
-e 2
```

### 安全等保常用配置

```
-a exit,always -F arch=b64 -S umask -S chown -S chmod 
-a exit,always -F arch=b64 -S unlink -S rmdir 
-a exit,always -F arch=b64 -S setrlimit 
-a exit,always -F arch=b64 -S setuid -S setreuid 
-a exit,always -F arch=b64 -S setgid -S setregid 
-a exit,always -F arch=b64 -S sethostname -S setdomainname 
-a exit,always -F arch=b64 -S adjtimex -S settimeofday 
-a exit,always -F arch=b64 -S mount -S _sysctl
 
-w /etc/group -p wa 
-w /etc/passwd -p wa 
-w /etc/shadow -p wa 
-w /etc/sudoers -p wa

-w /etc/ssh/sshd_config

-w /etc/bashrc -p wa   
-w /etc/profile -p wa   
-w /etc/profile.d/   
-w /etc/aliases -p wa   
-w /etc/sysctl.conf -p wa
 
-w /var/log/lastlog
```



## audit日志

主要字段及解释

> type=SYSCALL msg=audit(1648027154.838:333): arch=c000003e syscall=159 success=yes exit=0 a0=7fffca2da2b0 a1=0 a2=68754 a3=55da09a1                                                                      b520 items=0 ppid=1 pid=729 auid=4294967295 uid=998 gid=996 euid=998 suid=998 fsuid=998 egid=996 sgid=996 fsgid=996 tty=(none) ses                                                                      =4294967295 comm="chronyd" exe="/usr/sbin/chronyd" key="time-change"

- type=SYSCALL - 每条日志以type开头，具体可以参见https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/security_guide/sec-audit_record_types
- msg=audit(1648027154.838:333) - 在audit的括号中记录的是时间戳以及唯一ID标识，同一个事件产生的ID是相同的
- arch=c000003e - 标识系统的CPU架构，这个十六进制表示'x86_64',本机可以使用**ausearch -i --arch c000003e**打印出有这部分内容的audit.log中日志

```
[root@ambari target]# ausearch -i --arch c000003e
type=PROCTITLE msg=audit(2022年03月23日 14:04:27.726:139) : proctitle=/usr/sbin/iptables-restore -w -n
type=SYSCALL msg=audit(2022年03月23日 14:04:27.726:139) : arch=x86_64 syscall=setsockopt success=yes exit=0 a0=0x4 a1=ip a2=IPT_SO_SET_REPLACE a3=0x264db70 items=0 ppid=744 pid=1600 auid=unset uid=root gid=root euid=root suid=root fsuid=root egid=root sgid=root fsgid=root tty=(none) ses=unset comm=iptables-restor exe=/usr/sbin/xtables-multi subj=system_u:system_r:iptables_t:s0 key=(null)
type=NETFILTER_CFG msg=audit(2022年03月23日 14:04:27.726:139) : table=mangle family=ipv4 entries=52
----
......

```

- syscall=159 - 向内核的系统调用的类型，类型值为159，可以通过ausyscall 159查询调用具体的函数名

```shell
[root@ambari target]# ausyscall 159
adjtimex
```

- success=yes - 表示系统调用成功与否
- exit=0 - 系统调用结束时的返回码，可以使用ausearch --interpret --exit 0命令来查看返回值为3的日志解释

```shell
[root@ambari target]#ausearch --interpret --exit 0
type=PROCTITLE msg=audit(2022年03月24日 17:53:32.300:79) : proctitle=/usr/sbin/iptables --wait -I FORWARD -j DOCKER-USER
type=SYSCALL msg=audit(2022年03月24日 17:53:32.300:79) : arch=x86_64 syscall=setsockopt success=yes exit=0 a0=0x4 a1=ip a2=IPT_SO_SET_REPLACE a3=0xb0b4e0 items=0 ppid=1168 pid=1603 auid=unset uid=root gid=root euid=root suid=root fsuid=root egid=root sgid=root fsgid=root tty=(none) ses=unset comm=iptables exe=/usr/sbin/xtables-multi key=(null)
type=NETFILTER_CFG msg=audit(2022年03月24日 17:53:32.300:79) : table=filter family=ipv4 entries=22
----
......
```

- a0=7fffca2da2b0 a1=0 a2=68754 a3=55da09a1b520 - 为系统调用时的前四个arguments，这些arguments依赖于使用的系统调用，可以使用ausearch来查看解释（部分参数可以打印出数值具体的解释）
- items=0 - 表示跟在系统调用后，补充记录的个数，补充记录audit(time_stamp:ID)，ID与本条记录一致
- ppid=2354 - 父进程id
- pid=30729 - 进程id

## linux系统调用表(centos7 x64)

可以通过执行ausyscall --dump命令获取系统调用号以及函数名

```shell
[root@ambari target]# ausyscall --dump
Using x86_64 syscall table:
0       read
1       write
2       open
3       close
4       stat
5       fstat
6       lstat
7       poll
8       lseek
9       mmap
.......
```

也可以指定一个调用号查询函数名

```shell
#audit日志输出信息，其中syscall为159，通过ausyscall可以查询159代表什么函数名
type=SYSCALL msg=audit(1648027154.838:333): arch=c000003e syscall=159 success=yes exit=0 a0=7fffca2da2b0 a1=0 a2=68754 a3=55da09a1                                                                      b520 items=0 ppid=1 pid=729 auid=4294967295 uid=998 gid=996 euid=998 suid=998 fsuid=998 egid=996 sgid=996 fsgid=996 tty=(none) ses                                                                      =4294967295 comm="chronyd" exe="/usr/sbin/chronyd" key="time-change"

#该函数用来对于系统事件偏差进行纠正
[root@ambari target]# ausyscall 159
adjtimex

```

系统调用号	函数名	功能简介
0	read	读文件内容
1	write	向文件中写入内容
2	open	打开指定的文件
3	close	关闭指定的文件
4	stat	获取文件状态信息
5	fstat	获取文件状态信息
6	lstat	获取文件状态信息，对链接文件不解引用
7	poll	监听一组文件描述符上的发生的事件
8	lseek	在文件中定位	 	 
9	mmap	映射虚拟内存页	 	 
10	mprotect	控制虚拟内存权限	 	 
11	munmap	删除虚拟内存映射	 	 
12	brk	调整堆空间范围	 	 
13	sigaction	设置信号的处理函数	 	 
14	sigprocmask	检查并修改阻塞的信号	 	 
15	sigreturn	从信号处理函数中返回并清空栈帧	 	 
16	ioctl	输入输出控制	 	 
17	pread64	对大文件随机读	 	 
18	pwrite64	对大文件随机写	 	 
19	readv	从文件中读取内容并分散到指定的多个缓冲区	 	 
20	writev	从指定的多个缓冲区中获取数据并集中写入到文件	 	 
21	access	检查文件的访问权限	 	 
22	pipe	创建管道	 	 
23	select	多路同步IO轮询	 	 
24	sched_yield	进程主动放弃处理器，并把自己放到调度队列的队尾	 	 
25	mremap	重新映射虚拟内存页	 	 
26	msync	将映射内存中的内容刷新到磁盘	 	 
27	mincore	测试指定的内存页是否在物理内存中	 	 
28	madvise	为内存使用提供建议	 	 
29	shmget	获取共享内存	 	 
30	shmat	连接共享内存	 	 
31	shmctl	共享内存属性控制	 	 
32	dup	复制一个已经打开的文件描述符	 	 
33	dup2	复制一个已经打开的文件描述符	 	 
34	pause	将当前进程挂起，等待信号唤醒	 	 
35	nanosleep	精确的进程睡眠控制	 	 
36	getitimer	获取定时器值	 	 
37	alarm	设置进程的定时提醒	 	 
38	setitimer	设置定时器的值	 	 
39	getpid	获取当前进程的进程ID	 	 
40	sendfile	在文件或端口建传输数据	 	 
41	socket	创建一个套接字	 	 
42	connect	连接远程主机	 	 
43	accept	接受socket上的连接请求	 	 
44	sendto	发送UDP消息	 	 
45	recvfrom	接收UDP消息	 	 
46	sendmsg	发送消息	 	 
47	recvmsg	接收消息	 	 
48	shutdown	关闭Socket上的连接
49	bind	绑定socket	 	 
50	listen	在指定套接字上监听网络事件	 	 
51	getsockname	获取本地套接字的名字	 	 
52	getpeername	获取通信的对端套接字的名字	 	 
53	socketpair	创建一对已连接的无名socket	 	 
54	setsockopt	设置socket的各种属性	 	 
55	getsockopt	读取socket的各种属性	 	 
56	clone	创建线程或进程的底层支持接口	 	 
57	fork	创建子进程	 	 
58	vfork	创建子进程，比fork更加高效，但是有局限	 	 
59	execve	在当前进程中运行指定的程序	 	 
60	exit	退出当前进程	 	 
61	wait4	等待子进程终止，并可获取子进程资源使用数据	 	 
62	kill	给指定的进程发送信号	 	 
63	uname	获取系统名称、版本、主机等信息	 	 
64	semget	获取一组信号量	 	 
65	semop	操作指定的信号量	 	 
66	semctl	信号量属性控制	 	 
67	shmdt	卸载共享内存	 	 
68	msgget	获取消息队列	 	 
69	msgsnd	向消息队列发送消息	 	 
70	msgrcv	从消息队列中读取消息	 	 
71	msgctl	控制消息队列	 	 
72	fcntl	文件描述符属性控制	 	 
73	flock	文件加锁、解锁	 	 
74	fsync	将所有文件内容和文件元数据修改都同步到磁盘	 	 
75	fdatasync	将文件内容和重要的元数据修改同步到磁盘	 	 
76	truncate	截断文件	 	 
77	ftruncate	对文件执行截断	 	 
78	getdents	读取目录项	 	 
79	getcwd	获取当前工作目录	 	 
80	chdir	改变当前工作目录	 	 
81	fchdir	改变当前工作目录	 	 
82	rename	重命名指定的文件	 	 
83	mkdir	创建目录	 	 
84	rmdir	删除目录	 	 
85	creat	创建新文件	 	 
86	link	创建文件链接	 	 
87	unlink	删除文件链接	 	 
88	symlink	创建符号链接	 	 
89	readlink	读取符号链接的内容	 	 
90	chmod	修改文件权限	 	 
91	fchmod	修改文件权限，参数为已经打开的文件描述符	 	 
92	chown	修改文件所有者	 	 
93	fchown	修改文件所有者	 	 
94	lchown	修改链接文件的所有者，不解引用	 	 
95	umask	设置文件权限掩码	 	 
96	gettimeofday	获取当前系统时间	 	 
97	getrlimit	获取当前系统限制	 	 
98	getrusage	获取当前资源使用数据	 	 
99	sysinfo	获取系统信息	 	 
100	times	获取进程运行时间	 	 
101	ptrace	非常强大的进程跟踪系统调用	 	 
102	getuid	获取当前用户标识号	 	 
103	syslog	读取并清空内核消息环形缓存	 	 
104	getgid	获取组标识号	 	 
105	setuid	设置用户标识号	 	 
106	setgid	设置组标识号	 	 
107	geteuid	获取有效用户标识号	 	 
108	getegid	获取有效的组标识号	 	 
109	setpgid	设置指定进程组标识号	 	 
110	getppid	获取父进程的进程ID	 	 
111	getpgrp	获取指定进程组标识号	 	 
112	setsid	设置临时权限用户ID	 	 
113	setreuid	设置真实和有效的用户标识号	 	 
114	setregid	设置真实和有效的组标识号	 	 
115	getgroups	获取当前进程的附属组ID列表	 	 
116	setgroups	设置当前进程的附属组ID列表	 	 
117	setresuid	设置进程的真实用户ID、有效用户ID和特权用户ID	 	 
118	getresuid	获取进程的真实用户ID、有效用户ID和特权用户ID	 	 
119	setresgid	设置进程的真实组ID，有效组ID和特权组ID	 	 
120	getresgid	获取进程的真实组ID，有效组ID和特权组ID	 	 
121	getpgid	获取进程组ID	 	 
122	setfsuid	设置进程组ID	 	 
123	setfsgid	设置文件系统检查时使用的组ID	 	 
124	getsid	 获取特权用户ID	 	 
125	capget	获取进程权限	 	 
126	capset	设置进程权限	 	 
127	sigpending	检查挂起的信号	 	 
128	sigtimedwait	同步地等待排队的信号	 	 
129	sigqueueinfo	 	 	 
130	sigsuspend	挂起进程来等待一个信号	 	 
131	sigaltstack	定义或获取进程的信号栈	 	 
132	utime	修改文件的访问时间或修改时间	 	 
133	mknod	创建文件系统节点	 	 
134	uselib	加载要使用的动态链接库	 	 
135	personality	设置进程的运行域	 	 
136	ustat	获取文件系统信息	 	 
137	statfs	获取文件系统信息	 	 
138	fstatfs	获取文件系统信息	 	 
139	sysfs	获取系统支持的文件系统类型	 	 
140	getpriority	获取进程运行优先级	 	 
141	setpriority	设置进程运行优先级	 	 
142	sched_setparam	设置进程的调度参数	 	 
143	sched_getparam	获取进程的调度参数	 	 
144	sched_setscheduler	设置进程的调度策略和参数	 	 
145	sched_getscheduler	获取进程的调度策略和参数	 	 
146	sched_get_priority_max	获取进程静态优先级上限	 	 
147	sched_get_priority_min	获取进程静态优先级下限	 	 
148	sched_rr_get_interval	取得按RR算法调度的实时进程的时间片长度	 	 
149	mlock	为内存页面加锁	 	 
150	munlock	为内存页面解锁	 	 
151	mlockall	当前进程的所有内存页面加锁	 	 
152	munlockall	当前进程的所有内存页面解锁	 	 
153	vhangup	挂起当前终端	 	 
154	modify_ldt	读写进程的本地描述表	 	 
155	pivot_root	修改当前进程的根文件目录	 	 
156	_sysctl	读/写系统参数	 	 
157	prctl	进程特殊控制	 	 
158	arch_prctl	设置架构相关的线程状态	 	 
159	adjtimex	调整系统时钟	 	 
160	setrlimit	设置系统资源限制	 	 
161	chroot	修改根目录	 	 
162	sync	将内存缓冲区数据写回磁盘	 	 
163	acct	启用或关闭进程记账	 	 
164	settimeofday	设置当前系统时间和时区	 	 
165	mount	挂载文件系统	 	 
166	umount2	卸载文件系统	 	 
167	swapon	开启交换文件和设备	 	 
168	swapoff	关闭交换文件和设备	 	 
169	reboot	重启系统	 	 
170	sethostname	设置主机名称	 	 
171	setdomainname	设置主机域名	 	 
172	iopl	改变进程IO权限级别	 	 
173	ioperm	设置端口IO权限	 	 
174	create_module	创建可装载的模块	 	 
175	init_module	初始化模块	 	 
176	delete_module	删除可装载的模块	 	 
177	get_kernel_syms	获取核心符号（已经被query_module取代）	 	 
178	query_module	查询模块信息	 	 
179	quotactl	控制磁盘配额	 	 
180	nfsservctl	控制NFS守护进程	 	 
181	getpmsg	未实现的系统调用	 	 
182	putpmsg	未实现的系统调用	 	 
183	afs_syscall	未实现的系统调用	 	 
184	tuxcall	未实现的系统调用	 	 
185	security	未实现的系统调用	 	 
186	gettid	获取线程ID	 	 
187	readahead	把文件预读取到页缓存内	 	 
188	setxattr	设置文件或路径的扩展属性	 	 
189	lsetxattr	设置链接文件的扩展属性	 	 
190	fsetxattr	设置文件的扩展属性	 	 
191	getxattr	获取文件或路径的扩展属性	 	 
192	lgetxattr	获取链接文件的扩展属性	 	 
193	fgetxattr	获取文件或路径的扩展属性	 	 
194	listxattr	列出文件或路径的扩展属性	 	 
195	llistxattr	列出链接文件的扩展属性	 	 
196	flistxattr	列出文件或路径的扩展属性	 	 
197	removexattr	移除文件的扩展属性	 	 
198	lremovexattr	移除链接文件的扩展属性	 	 
199	fremovexattr	移除链接文件的扩展属性	 	 
200	tkill	给指定的线程发送信号	 	 
201	time	获取系统时间	 	 
202	futex	快速用户空间锁	 	 
203	sched_setaffinity	设置进程的CPU亲和性掩码	 	 
204	sched_getaffinity	获取进程的CPU亲和性掩码	 	 
205	set_thread_area	设置线程的本地存取区	 	 
206	io_setup	创建异步IO上下文	 	 
207	io_destroy	销毁异步IO上下文	 	 
208	io_getevents	从完成队列中获取异步IO事件	 	 
209	io_submit	提交异步IO块	 	 
210	io_cancel	取消一个未完成的同步IO操作	 	 
211	get_thread_area	获取线程本地存储区	 	 
212	lookup_dcookie	获取一个cookie的完整目录	 	 
213	epoll_create	创建epoll实例	 	 
214	epoll_ctl_old	老的epoll控制接口	 	 
215	epoll_wait_old	老的epoll监控接口	 	 
216	remap_file_pages	创建一个非线性的文件映射	 	 
217	getdents64	获取目录入口	 	 
218	set_tid_address	设置存储线程ID的内存地址
219	restart_syscall	重新启动一个被信号打断的系统调用
220	semtimedop	System V信号操作函数
221	fadvise64	提前声明一个文件的访问模式
222	timer_create	 	 	 
223	timer_settime	 	 	 
224	timer_gettime	 	 	 
225	timer_getoverrun	 	 	 
226	timer_delete	 	 	 
227	clock_settime	 	 	 
228	clock_gettime	 	 	 
229	clock_getres	 	 	 
230	clock_nanosleep	 	 	 
231	exit_group	 	 	 
232	epoll_wait	监听epoll上发生的事件	 	 
233	epoll_ctl	epoll控制接口	 	 
234	tgkill	 	 	 
235	utimes	修改文件的修改或访问时间	 	 
236	vserver	 	 	 
237	mbind	 	 	 
238	set_mempolicy	 	 	 
239	get_mempolicy	 	 	 
240	mq_open	 	 	 
241	mq_unlink	 	 	 
242	mq_timedsend	 	 	 
243	mq_timedreceive	 	 	 
244	mq_notify	 	 	 
245	mq_getsetattr	 	 	 
246	kexec_load	 	 	 
247	waitid	 	 	 
248	add_key	 	 	 
249	request_key	 	 	 
250	keyctl	 	 	 
251	ioprio_set	 	 	 
252	ioprio_get	 	 	 
253	inotify_init	 	 	 
254	inotify_add_watch	 	 	 
255	inotify_rm_watch	 	 	 
256	migrate_pages	 	 	 
257	openat	 	 	 
258	mkdirat	 	 	 
259	mknodat	 	 	 
260	fchownat	 	 	 
261	futimesat	 	 	 
262	newfstatat	 	 	 
263	unlinkat	 	 	 
264	renameat	 	 	 
265	linkat	 	 	 
266	symlinkat	 	 	 
267	readlinkat	 	 	 
268	fchmodat	 	 	 
269	faccessat	 	 	 
270	pselect6	 	 	 
271	ppoll	 	 	 
272	unshare	 	 	 
273	set_robust_list	 	 	 
274	get_robust_list	 	 	 
275	splice	 	 	 
276	tee	 	 	 
277	sync_file_range	 	 	 
278	vmsplice	 	 	 
279	move_pages	 	 	 
280	utimensat	 	 	 
281	epoll_pwait	 	 	 
282	signalfd	 	 	 
283	timerfd_create	 	 	 
284	eventfd	 	 	 
285	fallocate	 	 	 
286	timerfd_settime	 	 	 
287	timerfd_gettime	 	 	 
288	accept4	 	 	 
289	signalfd4	 	 	 
290	eventfd2	 	 	 
291	epoll_create1	 	 	 
292	dup3	 	 	 
293	pipe2	 	 	 
294	inotify_init1	 	 	 
295	preadv	 	 	 
296	pwritev	 	 	 
297	rt_tgsigqueueinfo	 	 	 
298	perf_event_open	 	 	 
299	recvmmsg	 	 	 
300	fanotify_init	 	 	 
301	fanotify_mark	 	 	 
302	prlimit64	 	 	 
303	name_to_handle_at	 	 	 
304	open_by_handle_at	 	 	 
305	clock_adjtime	 	 	 
306	syncfs	更新指定文件描述符的文件系统	 	 
307	sendmmsg	sendmsg的扩展，可在一次系统调用中向socket发送多块数据	 	 
308	setns	设置一个文件描述符的命名空间	 	 
309	getcpu	获取当前线程所在的处理器和节点	 	 
310	process_vm_readv	 	 	 
311	process_vm_writev	 	 	 
312	kcmp	 	 	 
313	finit_module	 	 	 
314	sched_setattr	 	 	 
315	sched_getattr	 	 	 
316	renameat2	 	 	 
317	seccomp	 	 	 
318	getrandom	 	 	 
319	memfd_create	 	 	 
320	kexec_file_load	 	 	 
323	userfaultfd	 	 	 
326	copy_file_range	把文件的一部分内容拷贝到另一个文件
329	pkey_mprotect	 	 	 
330	pkey_alloc	 	 	 
331	pkey_free	 

## 场景

​	 

标签：
#linux #centos #审计 #audit  #安装配置