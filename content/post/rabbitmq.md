---
title: "Note of RabbitMQ"
date: 2018-10-06T12:00:00+08:00
weight: 20
keywords: ["rabbitmq","prometheus"]
description: "Note of RabbitMQ"
tags: ["rabbitmq","prometheus"]
categories: ["Ops"]
author: "Fred"
banner: ""
---

## 1. Basic process

I made a flowchart of basic process of RabbitMQ as following.

[![rabbitmq01.png](https://i.postimg.cc/CK4NwBXd/rabbitmq01.png)](https://postimg.cc/Y4hg39pw)

Normally, we will use acknowledgements and confirms during the delivery except that lose of messages is accepted in specified scenarios.

Here comes a normal process of clients use.

a. Producer

1. Build connection with server.
2. Open a server channel to process the bulk of AMQP messages.
3. Declare an exchange on the server.
4. Send a Publishing from the client to an exchange on the server with routing key.
5. (RPC only) Choose or declare an exchange and a queue on the server and then starts delivering queued messages.
6. (RPC only) Listen the message and deal with it.

We can also declare a queue to hold messages and directly deliver to consumers without routing key.

b. Consumer

1. Build connection with server.
2. Open a unique, concurrent server channel to process the bulk of AMQP
messages.
3. Declare an exchange on the server.
4. Declare a queue to hold messages.
5. Bind an exchange to a queue.
6. Start delivering queued messages.
7. Listen the message and deal with it.
8. (RPC only) Choose or declares an exchange on the server and then publish it with routing key.

## 2. Some notes

The official document of RabbitMQ is sweet. The note I collected is as below. Not all of the RabbitMQ knowledge is involving in this note. I use it as an index of review. I will update it aperiodically.

<iframe src='https://www.xmind.net/embed/feRy/' width='750px' height='900px' frameborder='0' scrolling='no' allowfullscreen="true"></iframe>

## 3. Deployment

We can deploy RabbitMQ in many ways. Here is a basic yum installation in CentOS 7 I practiced for your reference. You can find detail configuration in the note.

```
#!/bin/bash
wget https://bintray.com/rabbitmq/rpm/download_file?file_path=erlang/21/el/7/x86_64/erlang-21.1-1.el7.centos.x86_64.rpm
mv download* erlang-21.1-1.el7.centos.x86_64.rpm
rpm -Uvh erlang-21.1-1.el7.centos.x86_64.rpm --force
sudo bash -c 'cat <<EOF > /etc/yum.repos.d/rabbitmq.repo
[bintray-rabbitmq-server]
name=bintray-rabbitmq-rpm
baseurl=https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.7.x/el/7/
gpgcheck=0
repo_gpgcheck=0
enabled=1
EOF'
sudo yum -y install rabbitmq-server
for port in 4369 5671 5672 25672 35672-35682 15672 61613 61614 1883 8883 15674 15675;do
  sudo firewall-cmd --add-port=$port/tcp
done
sudo firewall-cmd --runtime-to-permanent
mkdir /etc/systemd/system/rabbitmq-server.service.d
sudo bash -c 'cat <<EOF > /etc/systemd/system/rabbitmq-server.service.d/limits.conf
[Service]
LimitNOFILE=300000
EOF'
sudo systemctl enable rabbitmq-server &&
sudo systemctl restart rabbitmq-server
rabbitmq-plugins enable rabbitmq_management
version=$(sudo rabbitmqctl status | grep 'RabbitMQ' | awk -F '"' '{print $4}')
sudo wget https://raw.githubusercontent.com/rabbitmq/rabbitmq-management/v$version/bin/rabbitmqadmin -P /usr/local/bin
sudo chmod a+x /usr/local/bin/rabbitmqadmin
sudo sh -c '/usr/local/bin/rabbitmqadmin --bash-completion > /etc/bash_completion.d/rabbitmqadmin'
```

## 4. RabbitMQ Exporter

The monitor part has been mentioned in the note. Here is the basic deployment of RabbitMQ exporter for Prometheus.

```
wget https://github.com/kbudde/rabbitmq_exporter/releases/download/v0.29.0/rabbitmq_exporter-0.29.0.linux-amd64.tar.gz
tar -zxvf rabbitmq_exporter-0.29.0.linux-amd64.tar.gz
sudo mv rabbitmq_exporter-0.29.0.linux-amd64/rabbitmq_exporter /usr/local/bin
chown root:root /usr/local/bin/rabbitmq_exporter
sudo firewall-cmd --add-port=9419/tcp
sudo firewall-cmd --runtime-to-permanent
sudo bash -c "cat <<EOF > /usr/lib/systemd/system/rabbitmq-exporter.service
[Unit]
Description=RabbitMQ Exporter Service
After=network.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/bash -c 'PUBLISH_PORT=9419 RABBIT_CAPABILITIES=bert,no_sort /usr/local/bin/rabbitmq_exporter'
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF"
sudo systemctl daemon-reload
sudo systemctl enable rabbitmq-exporter.service &&
sudo systemctl restart rabbitmq-exporter.service
```

You can check the status with this link http://$SERVER_IP:9419/metrics. Please replace the var with your server ip.

## 5. Deployment in K8S

This part will be updated later.

Thank you for your reading.
