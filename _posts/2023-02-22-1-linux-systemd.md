---
title: systemctl Linux用于管理系统和管理服务的工具
date: 2023-2-22
category: Linux
tags: systemctl
excerpt: Systemctl是systemd用于管理系统和管理服务的工具。许多现代Linux发行版，如Ubuntu、Debian、Fedora、Linux Mint、OpenSuSE、Redhat都采用systemd作为默认的init系统。
---
Systemctl是systemd用于管理系统和管理服务的工具。许多现代Linux发行版，如Ubuntu、Debian、Fedora、Linux Mint、OpenSuSE、Redhat都采用systemd作为默认的init系统。

使用systemctl，可以启动、停止、重新加载、重启服务、列出服务单元、检查服务状态、启用/禁用服务、管理运行级别和电源管理。在本文中将展示如何在Linux中使用systemctl命令来管理systemd服务。

## 使用systemctl命令 Start/Stop/Restart/Reload 服务

使用systemctl启动服务时，命令格式：`systemctl start [service-name]`。例如，启动firewalld服务：

```shell
systemctl start firewalld
```

与以前老版本的linux中的`service`命令相反，systemctl start命令不输出任何内容。

要停止服务，请使用`systemctl stop [service-name]`。例如，停止firewalld服务：

```shell
systemctl stop firewalld
```



要重新启动服务，请使用`systemctl restart [service-name]`，例如：

```shell
systemctl restart firewalld
```




要重新加载服务的配置（例如ssh）而不重新启动它，请使用`systemctl reload [service-name]`，例如：

```shell
systemctl reload sshd
```





## systemctl检查服务状态

为了查看服务是否正在运行，我们可以使用`systemctl status [service-name]`来查看。

```shell
systemctl status firewalld
```




## 检查服务是否设置为开机启动

要在引导时启用服务，请使用`systemctl enable [service-name]`，例如：

```shell
systemctl enable httpd.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```



同样，disable时取消引导时启用服务：

```shell
systemctl disable httpd.service 
```




可以使用is-enabled选项检查开机是否启动该服务，请运行：

```shell
systemctl is-enabled httpd.service 
```




输出的内容`enabled`表示开机时启动该服务，`disabled`表示开机时不启动该服务。

## systemctl列出单元

要列出所有激活的单元，使用`list-units`选项。

```shell
systemctl list-units
```




要列出所有活动的服务，请运行：

```shell
systemctl list-units -t service 
```




## 使用systemctl重启、关机系统

像`poweroff`、`shutdown`命令一样，systemctl命令可以关闭系统，重启或进入休眠状态。

关机：

```shell
systemctl poweroff 
```

重启：

```shell
systemctl reboot 
```

系统休眠：

```shell
systemctl hibernate
```

## 使用systemclt管理远程系统

通常，上述所有systemctl命令都可以用于通过systemctl命令本身管理远程主机。这将使用ssh与远程主机进行通信。如下所示：

```shell
systemctl status httpd -H root@192.168.0.12
```



`-H`选项，指定远程主机的用户名和密码。

## 管理Targets

Systemd具有Targets的概念，这些Targets的目的与sysVinit系统中的运行级别相似。sysVinit中的运行级别主要是数字（0,1,2,-6）。以下是sysVinit中的运行级别及其对应的systemd中的target：

```text
0   runlevel0.target, poweroff.target
1  runlevel1.target, rescue.target
2,3,4 runlevel2.target, runlevel3.target,runlevel4.target, multi-user.target
5   runlevel5.target, graphical.target
6   runlevel6.target, reboot.target
```

如果想要查看当前的运行级别，可以使用如下命令：

```shell
systemctl get-default 
multi-user.target
```



设置默认的运行级别为graphical，命令如下：

```shell
systemctl set-default graphical.target 
Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/graphical.target.
```


想要列出所有激活的target，可以使用下面命令：

```shell
systemctl list-units -t target
```




## systemd工具的其他命令

### journalctl日志收集

systemd有自己的日志系统，称为journald。它替换了sysVinit中的syslogd。

```shell
journalctl 
```


要查看所有引导消息，请运行命令`journalctl -b`

```shell
journalctl -b
```

以下命令实时跟踪系统日志（类似于tail -f）：

```shell
journalctl -f
```




### 查询系统启动过程的持续时间

```shell
systemd-analyze
Startup finished in 497ms (kernel) + 1.836s (initrd) + 6.567s (userspace) = 8.901s
```


最后显示系统启动时间为8.901秒。

查看服务的启动时间：

```shell
systemd-analyze blame
```



### hostnamectl命令

查看主机名称：

```shell
hostnamectl 
```




## 总结

在本文学习了systemctl命令来管理Linux发行版中的系统服务。