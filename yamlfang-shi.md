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



## 特殊服务处理

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



