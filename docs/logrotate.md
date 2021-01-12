# Linux - logrotate+crontab自动切割日志

[TOC]



https://tool.lu/crontab crontab 自动校验工具

在Linux上，有时候服务日志很多，需要定时清理，那就需要进行日志切割，并定时清理了。网上搜到很好的两篇文章，讲了Linux下面已经自带了超级好用的日记清理软件logrotate，无需再自己造轮子，拿过来一用，果然很棒。

logrotate程序是一个日志文件管理工具。用于分割日志文件，删除旧的日志文件，并创建新的日志文件，起到“转储”作用。可以节省磁盘空间。下面就对logrotate日志轮转操作做一梳理记录：

具体的配置参考了这两篇文章

[Linux 日志切割神器 logrotate 原理介绍和配置详解](https://wsgzao.github.io/post/logrotate/)  这篇文章还有很多的模板介绍

[日志切割方法小结 Logrotate、python、shell脚本实现  - 散尽浮华 - 博客园](https://www.cnblogs.com/kevingrace/p/6307298.html)

## 原理介绍

### logrotate配置

#### Logrotate配置文件

Linux系统默认安装logrotate工具，它默认的配置文件在：

- **/etc/logrotate.conf**   ： 主要的配置文件
- **/etc/logrotate.d/** ： 是一个目录，该目录里的所有文件都会被主动的读入`/etc/logrotate.conf`中执行。

另外，如果 `/etc/logrotate.d/` 里面的文件中没有设定一些细节，则会以`/etc/logrotate.conf`这个文件的设定来作为默认值。
  Logrotate是基于CRON来运行的，其脚本是`/etc/cron.daily/logrotate`，日志轮转是系统自动完成的。
  实际运行时，Logrotate会调用配置文件`/etc/logrotate.conf`。
  可以在/etc/logrotate.d目录里放置自定义好的配置文件，用来覆盖Logrotate的缺省值。

```bash
[root@huanqiu_web1 ~]# cat /etc/cron.daily/logrotate 
#!/bin/sh
 
/usr/sbin/logrotate /etc/logrotate.conf >/dev/null 2>&1
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```
> 如果等不及cron自动执行日志轮转，想手动强制切割日志，需要加-f参数；不过正式执行前最好通过Debug选项来验证一下（-d参数），这对调试也很重要

```bash
# /usr/sbin/logrotate -f /etc/logrotate.d/nginx
# /usr/sbin/logrotate -d -f /etc/logrotate.d/nginx
```
#### logrotate命令格式

```
logrotate [OPTION...] <configfile>
-d, --debug ：debug模式，测试配置文件是否有错误。
-f, --force ：强制转储文件。
-m, --mail=command ：压缩日志后，发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。
```
Eg：根据日志切割设置进行操作，并显示详细信息
```
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -v /etc/logrotate.conf 
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -v /etc/logrotate.d/php
```
Eg：根据日志切割设置进行执行，并显示详细信息,但是不进行具体操作，debug模式
```
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -d /etc/logrotate.conf 
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -d /etc/logrotate.d/nginx
查看各log文件的具体执行情况（有执行记录）
[root@fangfull_web1 ~]# cat /var/lib/logrotate.status
```

#### logrotate参数配置方法

在`/etc/logrotate.d/`目录下新增自己的配置文件，根据自己的配置文件，可以进行不同的配置，多久切割一次，是否压缩等等。

- 执行文件： `/usr/sbin/logrotate`
- 主配置文件: `/etc/logrotate.conf`
- 自定义配置文件: `/etc/logrotate.d/xxx`

> 注意. 修改配置文件后，并不需要重启服务。
>   由于 logrotate 实际上只是一个可执行文件，不是以 daemon 运行。
>   /etc/logrotate.conf - 顶层主配置文件，通过 include 指令，会引入 /etc/logrotate.d 下的配置文件

```
[root@gop-sg-192-168-56-103 wangao]# cat /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
    minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```
`/etc/logrotate.d/` 通常一些第三方软件包，会把自己私有的配置文件，也放到这个目录下。 如 yum，zabbix-agent，syslog，nginx 等。

```
[root@gop-sg-192-168-56-103 logrotate.d]# cat yum
/var/log/yum.log {
    missingok
    notifempty
    size 30k
    yearly
    create 0600 root root
}

```

#### logrotate常见配置参数

> daily ：指定转储周期为每天
weekly ：指定转储周期为每周
monthly ：指定转储周期为每月
rotate count ：指定日志文件删除之前转储的次数，0 指没有备份，5 指保留 5 个备份
tabooext [+] list：让 logrotate 不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和～
missingok：在日志轮循期间，任何错误将被忽略，例如 “文件无法找到” 之类的错误。
size size：当日志文件到达指定的大小时才转储，bytes (缺省) 及 KB (sizek) 或 MB (sizem)
compress： 通过 gzip 压缩转储以后的日志
nocompress： 不压缩
copytruncate：用于还在打开中的日志文件，把当前日志备份并截断
nocopytruncate： 备份日志文件但是不截断
create mode owner group ： 转储文件，使用指定的文件模式创建新的日志文件
nocreate： 不建立新的日志文件
delaycompress： 和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress： 覆盖 delaycompress 选项，转储同时压缩。
errors address ： 专储时的错误信息发送到指定的 Email 地址
ifempty ：即使是空文件也转储，这个是 logrotate 的缺省选项。
notifempty ：如果是空文件的话，不转储
mail address ： 把转储的日志文件发送到指定的 E-mail 地址
nomail ： 转储时不发送日志文件
olddir directory：储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir： 转储后的日志文件和当前日志文件放在同一个目录下
prerotate/endscript： 在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
更多信息请参考 man logrotate 帮助文档

### crontab定时任务配置

#### crontab设置定时任务

- 添加定时任务`crontab -e `

```bash
[root@rocketmq-test-2 logrotate.d]# crontab -l

//添加以下内容
59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/0x4a4f4d482d1c3686cb9140718bba52b7768a0d45_besu.log >/dev/null 2>&1
```

- 查看定时任务`crontab -l`

```bash
[root@rocketmq-test-2 logrotate.d]# crontab -l
*/30 * * * * /opt/.CloudOptimus/upgrade


59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/0x4a4f4d482d1c3686cb9140718bba52b7768a0d45_besu.log >/dev/null 2>&1
59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/0x6a96ee2d082b36241b8a6240408c780e6c8f5789_besu.log >/dev/null 2>&1
59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/0xa3981bc97550088e44d72ffca2c7ba1a90dcd5cd_besu.log >/dev/null 2>&1
59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/0xa57143801563d9e03625fc190ebe47031ad824c9_besu.log >/dev/null 2>&1
```

这样就OK啦，之后就能查看，是不是生效了。

> 如果等不及cron自动执行日志轮转，想手动强制切割日志，需要加-f参数；不过正式执行前最好通过Debug选项来验证一下（-d参数），这对调试也很重要

```bash
# /usr/sbin/logrotate -f /etc/logrotate.d/nginx

# /usr/sbin/logrotate -d -f /etc/logrotate.d/nginx
```

####  crontab 参数说明

>     具体格式如下：
>       Minute Hour Day Month Dayofweek   command
>       分钟     小时   天     月       天每星期       命令
>      每个字段代表的含义如下：
>      Minute             每个小时的第几分钟执行该任务
>      Hour               每天的第几个小时执行该任务
>      Day                 每月的第几天执行该任务
>      Month             每年的第几个月执行该任务
>      DayOfWeek     每周的第几天执行该任务
>      Command       指定要执行的程序
>      在这些字段里，除了“Command”是每次都必须指定的字段以外，其它字段皆为可选
>
>     字段，可视需要决定。对于不指定的字段，要用“*”来填补其位置。
>     举例如下：
>     5       *       *           *     *     ls             指定每小时的第5分钟执行一次ls命令
>     30     5       *           *     *     ls             指定每天的 5:30 执行ls命令
>     30     7       8         *     *     ls             指定每月8号的7：30分执行ls命令
>     30     5       8         6     *     ls             指定每年的6月8日5：30执行ls命令
>     30     6       *           *     0     ls             指定每星期日的6:30执行ls命令[注：0表示星期天，1表示星期1，
>
>     以此类推，也可以用英文来表示，sun表示星期天，mon表示星期一等。]
>
>     30     3     10,20     *     *     ls     每月10号及20号的3：30执行ls命令[注：“，”用来连接多个不连续的时段]
>
>     25     8-11 *           *     *     ls       每天8-11点的第25分钟执行ls命令[注：“-”用来连接连续的时段]
>
>     */15   *       *           *     *     ls         每15分钟执行一次ls命令 [即每个小时的第0 15 30 45 60分钟执行ls命令 ]
>
>      30   6     */10         *     *     ls       每个月中，每隔10天6:30执行一次ls命令[即每月的1、11、21、31日是的6：30执行一次ls 命令。 ]
>
>      50   7       *             *     *     root     run-parts     /etc/cron.daily   每天7：50以root 身份执行/etc/cron.daily目录中的所有可执行文件[ 注：run-parts参数表示，执行后面目录中的所有可执行文件。 

## 配置实例

### 1. 添加配置文件

直接选了一个模板，在`/etc/logrotate.d/`增加了几个配置文件，对应相应的需要切割的日志

这里面参数的定义是：

```bash
[root@G6-bs02 logrotate.d]# cat nstc_nohup.out
/data/nstc/nohup.out { #需要切割的日志路径
rotate 30   #一次将存储 30 个归档日志。对于第六个归档，时间最久的归档将被删除。
dateext  #让旧日志文件以创建日期命名。
daily #日志文件将按日轮循,其它可用值为 monthly，weekly 或者 yearly。
copytruncate  #用于还在打开中的日志文件，把当前日志备份并截断
compress #在轮循任务完成后，已轮循的归档将使用 gzip 进行压缩。
notifempty #如果日志文件为空，轮循不会进行。
missingok #在日志轮循期间，任何错误将被忽略，例如 “文件无法找到” 之类的错误。
}
```

达到的效果就是，**每天凌晨进行日志切割，并且前一天的日志会被压缩成.gz文件，并且30天之前的会删除。**

### 2. 添加定时任务

添加定时任务

```bash
crontab -e
```

添加以下内容：

```
59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/nstc_nohup.out >/dev/null 2>&1
```

这个命令也就是说，每天23:59分，执行这条命令

查看定时任务是否成功

```bash
[root@rocketmq-test-2 logrotate.d]# crontab -l
*/30 * * * * /opt/.CloudOptimus/upgrade

59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/0x4a4f4d482d1c3686cb9140718bba52b7768a0d45_besu.log >/dev/null 2>&1
```

这样就OK啦，之后就能查看，是不是生效了。

如果等不及cron自动执行日志轮转，想手动强制切割日志，需要加-f参数；不过正式执行前最好通过Debug选项来验证一下（-d参数），这对调试也很重要

```
# /usr/sbin/logrotate -f /etc/logrotate.d/nginx

# /usr/sbin/logrotate -d -f /etc/logrotate.d/nginx
```

## 其他相关说明

1. 查看crontab执行日志：

```
 tail -f /var/log/cron
```

2. `/dev/null 2>&1` 是什么意思？

先将标准输出重定向到 `/dev/null`，然后将标准错误重定向到标准输出，由于标准输出已经重定向到了 `/dev/null`，因此标准错误也会重定向到 `/dev/null`

将这条语句拆解开来看：

> \>是重定向符
>
> /dev/null 是一个黑洞，任何发送给它的数据都会被丢弃
>
> 2 是标准错误的文件描述符
>
> \> 同上
>
> & symbol for file descriptor
>
> 1 标准输出描述符

若要将正确和错误日志都输出到load.log

```bash
*/1 * * * * /root/XXXX.sh > /tmp/load.log 2>&1 &
```



