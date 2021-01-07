# Java 相关目录

> 这是Java相关目录




## 定时任务


- Timer
- Spring Task
- Quartz

## Linux 定时 crontab

```bash
# 检查安装
rpm -qa | grep crontab

# 安装
yum install -y crontabs

# 启动
systemctl start crond

# 查看状态
systemctl status crond

# 停止定时服务
systemctl stop crond

# 开机启动
systemctl enable crond
```


定时任务参数说明：


```bash
cat /etc/crontab
# .---------------- 分钟，取值范围为 0-59
# |  .------------- 小时，取值范围为 0-23
# |  |  .---------- 日，取值范围为 1-31
# |  |  |  .------- 月，取值范围为 1-12
# |  |  |  |  .---- 星期，取值范围为 0-7，0 和 7 都表示星期日
# |  |  |  |  |      .-- 要执行的命令
# |  |  |  |  |      |
  0  19 *  *  *  username bash /home/test.sh
```

查询定时

```bash
crontab -l
```

设置定时

```bash
crontab -e
```
会自动打开一个空文件，可以设置任务。