# helm指南

## 参考

> [https://github.com/kubernetes/charts](#)
>
> [https://github.com/caiwenhao/zipkin-helm](https://github.com/caiwenhao/zipkin-helm)

## 基于github构建私有helm repo

> 在github上配置GitHub Pages  只允许docs目录
>
> 私有仓库 https://github.com/caiwenhao/charts.git

```bash
#切换到仓库根目录,生成包
helm package -d docs/ lifesense/*

#生成index
helm repo index docs --url https://caiwenhao.github.io/charts/

#提交github
git add * && git commit -m 'rebuild pages' --allow-empty && git push
```

## 项目常用命令

```
#需要进入项目目录
cd disconf
#获取依赖包
helm dependency update
#验证语法
helm lint .
#本地安装
helm install . --namespace=disconf --name=disconf
#本地升级
helm  upgrade  disconf .
#调试不实际运行
helm install --dry-run --debug
#已安装模板
helm get manifest
```

## 使用私有helm repo

```
helm repo add lifesense https://caiwenhao.github.io/charts/
helm repo update
```



