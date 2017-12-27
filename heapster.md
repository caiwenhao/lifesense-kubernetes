# heapster

> 集群资源数据监控

## 部署

```
helm install stable/heapster/ --namespace=kube-system
```

### 配置sink数据持久化

```
待更新
```

## 应用

查看node节点资源

```
kubectl top node
NAME      CPU(cores)   CPU%      MEMORY(bytes)   MEMORY%   
master2   5412m        34%       48130Mi         75%       
master3   4040m        25%       48336Mi         75%       
master1   6801m        43%       50973Mi         79%       
```

查看pod资源

```
kubectl top pod -n kube-system 
NAME                                                         CPU(cores)   MEMORY(bytes)   
kube-controller-manager-master2                              49m          111Mi           
kubedns-autoscaler-c4769c65b-hs7dh                           0m           8Mi             
kube-scheduler-master2                                       13m          45Mi            
kube-proxy-master1                                           25m          53Mi            
kube-scheduler-master1                                       32m          42Mi           
```



