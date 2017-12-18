# 部署过程

> 支持tag单步骤部署

```bash
cd /data/kubespray/
git pull origin lifesense-1.9
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense --list-tags
```

### 镜像下载

> 默认镜像为国外镜像源,为适应生产环境环境我们搭建了一个香港的镜像的仓库`reg.lifesense.com`

```bash
 ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -t facts,download -k
```

> 如果发现镜像缺失,需要通过香港的镜像服务器拉取

```
fatal: [master3]: FAILED! => {"attempts": 4, "changed": false, "cmd": "/usr/bin/docker pull reg.lifesense.com/coreos/hyperkube:v1.8.4_coreos.0", "msg": "[Errno 2] No such file or directory", "rc": 2}
```

```bash
#登陆香港镜像仓库服务器
/usr/bin/docker pull quay.io/coreos/hyperkube:v1.8.4_coreos.0
docker tag quay.io/coreos/hyperkube:v1.8.4_coreos.0 reg.lifesense.com/coreos/hyperkube:v1.8.4_coreos.0
docker push reg.lifesense.com/coreos/hyperkube:v1.8.4_coreos.0
```

> 其他惊险的缺失也如此操作



