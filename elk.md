# elk

`inventory\group_vars\k8s-cluster.yml`

```
efk_enabled: true
```

```
ansible-playbook -i inventory/inventory.cfg cluster.yml -vs -u lifesense -t helm -k
```



