---
title: "Note of Redis(ver.zh-CN)"
date: 2018-09-27T12:00:00+08:00
weight: 20
keywords: ["redis","zh-CN","prometheus"]
description: "Note of Redis(ver.zh-CN)"
tags: ["redis","zh-CN","prometheus"]
categories: ["Ops"]
author: "Fred"
banner: ""
---

*Notice: I will write an article in Chinese if the info I collected is translated well in Chinese.*

*注意事项：如果我收集的资料已经被很好地翻译成了中文，那我会使用中文写这篇文章。*

## 1. 关于Redis的一些笔记

<iframe src='https://www.xmind.net/embed/9248/' width='1000px' height='900px' frameborder='0' scrolling='no' allowfullscreen="true"></iframe>

以上是我收集的关于Redis的一些简要笔记，因为图片过大，所以以这种形式嵌入博客中。以上笔记没有涵盖所有官方的知识点，只作为复习时的索引使用。我会不定期更新笔记并放入博客中。

我推荐使用Redis Cluster作为高可用的分布式集群方案。当然你也可以使用Sentinel，两者都是优秀的集群方案。但是有几点需要注意的是：

1. Redis的数据复制是异步的，也就是说他不是一种强一致的数据库，无论哪种高可用方案都可能在故障转移中丢失极少量数据。
2. Redis不适合放置于虚拟化平台中，性能将有大幅度下降。
3. 不建议把Redis部署在K8S中。容器的特性和AOF的错误恢复无法兼容，而容器的通信方式也会对Redis节点间通信产生影响。如果愿意采坑填坑的话，请记得应用在SLA较低的场景下。
4. 关闭SWAP或禁止Redis使用SWAP。并在配置文件中设置合理的内存上限。同时，选择最符合场景的淘汰算法。
5. 在COW备份时，内存使用将有可能达到两倍。
6. 订阅和发布不支持持久化。
7. 避免将日志放到远程文件系统。
8. 注意运行Redis的用户在系统中最大的文件句柄获得数(fs.file-max)和配置的maxclient是否匹配。
9. 确定ntp同步。

## 2. 部署Redis Cluster

先安装Redis，这里是一个双节点，每节点三个实例的集群。以下脚本结合了一些优化，在CentOS7上可以一键跑完，仅做参考，请根据场景自定义。目前在Ansible Galaxy上没看到合适的role，打算写一个做成双节点和六节点两个模式的role，但一直没时间。另外，如果在K8S部署，推荐使用helm，但是请关注他使用的集群方式以及是否是stateful set。前一段时间看，还是没有适合的有状态集群部署模板。无状态是没问题的。

Redis的配置文件中的每一项都解释得非常清楚。这个赞一下。

1. 在每个节点部署服务

```
# CentOS Linux release 7.3和7.4已通过测试
# 可以自定义以下变量
redis_version=stable
redis_port=(6379 6380 6381)
redis_password=fred
localhost=$(/sbin/ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:")

sudo yum -y install gcc wget tcl ntp
sudo systemctl start ntpd
sudo ntpdate time.nist.gov
wget http://download.redis.io/releases/redis-$redis_version.tar.gz
sudo tar -zxvf redis-$redis_version.tar.gz -C /usr/local/src/
cd /usr/local/src/redis-$redis_version
sudo make
cd /usr/local/src/redis-$redis_version/src
sudo mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server /usr/local/bin
for port in ${redis_port[*]} ;
do
sudo mkdir -p /usr/local/redis-cluster/$port ;
sudo cp /usr/local/src/redis-$redis_version/redis.conf /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s/daemonize no/daemonize yes/g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s/bind 127.0.0.1/bind $localhost/g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s/port 6379/port $port/g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s:dir .:dir /usr/local/redis-cluster/$port:g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's:logfile "":logfile:g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s:logfile:logfile /usr/local/redis-cluster/$port/redis.log:g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's/# cluster-enabled yes/cluster-enabled yes/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s/# cluster-config-file nodes-6379.conf/cluster-config-file nodes-$port.conf/g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's/# cluster-node-timeout 15000/cluster-node-timeout 5000/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's/appendonly no/appendonly yes/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s/# requirepass foobared/requirepass $redis_password/g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i "s/# masterauth <master-password>/masterauth $redis_password/g" /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's/slowlog-log-slower-than 10000/slowlog-log-slower-than 100000/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's/# maxmemory <bytes>/maxmemory 1073741824/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's/# min-slaves-to-write 3/min-slaves-to-write 1/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i 's/# min-slaves-max-lag 10/min-slaves-max-lag 5/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo sed -i '# cluster-migration-barrier 1/cluster-migration-barrier 1/g' /usr/local/redis-cluster/$port/redis.conf ;
sudo bash -c 'cat <<EOF >> /usr/local/redis-cluster/$port/redis.conf
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command KEYS ""
EOF'
sudo bash -c "echo 'vm.overcommit_memory = 1' >> /etc/sysctl.conf"
sudo bash -c "echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled"
sudo bash -c "cat <<EOF > /usr/lib/systemd/system/redis$port.service
[Unit]
Description=Redis $port Service
After=network.target
After=network-online.target
Wants=network-online.target
Before=time-sync.target
Wants=time-sync.target

[Service]
ExecStart=/usr/local/bin/redis-server /usr/local/redis-cluster/$port/redis.conf
Restart=on-failure
RestartSec=5
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF"
sudo systemctl daemon-reload
sudo systemctl restart redis$port.service
sudo systemctl enable redis$port.service
sudo firewall-cmd --add-port=$port/tcp
sudo firewall-cmd --add-port=$(($port+10000))/tcp
sudo firewall-cmd --runtime-to-permanent
done
```

2. 在master节点配置集群

ruby脚本有些小问题，用是可以用，但还是自己写了一下。这个输出比较多，跑的时候可以做成sh文件加参数把输出重定向到dev/null

```
redis_port=(6379 6380 6381)
redis_password=fred
redis_role=master
master_ip=172.16.0.13
slave_ip=172.16.0.14
localhost=$(/sbin/ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:")

if [ $redis_role == master ]
then
for ip in $master_ip $slave_ip;do
for port in ${redis_port[*]} ;do
redis-cli -h $localhost -p ${redis_port[0]} -a $redis_password -c CLUSTER MEET $ip $port;
done
done
for slot in {0..5500};do
redis-cli -h $master_ip -p ${redis_port[0]} -a $redis_password -c CLUSTER ADDSLOTS $slot;
done
for slot in {5501..11000};do
redis-cli -h $master_ip -p ${redis_port[1]} -a $redis_password -c CLUSTER ADDSLOTS $slot;
done
for slot in {11001..16383};do
redis-cli -h $master_ip -p ${redis_port[2]} -a $redis_password -c CLUSTER ADDSLOTS $slot;
done
for port in ${redis_port[*]} ;do
master_id=$(sudo cat /usr/local/redis-cluster/$port/nodes-$port.conf | grep $master_ip:$port | awk '{print $1}');
echo $master_id
done
elif [ $redis_role == slave ]
then
echo "Sorry, slave redis_cli feature will be completed in ansible."
else
echo "redis_role error"
fi
```

请记下出现的三个masterID。

以下是我实践的官方单节点六实例部署方式，请参考。

```
sudo yum -y install ruby rubygems
sudo gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
sudo gem install redis --version 3.0.0
sudo /usr/local/src/redis-$redis_version/src/redis-trib.rb create --replicas 1 172.16.0.13.106:1079 172.16.0.13.106:1080 172.16.0.13.106:1081 172.16.0.13.107:1079 172.16.0.13.107:1080 172.16.0.13.107:1081
```

3. 在slave节点配置集群

这里请按顺序分别把三个masterID替换进去再跑。

```
redis_port=(6379 6380 6381)
redis_password=fred
slave_ip=172.16.0.14

redis-cli -h $slave_ip -p ${redis_port[0]} -a $redis_password -c CLUSTER REPLICATE $master_id1;
redis-cli -h $slave_ip -p ${redis_port[1]} -a $redis_password -c CLUSTER REPLICATE $master_id2;
redis-cli -h $slave_ip -p ${redis_port[2]} -a $redis_password -c CLUSTER REPLICATE $master_id3;
```
