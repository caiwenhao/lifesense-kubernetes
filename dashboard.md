# dashboard

## 部署方式一

> 不建议采用

`inventory\group_vars\k8s-cluster.yml`

```
dashboard_enabled: true
```

```
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -t dashboard -k
```

**配置访问入口**

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: k8s-kibana.lifesense.com
  namespace: kube-system  
spec:
  rules:
  - host: k8s-kibana.lifesense.com
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard
          servicePort: 9090
```

## helm方式部署

```bash
#生成证书
kubectl create secret tls lifesense --key _.lifesense.com.key --cert _.lifesense.com.crt -n kube-system 
```

```
helm install stable/kubernetes-dashboard/ --namespace=kube-system
```

> 访问 https://kubernetes-dashboard.lifesense.com



