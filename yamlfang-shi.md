# yaml方式

> 早期的部署方式采用大量的yaml脚本

```bash
kubectl get deployment -n lifesense-qa -o wide|awk '{print $8}'|while read f
do
 image=$(echo $f| awk -F/ '{print $3}')
 name=$(echo $f| awk -F/ '{print $3}'|awk -F- '{print $1}')
 sed -i "s/${name}-services:.*/${image=}/g" ${name}-controller.yaml 
done
```

> 微服务迁移,生成最新的配置



## 负载均衡器

> 公有云的流量入口, 80 443 默认绑定所有节点的ingress端口

![](/assets/1.png)

