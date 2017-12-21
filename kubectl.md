# 常用命令

## 部署kubectl命令

```bash
mkdir kubectl
ssh -i ~/.ssh/id_rsa_sbexx core@master1_ip sudo cat /etc/kubernetes/ssl/admin-node1.pem > /tmp/master/admin-master1.pem
ssh -i ~/.ssh/id_rsa_sbexx core@master1_ip sudo cat /etc/kubernetes/ssl/admin-node1-key.pem > /tmp/master/admin-master1-key.pem
ssh -i ~/.ssh/id_rsa_sbexx core@master1_ip sudo cat /etc/kubernetes/ssl/ca.pem > /tmp/master/ca.pem
```

```bash
kubectl config set-cluster kargo --server=https://10.9.94.239:6443 --certificate-authority=/tmp/master/ca.pem  

kubectl config set-credentials kadmin \
--certificate-authority=/tmp/master/ca.pem \
--client-key=/tmp/master/admin-master1-key.pem \
--client-certificate=/tmp/master/admin-master1.pem

kubectl config set-context kargo --cluster=kargo --user=kadmin
kubectl config use-context kargo

kubectl version
kubectl get node
kubectl get all --all-namespaces
```

```bash
kubectl completion bash >/etc/bash_completion.d/kubectl.sh
source <(kubectl completion bash)
kubectl get nod +[TAB]
```

#### 节点操作

**查看节点**

```
kubectl get node
NAME      STATUS    ROLES         AGE       VERSION
master1   Ready     master,node   48m       v1.8.4+coreos.0
master2   Ready     master,node   49m       v1.8.4+coreos.0
master3   Ready     master,node   48m       v1.8.4+coreos.0
```



