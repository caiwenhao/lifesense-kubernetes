# ingress

> ingress 是基于nginx的服务入口. 实现了nginx的全功能配置\(支持http2 支持tcp dup转发\).由于是使用kube api 获取容器节点的ip更新nginx upstream. 比nodeport暴露服务的方式减少了一层转发,理论上性能有较大提升

使用案例  
[https://github.com/kubernetes/ingress/tree/master/examples](https://github.com/kubernetes/ingress/tree/master/examples)  
[https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples](https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples)

nginx配置  
[https://github.com/kubernetes/ingress/blob/3a9e3fd7e27e5987d51028781e20db0db3ecc6df/controllers/nginx/configuration.md](https://github.com/kubernetes/ingress/blob/3a9e3fd7e27e5987d51028781e20db0db3ecc6df/controllers/nginx/configuration.md)

## helm方式部署

> [https://github.com/caiwenhao/charts/tree/master/stable/nginx-ingress](https://github.com/caiwenhao/charts/tree/master/stable/nginx-ingress)
>
> 参照官方的chart, 默认不是hostNetwork 所以创建了独立的版本

```bash
git clone https://github.com/caiwenhao/charts.git
cd charts/lifesense/
helm install nginx-ingress --namespace=kube-system
```

## 创建ingress

> ingress是区分namespace的

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

> 微服务实例

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: sports-qa.lifesense.com
  namespace: lifesense-qa
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    ingress.kubernetes.io/force-ssl-redirect: "false"
    ingress.kubernetes.io/ssl-redirect: "false"    
    ingress.kubernetes.io/proxy-body-size: 2m
spec:
  tls:
  - hosts:
    - sports-qa.lifesense.com
    secretName: lifesense
  rules:
  - host: sports-qa.lifesense.com
    http:
      paths: 
      - path: /sms
        backend:
          serviceName: sms-svc
          servicePort: 8080    
```

> 证书支持
>
> 生产环境我们一般会把证书放到负债均衡器. 后端只是使用80

```
  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

## 支持tcp与udp负载均衡

> [https://github.com/kubernetes/ingress/tree/master/examples/tcp/nginx](https://github.com/kubernetes/ingress/tree/master/examples/tcp/nginx)

**增加端口映射,与配置支持**

```
        - containerPort: 65001
          hostPort: 65001
        - containerPort: 65002
          hostPort: 65002
```

```
        args:
        - /nginx-ingress-controller
        - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
        - --tcp-services-configmap=$(POD_NAMESPACE)/nginx-tcp-ingress-configmap
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-tcp-ingress-configmap
  namespace: kube-system
data:
  65002: "lifesense-qa/socket-svc:8080"
  65001: "lifesense-qa2/socket-svc:8080"
```



