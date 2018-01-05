# 组件日志

## 部署方式一

```
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -t efk -k
```

> 采用kubespary 部署,   缺点是es非持久化,以及kibana访问困难

## helm部署方式

> 需要实现三个组件chart包
>
> * es
> * fluentd
> * kibana





