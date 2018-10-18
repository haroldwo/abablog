---
title: "Note of Ceph"
date: 2018-10-16T12:00:00+08:00
weight: 20
keywords: ["ceph"]
description: "Note of Ceph"
tags: ["ceph"]
categories: ["Ops"]
author: "Fred"
banner: ""
---

## 1. Basic Knowledge

The note I collected is as below. Not all of the Ceph knowledge is involving in this note. I use it just for review. I will update it aperiodically.

<iframe src='https://www.xmind.net/embed/TYjA/' width='750px' height='900px' frameborder='0' scrolling='no' allowfullscreen="true"></iframe>

## 2. Deployment

The official documents has involved complete process for deployment. I just write some notes here during my deployment.

1. Updating kernel is necessary for Ceph. For now, kernel version 4.9 and 4.14 is recommmanded according to the official suggestion, or new CRUSH feature cannot be applied. Remember to update all software on the server after updating kernel.
2. Disable selinux.
3. Official repository can be unreachable sometimes. You may try some other mirror such as hk.ceph.com with a high priority.
4. "public network" should be set for Ceph cluster in ceph.conf or the node deployment can be failed.
5. Prometheus exporter can be easily deployed within Ceph-manager module. `ceph mgr module enable prometheus`. The module will accept HTTP requests on port 9283 on all IPv4 and IPv6 addresses on the host.
6. For K8S, you can use Rook or Helm to deploy Ceph. Both of these two ways are under active development now. Please read those documents carefully if you use it in production.

Thank you for your reading.
