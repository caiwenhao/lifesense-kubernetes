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
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required - lifesense"
```

```
$ curl -v http://10.2.29.4/ -H 'Host: foo.bar.com' -u 'foo:bar'
```



