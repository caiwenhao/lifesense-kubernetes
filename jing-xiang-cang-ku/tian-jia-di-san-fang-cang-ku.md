# 添加私有仓库

给sa添加镜像认证

```
kubectl create secret docker-registry online-registry --docker-server=10.10.185.222 --docker-username=dockerhub --docker-password='cg$a~<Bzo9*)wYzX' --docker-email=xiaojian.guan@lifesense.com
kubectl patch serviceaccounts default -p '{"imagePullSecrets":[{"name":"online-registry"}]}'
kubectl describe sa default
```



