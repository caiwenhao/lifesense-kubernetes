# yaml方式

```
kubectl create namespace lifesense-qa2
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: lifesense-qa2  
data:
  DISCONF_ENV: "lifesense-qa2"
  TingYun: "false"
  OPTS: "-Dlog.level=debug"
  lifesense_gray: ""
  lifesense_dubbo_version: ""
```

```
apiVersion: v1
kind: LimitRange
metadata:
  name: lifesenselimits
spec:
  limits:
  - default:
      cpu: 500m
      memory: 640Mi
    defaultRequest:
      cpu: 160m
      memory: 500Mi
    max:
      cpu: "1"
      memory: 1Gi
    min:
      cpu: 100m
      memory: 50Mi
    type: Container
```

> 早期的部署方式采用大量的yaml脚本

```bash
kubectl get deployment -n lifesense-qa -o wide|awk '{print $8}'|while read f
do
 image=$(echo $f| awk -F/ '{print $3}')
 name=$(echo $f| awk -F/ '{print $3}'|awk -F- '{print $1}')
 sed -i "s/${name}-services:.*/${image=}/g" ${name}-controller.yaml 
done
```

## 微服务配置

> 这里以用户服务为例子

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: user-services-rest
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: user-services-rest
    spec:
      containers:
      - name: user-services-rest
        image: 10.10.185.222/rest/user-services:master_9a044c9_152
        env:
        - name: DISCONF_ENV
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: DISCONF_ENV
        - name: TingYun
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: TingYun
        - name: OPTS
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: OPTS
        - name: lifesense_gray
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: lifesense_gray
        - name: lifesense_dubbo_version
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: lifesense_dubbo_version                            
        livenessProbe:
          httpGet:
            path: /echo?requestId=k8s
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 300
          timeoutSeconds: 30
          successThreshold: 1
          failureThreshold: 5
          periodSeconds: 60
        readinessProbe:
          httpGet:
            path: /echo?requestId=k8s
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 30
          successThreshold: 1
          failureThreshold: 5
          periodSeconds: 30
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: user-services-soa
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: user-services-soa
    spec:
      containers:
      - name: user-services-soa
        image: 10.10.185.222/soa/user-services:master_9a044c9_152
        env:
        - name: DISCONF_ENV
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: DISCONF_ENV
        - name: TingYun
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: TingYun
        - name: OPTS
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: OPTS
        - name: lifesense_gray
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: lifesense_gray
        - name: lifesense_dubbo_version
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: lifesense_dubbo_version               
        livenessProbe:
          exec:
            command:
            - /data/apps/livenessProbe
            - check_dubbo
          initialDelaySeconds: 600
          timeoutSeconds: 30
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          tcpSocket:
            port: 20882
          initialDelaySeconds: 30
          periodSeconds: 30
          successThreshold: 1
          failureThreshold: 5      
        ports:
        - containerPort: 20882
          name: dubbo
          protocol: TCP
```

### 创建svc

```
apiVersion: v1
kind: Service
metadata:
  name: user-svc
  labels:
    app: user-svc
spec:
  selector:
    app: user-services-rest
  ports:
    - name: http
      port: 8080
      targetPort: 8080
```

### 创建ingress

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
      - path: /user_service
        backend:
          serviceName: user-svc
          servicePort: 8080
```

### 负载均衡器

> 公有云的流量入口, 80 443 默认绑定所有节点的ingress端口

![](/assets/1.png)测试

```bash
#内网测试
curl --silent -H "Host: sports-qa.lifesense.com" "10.9.94.239/user_service/echo?requestId=88"
#外网测试
curl --silent -H "Host: sports-qa.lifesense.com" "106.75.16.142/user_service/echo?requestId=88"
```

> 以上为用户服务的完整例子, 依此类推添加上其他微服务.

## 特殊协议

### websocket

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: websocket-qa.lifesense.com
  namespace: lifesense-qa  
spec:
  rules:
  - host: websocket-qa.lifesense.com
    http:
      paths: 
      - path: /websocket
        backend:
          serviceName: websocket-svc
          servicePort: 8080
```

### socket

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

## 静态资源

```
apiVersion: v1
kind: Endpoints
metadata:
  name: static-qa
subsets:
  - addresses:
    - ip: 10.9.84.193
    ports:
      - port: 80
```

```
apiVersion: v1
kind: Service
metadata:
  name: static-qa
spec:
  ports:
    - port: 80
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: static-qa.lifesense.com
  namespace: lifesense-qa
  annotations:
    ingress.kubernetes.io/app-root: static-qa.lifesense.com
spec:
  rules:
  - host: static-qa.lifesense.com
    http:
      paths: 
      - path: /
        backend:
          serviceName: static-qa
          servicePort: 80
```

```
curl --silent -H "Host: static-qa.lifesense.com" "10.9.94.239/zepto.min.js"
```



