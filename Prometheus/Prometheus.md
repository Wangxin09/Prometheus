# Prometheus学习手册

## 下载和运行 Prometheus

```shell
#	下载wget
[root@localhost ~]# yum  install -y wget
#	下载安装包
[root@localhost ~]# wget https://github.com/prometheus/prometheus/releases/download/v2.8.1/prometheus-2.8.1.linux-amd64.tar.gz
#	解压
[root@localhost ~]# tar -zxvf prometheus-2.8.1.linux-amd64.tar.gz -C /usr/local/
#	基本操作
[root@localhost ~]# cd /usr/local
#	重命名文件名
[root@localhost local]# mv prometheus-2.8.1.linux-amd64/ prometheus
[root@localhost local]# cd prometheus/
[root@localhost prometheus]#  ./prometheus --version
prometheus, version 2.8.1 (branch: HEAD, revision: 4d60eb36dcbed725fcac5b27018574118f12fffb)
  build user:       root@bfdd6a22a683
  build date:       20190328-18:04:08
  go version:       go1.11.6
 #	下载nano编辑器
 [root@localhost prometheus]# yum install -y nano
 #	修改主机启动IP地址
 [root@localhost prometheus]# nano prometheus.yml 
 # 启动prometheus服务
 [root@localhost prometheus]# ./prometheus 
 #	&连接符，代表后台运行，不占用终端窗口
 [root@localhost prometheus]# /usr/local/Prometheus/prometheus --config.file=/usr/local/Prometheus/prometheus.yml &
 # 查看Prometheus版本信息
 [root@server prometheus]# ./prometheus --version
  prometheus, version 2.8.1 (branch: HEAD, revision: 4d60eb36dcbed725fcac5b27018574118f12fffb)
  build user:       root@bfdd6a22a683
  build date:       20190328-18:04:08
  go version:       go1.11.6
# 使用promtool验证配置文件
[root@server prometheus]# ./promtool check config prometheus.yml 
  Checking prometheus.yml
  SUCCESS: 0 rule files found
 # 添加用户，后期用此账号启动服务
[root@localhost prometheus]# groupadd prometheus
[root@localhost prometheus]# useradd -g prometheus -s /sbin/nologin prometheus
# 赋权和创建prometheus运行数据目录
[root@localhost prometheus]# cd ~
[root@localhost ~]# chown -R prometheus:prometheus /usr/local/prometheus/
[root@localhost ~]# mkdir -p /home/software/prometheus-data
[root@localhost ~]# chown -R prometheus:prometheus /home/software/prometheus-data
#	查看服务端口
[root@server ~]# ss -naltp |grep 9090
#	查看服务状态
[root@localhost ~]# systemctl status prometheus
#	开启prometheus服务
[root@prometheus ~]# systemctl enable prometheus
[root@prometheus ~]# systemctl start prometheus


```

## prometheus开机自动启动设置流程

```sh
#	**设置开机启动**
[root@localhost ~]# touch /usr/lib/systemd/system/prometheus.service

[root@localhost ~]# vim /usr/lib/systemd/system/prometheus.service

# 文件内容
[Unit]
Description=Prometheus
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml

[Install]
WantedBy=multi-user.target


#	设置开机自启
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl enable prometheus
[root@localhost ~]# systemctl start prometheus
```

## 下载和运行 grafana

```sh
#	下载rpm包文件
[root@localhost ~]# wget https://dl.grafana.com/oss/release/grafana-6.1.3-1.x86_64.rpm 
#	下载grafana
[root@localhost ~]# yum -y localinstall grafana-6.1.3-1.x86_64.rpm
# 设置开机启动grafana
[root@localhost ~]# systemctl enable grafana-server
[root@localhost ~]# systemctl start grafana-server
[root@localhost ~]# systemctl status grafana-server


```

## 实验环境的配置

#### 设置系统IP地址

```sh
[root@localhost ~]# systemctl restart network
```

#### 设置Prometheus主机名

```sh
[root@localhost ~]# hostnamectl set-hostname --static server.happy.com
```

#### 设置客户端测试主机名

```sh
[root@localhost ~]# hostnamectl set-hostname --static agent.happy.com
```

#### 设置域名与主机IP

```sh
[root@localhost etc]# nano hosts
#	添加文件
10.130.185.92  server.happy.com
10.130.185.106 agent.happy.com


```

#### 设置系统时间

```sh
#	设置时区
[root@server ~]# timedatectl set-timezone Asia/Shanghai

#	设置NTP时间
[root@server ~]# yum install -y ntpdate
[root@server ~]# ntpdate cn.ntp.org.cn
```

## 远程主机下载安装node_export组件

```sh
 #	下载node_exporter安装包 https://prometheus.io/download/ #	传输node_exporter安装包到Agent主机 wangxin@wangxindeMacBook-Pro-4 ~ % scp node_exporter-1.2.0.linux-amd64.tar root@10.130.185.106:/  # 解压安装node_exporter安装包安装包[root@agent /]# tar -xf node_exporter-1.2.0.linux-amd64.tar -C /usr/local/[root@agent /]# mv /usr/local/node_exporter-1.2.0.linux-amd64/ /usr/local/node_exporter[root@agent /]# ls /usr/local/node_exporter/#	启动node_exporter服务[root@agent ~]# nohup /usr/local/node_exporter/node_exporter &#	检查启动端口[root@agent ~]# ss -naltp|grep 9100 # 配置开机启动文件 [root@agent ~]#  vim /usr/lib/systemd/system/node_exporter.service[Unit]Description=node_exporterDocumentation=https://prometheus.io/After=network.target[Service]Type=simple# User=prometheusExecStart=/usr/local/node_exporter/node_exporterRestart=on-failure[Install]WantedBy=multi-user.target #	设置开机启动[root@node1 ~]# systemctl enable node_exporter[root@node1 ~]# systemctl start node_exporter
```

## 配置Peometheus拉取node远程主机

```sh
#	prometheus配置文件添加监控项[root@server ~]# nano /usr/local/prometheus/prometheus.yml#	默认node-exporter端口为9100  - job_name: 'node_exporter'    static_configs:    - targets:      - 'localhost:9100'#	默认Prometheus端口为9090[root@server ~]# ss -naltp|grep 9090#	重启prometheus[root@server ~]# pkill prometheus[root@server ~]# cd /usr/local/prometheus/./prometheus --config.file=/usr/local/prometheus/prometheus.yml &#	重启prometheus[root@server ~]# systemctl restart prometheus
```

## 下载和运行 Alertmanager

###  一、防火墙的开启、关闭、禁用命令

#### （1）设置开机启用防火墙：systemctl enable firewalld.service

#### （2）设置开机禁用防火墙：systemctl disable firewalld.service

#### （3）启动防火墙：systemctl start firewalld

#### （4）关闭防火墙：systemctl stop firewalld

#### （5）检查防火墙状态：systemctl status firewalld 


###  二，登录UI界面：http://10.130.185.92:9090/graph

### 三，四大组件

#### **Prometheus Server** : 根据配置完成数据采集，数据存储，根据告警规则产生告警并发送给Alertmanager，提供PromQL查询语言的支持。

#### **Push Gateway** : 为应对部分push场景提供的插件，这部分监控数据先推送到 Push Gateway 上，然后再由 Prometheus Server端拉取 。用于存在时间较短，可能在 Prometheus 来拉取之前就消失了的 jobs （若 Prometheus Server 采集间隔期间，Push Gateway 上的数据没有变化， Prometheus Server 将采集到2次相同的数据，仅时间戳不同）

#### **Exporters**:Exporters(探针) 是Prometheus的一类数据采集组件的总称。它负责从目标处搜集数据，并将其转化为Prometheus支持的格式。它并不向中央服务器发送数据，而是等待中央服务器主动前来抓取。

#### **Alertmanager**:　Prometheus server 主要负责根据基于PromQL的告警规则分析数据，如果满足PromQL定义的规则，则会产生一条告警，并发送告警信息到Alertmanager，Alertmanager则是根据配置处理告警信息并发送。常见的接收方式有：电子邮件，webhook,微信 等。Alertmanager三种处理告警信息的方式：分组，抑制，静默。


### 四，收集远程主机信息

通过浏览器访问：http://agentIP:9100/metrics 就可以看到Agent主机的监控信息

### 五，Prometheus的动态发现

1. 基于文件的服务发现
2. 基于DNS的服务发现
3. 基于API的服务发现


```sh
#	查询DNS[root@server ~]# cat /etc/resolv.conf 
```

### 六，Prometheus配置文件

1. scrape_interval 应用程序抓取数据的间隔
2. evaluation_interval 评估规则的频率
    
····1：记录规则

····2：警报规则
   




参考：

https://www.cnblogs.com/fatyao/p/11007357.html

https://blog.csdn.net/qq_31086997/article/details/116904352

https://www.cnblogs.com/xiao987334176/p/11944104.html

https://www.cnblogs.com/yqq-blog/p/13712663.html

https://blog.csdn.net/ywd1992/article/details/85989259

https://www.cnblogs.com/chenqionghe/p/10494868.html







