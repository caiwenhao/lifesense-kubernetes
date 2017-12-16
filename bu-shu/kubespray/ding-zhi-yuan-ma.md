# 定制源码

**解决什么问题**

> 1. 默认的镜像源是海外服务器,下载慢而且可能是被墙. 
> 2. docker参数的优化
> 3. 以及需要修改环境参数
>
> 官方给出的最佳实践
>
> [https://github.com/caiwenhao/kubespray/blob/master/docs/integration.md](https://github.com/caiwenhao/kubespray/blob/master/docs/integration.md)

## 具体实践

1. github上fork 一份到自己仓库 , 官方 `https://github.com/kubernetes-incubator/kubespray.git`
2. 创建lifesense分支,在分支上做变更
3. 不定期铜鼓不官方源
   ```
   git remote add upstream https://github.com/kubernetes-incubator/kubespray.git
   git checkout master
   git fetch upstream
   git merge upstream/master
   git push origin master
   ```
4. 根据版本需要合并到lifesense分支解决冲突
   ```
   暂时留
   ```



