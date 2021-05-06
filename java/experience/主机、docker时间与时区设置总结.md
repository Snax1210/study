[TOC]: # "主机、Docker与时区设置总结"


最近在使用Docker容器时，部署java程序发现时间输出不对，在修改问题时总结如下。

　　#date　　　　　　　　　　　　　　　　　　　　　　#查看主机时间  
　　#timedatectl 　　　　　　　　　　　　　　　　　　 　#查看主机时区  
　　#tzselect 　　　　　　　　　　　　　　　　　　　　　　#选择时区，5 选择亚洲 > 9 选择中国时区 -> 1选择北京时间 -> 1 选择Yes

修改主机时区

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime             #上海时间

rm /etc/localtime  
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime          #上海时间

更新主机时间

date                                      #查看当前系统时间  
yum install -y ntpdate                    #安装ntpdate程序  
ntpdate cn.pool.ntp.org                   #更新系统时间  
date                                      #再次查看当前系统时间

网络时间服务器


ntp1.aliyun.com  
ntp2.aliyun.com  
ntp3.aliyun.com  
ntp4.aliyun.com  
ntp5.aliyun.com  
ntp6.aliyun.com  
ntp7.aliyun.com  
0.cn.pool.ntp.org  
1.cn.pool.ntp.org  
2.cn.pool.ntp.org  
3.cn.pool.ntp.org


同步BIOS时钟，强制把系统时间写入CMOS

clock --show                   　　　　　　#查看硬件时间  
clock -w                       　　　　　　#强制把系统时间写入CMOS  
clock --show                   　　　　　　#查看硬件时间  
reboot                         　　　　　　#重起机器

设置系统自动同步时间

vi /etc/crontab                            #设置定时任务

00 0 1 \* \* ntpdate -s cn.pool.ntp.org      --每月一号同步  
\* \*/1 \* \* \* ntpdate -s cn.pool.ntp.org     --每一个小时同步

Docker时间和宿主同步方法

　　1.在run容器时添加参数挂载宿主时间配置：　　-v /etc/localtime:/etc/localtime

　　2.复制宿主localtime时间配置覆盖：　　docker cp /etc/localtime container\_id:/etc/localtime

　　3.在启动jar包添加时区参数：　　-Duser.timezone=GMT+08

Docker容器设置时区

　　#docker exec -it container\_id /bin/bash 　　　　　　　　　　 #进入容器命令

　　#修改时区，设置为上海时区  
　　ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  
　　或者  
　　cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

Docker设置build参数

　　ENV TZ=Asia/Shanghai  
　　RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

构建dockerfile镜像.

