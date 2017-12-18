# 部署过程

> 支持tag单步骤部署

```
cd /data/kubespray/
git pull origin lifesense-1.9
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense --list-tags
```

### 镜像下载

```
 ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -t facts,download -k
```



