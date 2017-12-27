# 访问验证

## basic

```
yum install httpd-tools
htpasswd -c auth lifesense
kubectl create secret generic basic-auth --from-file=auth -n kube-system
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-with-auth
  annotations:
    ingress.kubernetes.io/auth-realm: Authentication Required - lifesense
    ingress.kubernetes.io/auth-secret: basic-auth
    ingress.kubernetes.io/auth-type: basic
```



