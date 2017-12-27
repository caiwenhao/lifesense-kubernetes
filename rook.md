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



\*\*支持rbd挂载\*\*

\`\`\`shell

kubectl apply -f deploy/rook-storageclass.yaml

\#测试

kubectl apply -f deploy/test/wordpress.yaml

\`\`\`

\*\*支持cephfs\*\*

\`\`\`shell

\#5.1版本

kubectl exec -ti rook-tools /bin/bash -n rook

rookctl filesystem create --name cephfs

ceph osd pool set cephfs-data size 2

ceph osd pool set cephfs-metadata size 2

\`\`\`

\`\`\`shell

\#应用

yum install jq

kubectl get secret rook-admin -n rook -o json \| jq

'.metadata.namespace = "default"'

\| kubectl apply -f -

export

MONS=

$\(kubectl -n rook get service -l app=rook-ceph-mon -o json

\|

jq ".items\[\].spec.clusterIP"

\|

tr -d "\""

\|

sed -e 's/$/:6790/'

\|

paste -s -d, -\)

sed

"s/INSERT\_MONS\_HERE/$MONS/g"

./test/kube-registry.yaml \| kubectl create -f -

\`\`\`

4.

对象存储

\`\`\`shell

\#未验证

\`\`\`

\*\*\*\*



## 扩容

```bash
xfs_repair /dev/vdb
mount -t xfs /dev/vdb /data
xfs_growfs /data
```



