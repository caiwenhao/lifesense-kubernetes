# 源码定制

### 1.修改ansible通讯端口

`ansible.cfg`

```
remote_port = 16822
```

> 我们服务器ssh默认端口为16822

### 2.修改主机清单

`inventory/inventory.cfg`

```
master1 ansible_ssh_host=10.9.94.239
master2 ansible_ssh_host=10.9.173.124
master3 ansible_ssh_host=10.9.112.195

[kube-master]
master1
master2
master3

[etcd]
master1
master2
master3

[kube-node]
master1
master2
master3

[k8s-cluster:children]
kube-node
kube-master
```

### 3.优化docker

`inventory\group_vars\all.yml`

```
docker_storage_options: -s overlay2 --storage-opt="overlay2.override_kernel_check=true"
docker_dns_servers_strict: false
```

> 1. docker存储驱动为overlay2 , 并忽略内核版本检查,以便在3.0内核上开启overlay2支持
> 2. 使用默认前3个dns服务器

`inventory\group_vars\k8s-cluster.yml`

```
docker_daemon_graph: "/data/docker_root"
docker_options: "--registry-mirror=http://b377ad59.m.daocloud.io --insecure-registry={{ kube_service_addresses }} --graph={{ docker_daemon_graph }} {{ docker_log_opts }}"
```

> 1. 修改docker存储路径
> 2. 启动镜像国内加速  --registry-mirror=[http://b377ad59.m.daocloud.io](http://b377ad59.m.daocloud.io)

### 4.修改kube全局参数

`inventory\group_vars\k8s-cluster.yml`

```
kube_log_dir: "/data/log/kubernetes"
kube_version: v1.8.4
kube_log_level: 2
kube_network_plugin: flannel
cluster_name: cluster.local
dashboard_enabled: true
helm_enabled: true
```

> 1. kube日志路径
> 2. kube版本
> 3. kube日志级别
> 4. 网络插件 flannel
> 5. 集群域名
> 6. 控制台插件
> 7. helm插件

### 5.DNS服务器

`inventory\group_vars\all.yml`

```
upstream_dns_servers:
  - 10.9.255.1
  - 10.9.255.2
  - 114.114.114.114
```

> 前面两个为ucloud内网dns服务器

### 6.全局参数

`inventory\group_vars\all.yml`

```
bootstrap_os: centos
kubelet_load_modules: true
```

### 7.修改etcd路径

`inventory\group_vars\all.yml`

```
etcd_data_dir: /data/etcd
```

### 8.解决小内存部署问题

```
kubelet_fail_swap_on: false
```

### 9.helm

```
helm_enabled: true
```

### 10.替换仓库源

`roles\download\defaults\main.yml`

```
quay.io reg.lifesense.com
gcr.io reg.lifesense.com
```

```
nginx_image_repo: nginx
nginx_image_repo: reg.lifesense.com/library/nginx
busybox reg.lifesense.com/library/busybox
```

`roles\kubernetes\node\defaults\main.yml`

```
library/nginx reg.lifesense.com/library/nginx
```

`roles\kubernetes-apps\ansible\defaults\main.yml`

```
gcr.io reg.lifesense.com
```

`roles\dnsmasq\templates\dnsmasq-autoscaler.yml.j2`

```
gcr.io reg.lifesense.com
```



```
wget https://kubernetes-helm.storage.googleapis.com/helm-v2.6.2-linux-amd64.tar.gz
mv helm /usr/local/bin/helm
```

```
helm repo add incubator 
https://kubernetes-charts-incubator.storage.googleapis.com/

helm repo add lifesense 
https://caiwenhao.github.io/charts/

helm repo update
```



