# helm

## 部署

`inventory\group_vars\k8s-cluster.yml`

```
dashboard_enabled: true
```

```

```

> helm初始化出现错误, 需要多次执行

```
/usr/local/bin/helm init --upgrade --tiller-image=reg.lifesense.com/kubernetes-helm/tiller:v2.7.2 --tiller-namespace=kube-system --service-account=tiller
```



