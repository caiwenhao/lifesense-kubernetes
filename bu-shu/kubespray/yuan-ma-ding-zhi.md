# 源码定制

### 1.修改ansible通讯端口

`ansible.cfg`

```
remote_port = 16822
```

> 我们服务器ssh默认端口为16822

### 替换仓库源

```
quay.io reg.lifesense.com
library/nginx reg.lifesense.com/library/nginx
gcr.io reg.lifesense.com
busybox reg.lifesense.com/library/busybox
nginx_image_repo: nginx
nginx_image_repo: reg.lifesense.com/library/nginx
```

### 修改主机清单

`inventory/inventory.cfg`

```

```

### 优化docker

```
docker_daemon_graph: "/data/docker_root"
docker_options: "--registry-mirror=http://b377ad59.m.daocloud.io --insecure-registry={{ kube_service_addresses }} --graph={{ docker_daemon_graph }} {{ docker_log_opts }}"
```

inventory/group\_vars/k8s-cluster.yml

### 解决小内存部署问题

```
kubelet_fail_swap_on: false
```

### helm

```
helm_version: "v2.6.2"
helm_enabled: true
```

```
wget https://kubernetes-helm.storage.googleapis.com/helm-v2.6.2-linux-amd64.tar.gz
mv helm /usr/local/bin/helm
```



  


kube\_log\_dir: "/data/log/kubernetes"

kube\_network\_plugin: flannel

dashboard\_enabled: false

efk\_enabled: false



\`\`\`

  


4.

inventory/group\_vars/all.yml

  


\`\`\`

bootstrap\_os: centos

etcd\_data\_dir: /data/etcd

kubelet\_load\_modules: true

upstream\_dns\_servers:

- 114.114.114.114

kube\_network\_prefix: 18

docker\_storage\_options: -s overlay2 --storage-opt="overlay2.override\_kernel\_check=true"

docker\_dns\_servers\_strict: false

\`\`\`

  


\#\# 部署

  


1.

ansible.cfg

  


\`\`\`

remote\_port = 16822

\`\`\`

  


  
  


\`\`\`



\`\`\`

  


\#\# app部署

  


charts源

\`\`\`

helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/

helm repo add lifesense https://caiwenhao.github.io/charts/

helm repo update

\`\`\`

  


部署heapster

\`\`\`

helm install stable/heapster --namespace=kube-system -f heapster.yml

\`\`\`

  


  


