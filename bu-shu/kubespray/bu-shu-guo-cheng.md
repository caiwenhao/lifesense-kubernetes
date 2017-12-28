# 部署过程

> 参考文档 [https://github.com/kubernetes-incubator/kubespray/tree/master/docs](https://github.com/kubernetes-incubator/kubespray/tree/master/docs)
>
> 支持tag单步骤部署

```bash
cd /data/kubespray/
git pull origin lifesense-1.9
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense --list-tags
```

### 镜像下载

> 默认镜像为国外镜像源,为适应生产环境环境我们搭建了一个香港的镜像的仓库`reg.lifesense.com`

```bash
 ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -t facts,docker,download -k
```

> 如果发现镜像缺失,需要通过香港的镜像服务器拉取

```
fatal: [master1]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/bin/docker", "pull", "reg.lifesense.com/library/nginx:1.13"], "delta": "0:00:02.449327", "end": "2017-12-18 20:38:25.971066", "msg": "non-zero return code", "rc": 1, "start": "2017-12-18 20:38:23.521739", "stderr": "Error: image library/nginx:1.13 not found", "stderr_lines": ["Error: image library/nginx:1.13 not found"], "stdout": "Pulling repository reg.lifesense.com/library/nginx", "stdout_lines": ["Pulling repository reg.lifesense.com/library/nginx"]}
```

```bash
#登陆香港镜像仓库服务器
docker pull library/nginx:1.13  
docker tag library/nginx:1.13 reg.lifesense.com/library/nginx:1.13
docker push reg.lifesense.com/library/nginx:1.13

/usr/bin/docker pull quay.io/coreos/hyperkube:v1.8.4_coreos.0
docker tag quay.io/coreos/hyperkube:v1.8.4_coreos.0 reg.lifesense.com/coreos/hyperkube:v1.8.4_coreos.0
docker push reg.lifesense.com/coreos/hyperkube:v1.8.4_coreos.0
```

> 其他惊险的缺失也如此操作

### 集群部署

```
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -k
```

### 部署单个组件

```
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense --list-tags
```

## 添加节点

环境初始化

```
ansible-playbook -i Inventory/k8s Initialize.yml -vs -l 10.9.120.127 -u root
```

集群节点扩容

```
ansible-playbook -i inventory/inventory.cfg  scale.yml -vs -u lifesense -k 
yum  install ceph-common-0.94.5-2.el7.x86_64
```



