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

```
git clone https://github.com/caiwenhao/charts.git
cd charts/lifesense/
```

### 支持nginx模板

> 通过kubectl cp 命令从容器中拷贝nginx.tmpl

```
kubectl cp kube-system/nginx-ingress-lb-cn5v1:/etc/nginx/nginx.conf /tmp/nginx.conf
kubectl create configmap nginx-template --from-file=./nginx.tmpl -n kube-system
```

```
        volumeMounts:
          - mountPath: /etc/nginx/template
            name: nginx-template-volume
            readOnly: true                
        args:
        - /nginx-ingress-controller
        - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
      volumes:
      - name: nginx-template-volume
        configMap:
          name: nginx-template
          items:
          - key: nginx.tmpl
            path: nginx.tmpl
```

### 修改暴露端口

> 考虑到接入api网关与支持http2的需求

```
        ports:
        - containerPort: 80
          hostPort: 79
        - containerPort: 442
          hostPort: 442
```

### 支持tcp与udp负载均衡

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



