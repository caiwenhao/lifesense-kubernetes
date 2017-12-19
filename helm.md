# helm

## 部署

`inventory\group_vars\k8s-cluster.yml`

```
dashboard_enabled: true
```

```
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -t helm -k
```

> helm初始化出现错误, 需要多次执行

```
/usr/local/bin/helm init --upgrade --tiller-image=reg.lifesense.com/kubernetes-helm/tiller:v2.7.2 --tiller-namespace=kube-system --service-account=tiller
```

### 解决容器化部署无法读取本地的问题

> 1.8.4版本之前helm是容器化部署, 目前最新版本支持host部署,已经解决了这个问题, 故可忽略

```
helm version
Client: &version.Version{SemVer:"v2.7.2", GitCommit:"8478fb4fc723885b155c924d1c8c410b7a9444e6", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.7.2", GitCommit:"8478fb4fc723885b155c924d1c8c410b7a9444e6", GitTreeState:"clean"}
```

```
wget https://kubernetes-helm.storage.googleapis.com/helm-v2.7.2-linux-amd64.tar.gz
mv helm /usr/local/bin/helm
```

### 添加仓库

```
helm repo list
NAME    URL                                             
stable  https://kubernetes-charts.storage.googleapis.com
local   http://127.0.0.1:8879/charts
```

```
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo add lifesense https://caiwenhao.github.io/chart
helm repo update
```



