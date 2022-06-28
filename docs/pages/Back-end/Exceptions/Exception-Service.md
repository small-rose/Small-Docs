# 常见异常

## MySQL 相关
----------------

**现象**: MYSQL安装运行正常，计划性断电，服务器关机，重启后MySQL启动失败

**错误信息**: 

状态查询

```text
[root@localhost ~]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: failed (Result: start-limit) since Mon 2022-06-27 10:38:14 CST; 6s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 3606 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=1/FAILURE)
  Process: 3587 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)

Jun 27 10:38:14 localhost systemd[1]: Failed to start MySQL Server.
Jun 27 10:38:14 localhost systemd[1]: Unit mysqld.service entered failed state.
Jun 27 10:38:14 localhost systemd[1]: mysqld.service failed.
Jun 27 10:38:14 localhost systemd[1]: mysqld.service holdoff time over, scheduling restart.
Jun 27 10:38:14 localhost systemd[1]: Stopped MySQL Server.
Jun 27 10:38:14 localhost systemd[1]: start request repeated too quickly for mysqld.service
Jun 27 10:38:14 localhost systemd[1]: Failed to start MySQL Server.
Jun 27 10:38:14 localhost systemd[1]: Unit mysqld.service entered failed state.
Jun 27 10:38:14 localhost systemd[1]: mysqld.service failed.

```

journalctl -xe 日志

```
[root@localhost ~]# journalctl -xe
Jun 27 10:38:14 localhost mysqld[3606]: Initialization of mysqld failed: 0
Jun 27 10:38:14 localhost systemd[1]: mysqld.service: control process exited, code=exited status=1
Jun 27 10:38:14 localhost systemd[1]: Failed to start MySQL Server.
-- Subject: Unit mysqld.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit mysqld.service has failed.
-- 
-- The result is failed.
Jun 27 10:38:14 localhost systemd[1]: Unit mysqld.service entered failed state.
Jun 27 10:38:14 localhost systemd[1]: mysqld.service failed.
Jun 27 10:38:14 localhost systemd[1]: mysqld.service holdoff time over, scheduling restart.
Jun 27 10:38:14 localhost systemd[1]: Stopped MySQL Server.
-- Subject: Unit mysqld.service has finished shutting down
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit mysqld.service has finished shutting down.
Jun 27 10:38:14 localhost systemd[1]: start request repeated too quickly for mysqld.service
Jun 27 10:38:14 localhost systemd[1]: Failed to start MySQL Server.
-- Subject: Unit mysqld.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit mysqld.service has failed.
-- 
-- The result is failed.
Jun 27 10:38:14 localhost systemd[1]: Unit mysqld.service entered failed state.
Jun 27 10:38:14 localhost systemd[1]: mysqld.service failed.
Jun 27 10:40:01 localhost systemd[1]: Started Session 4 of user root.
-- Subject: Unit session-4.scope has finished start-up
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit session-4.scope has finished starting up.
-- 
-- The start-up result is done.
Jun 27 10:40:01 localhost CROND[3634]: (root) CMD (/usr/lib64/sa/sa1 1 1)
```

MYSQL 错误日志:



```bash
[root@localhost ~]# tail -200f /var/log/mysqld.log

2022-06-27T02:38:13.428303Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2022-06-27T02:38:13.428508Z 0 [Warning] 'NO_AUTO_CREATE_USER' sql mode was not set.
2022-06-27T02:38:13.428688Z 0 [Warning] Can't create test file /u01/software/mysql_data/mysql/localhost.lower-test
2022-06-27T02:38:13.428746Z 0 [Note] /usr/sbin/mysqld (mysqld 5.7.33) starting as process 3608 ...
2022-06-27T02:38:13.432149Z 0 [Warning] Can't create test file /u01/software/mysql_data/mysql/localhost.lower-test
2022-06-27T02:38:13.432177Z 0 [Warning] Can't create test file /u01/software/mysql_data/mysql/localhost.lower-test
2022-06-27T02:38:13.434138Z 0 [Note] InnoDB: PUNCH HOLE support available
2022-06-27T02:38:13.434165Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2022-06-27T02:38:13.434169Z 0 [Note] InnoDB: Uses event mutexes
2022-06-27T02:38:13.434175Z 0 [Note] InnoDB: GCC builtin __atomic_thread_fence() is used for memory barrier
2022-06-27T02:38:13.434178Z 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2022-06-27T02:38:13.434183Z 0 [Note] InnoDB: Using Linux native AIO
2022-06-27T02:38:13.434474Z 0 [Note] InnoDB: Number of pools: 1
2022-06-27T02:38:13.434587Z 0 [Note] InnoDB: Using CPU crc32 instructions
2022-06-27T02:38:13.436376Z 0 [Note] InnoDB: Initializing buffer pool, total size = 128M, instances = 1, chunk size = 128M
2022-06-27T02:38:13.445535Z 0 [Note] InnoDB: Completed initialization of buffer pool
2022-06-27T02:38:13.447829Z 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2022-06-27T02:38:13.459984Z 0 [Note] InnoDB: Highest supported file format is Barracuda.
2022-06-27T02:38:13.460197Z 0 [ERROR] InnoDB: Operating system error number 13 in a file operation.
2022-06-27T02:38:13.460225Z 0 [ERROR] InnoDB: The error means mysqld does not have the access rights to the directory.
2022-06-27T02:38:13.460231Z 0 [ERROR] InnoDB: os_file_readdir_next_file() returned -1 in directory ./, crash recovery may have failed for some .ibd files!
2022-06-27T02:38:13.460286Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Generic error
2022-06-27T02:38:14.060962Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2022-06-27T02:38:14.060992Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2022-06-27T02:38:14.060998Z 0 [ERROR] Failed to initialize builtin plugins.
2022-06-27T02:38:14.061002Z 0 [ERROR] Aborting

2022-06-27T02:38:14.061064Z 0 [Note] Binlog end
2022-06-27T02:38:14.061135Z 0 [Note] Shutting down plugin 'CSV'
2022-06-27T02:38:14.061465Z 0 [Note] /usr/sbin/mysqld: Shutdown complete
```


**思路1**：

根据日志错误信`Can't create test file /u01/software/mysql_data/mysql/localhost.lower-test` 可以判断出这个mysql数据文件夹对于mysql用户没有写的权限，所以需要授权。(注意，因为这里是开发机器，权限没有严格控制)
```bash
[root@localhost mysql_data]# ls -lrt
drwxr-x--x. 15 mysql mysql      4096 Jun 27 10:54 mysql
[root@localhost mysql_data]# chmod 777 mysql
[root@localhost mysql_data]# ls -lrt
drwxrwxrwx. 15 mysql mysql      4096 Jun 27 10:54 mysql
```

**思路2**：
如果目录有权限还是无法启动可以尝试关闭SELinux.

```txt
它叫做“安全增强型 Linux（Security-Enhanced Linux）”，简称 SELinux，它是 Linux 的一个安全子系统。

其主要作用就是最大限度地减小系统中服务进程可访问的资源（根据的是最小权限原则）。避免权限过大的角色给系统带来灾难性的结果。
```
可以从配置文件这看到三种值，增强、允许、关闭:
```bash
[root@localhost mysql]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 
```

临时关闭SELinux
```
[root@localhost mysql]# getenforce
Enforcing
[root@localhost mysql]# setenforce 0
[root@localhost mysql]# getenforce
Permissive
```

启动MySQL
```
[root@localhost mysql]# systemctl start mysqld
[root@localhost mysql]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-06-27 10:54:50 CST; 5s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 3756 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 3738 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 3759 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─3759 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Jun 27 10:54:46 localhost systemd[1]: Starting MySQL Server...
Jun 27 10:54:50 localhost systemd[1]: Started MySQL Server.
```

 

