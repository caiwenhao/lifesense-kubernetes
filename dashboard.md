# dashboard

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



