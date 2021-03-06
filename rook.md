# rook

> [https://rook.io/docs/rook/master/kubernetes.html](https://rook.io/docs/rook/master/kubernetes.html)
>
> go版ceph 原生支持k8s部署, 无缝对接

```
https://rook.io/docs/rook/master/helm-operator.html
```

## 环境准备

节点添加云硬盘并挂载

```
mkfs.xfs /dev/vdc
mkdir /udisk
mount /dev/vdc /udisk
```

```
#加入自启动
vim /etc/fstab
/dev/vdc /udisk ext4 defaults,noatime 0 0
```

```bash
#node节点支持rbd命令
yum  install ceph-common-0.94.5-2.el7.x86_64
```

## 部署

```bash
helm install lifesense/rook/ --name=rook --namespace=rook
#打标签
kubectl label node master1 role=storage-node
kubectl label node master2 role=storage-node
kubectl label node master3 role=storage-node
kubectl get node --show-labels
#初始化集群
kubectl apply -f rook-cluster.yaml
```

```yaml
apiVersion: rook.io/v1alpha1
kind: Cluster
metadata:
  name: rook
  namespace: rook
spec:
  versionTag: v0.5.1
  dataDirHostPath: /udisk/
  monCount: 3
  storage:
    useAllNodes: true
    useAllDevices: false
    storeConfig:
      storeType: bluestore
      databaseSizeMB: 1024
      journalSizeMB: 1024
  placement:
    all:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: role
              operator: In
              values:
              - storage-node
      tolerations:
      - key: storage-node
        operator: Exists
    api:
      nodeAffinity:
      tolerations:
    mgr:
      nodeAffinity:
      tolerations:
    mon:
      nodeAffinity:
      tolerations:
    osd:
      nodeAffinity:
      tolerations:
```

> 不可使用master版本, 由于master变更过于频繁,导致版本不一致,会产生比较多的问题
>
> 当前稳定版本为 v0.5.1

```bash
#部署rook-tools
kubectl apply -f deploy/rook-tools.yaml -n rook
```

### 支持rbd挂载

```
kubectl apply -f deploy/rook-storageclass.yaml -n rook
kubectl get storageclass rook-block 
NAME         PROVISIONER
rook-block   rook.io/block

#测试
kubectl get secret rook-rook-user -o json | jq '.metadata.namespace = "base"' | kubectl apply -f - 
kubectl apply -f deploy/test/wordpress.yaml
```

### 支持cephfs

```
kubectl exec -ti rook-tools /bin/bash -n rook
rookctl filesystem create --name cephfs
ceph osd pool set cephfs-data size 2
ceph osd pool set cephfs-metadata size 2
```

**应用**

```bash
yum install jq
kubectl get secret rook-admin -n rook -o json | jq '.metadata.namespace = "default"' | kubectl apply -f -
export MONS=$(kubectl -n rook get service -l app=rook-ceph-mon -o json|jq ".items[].spec.clusterIP"|tr -d "\""|sed -e 's/$/:6790/'|paste -s -d, -)
sed "s/INSERT_MONS_HERE/$MONS/g" deploy/kube-registry.yaml | kubectl create -f -
```

**支持storageclass**

```
helm install ./cephfs-provisioner/ --namespace=rook
kubectl get storageclass 
NAME         PROVISIONER
cephfs       kubernetes.io/cephfs
rook-block   rook.io/block

#测试
kubectl apply -f deploy/cephfs-pvc.yaml 
kubectl apply -f deploy/cephfs-deploy.yaml
```

> 不同的namespace使用需要配置secret rook-admin

## 扩容

```bash
xfs_repair /dev/vdb
mount -t xfs /dev/vdb /data
xfs_growfs /data
```



