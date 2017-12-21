## ingress全局配置

### 跳转到后端根目录

> 在某些情况下，后端服务中暴露的URL与Ingress规则中指定的路径不同。 没有重写任何请求将返回404.将注释ingress.kubernetes.io/rewrite-target设置为服务预期的路径。

```
  annotations:
    ingress.kubernetes.io/rewrite-target: /

```

### 支持https与http2

> 如果开启https 则默认开 http2

**去掉强制跳转,因为需要兼容http访问**

```
  annotations:
    ingress.kubernetes.io/force-ssl-redirect: "false"
    ingress.kubernetes.io/ssl-redirect: "false"  

```

**创建证书secret**

> 由于ingress区分namespace,所以每个namespace都需要创建secret

```
kubectl create secret tls lifesense --key _.lifesense.com.key --cert _.lifesense.com.crt -n system-kube

```

```
spec:
  tls:
  - hosts:
    - {{item}}
    secretName: lifesense

```

### 支持tcp转发

> 详见 ingree部署环节



