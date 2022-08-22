[TOC]

# K8S
## 好用的客户端 GUI 软件
**lens**
- github：https://github.com/lensapp/lens
- 官网：https://k8slens.dev/
- 只需要导入 kube config 的 yaml 文件就能轻松管理 k8s 集群
- 用 electron 做的，支持 Windows、MacOS、Linux

## 好用的集群管理
**Rancher**
- https://docs.rancher.cn/

## 常用命令
### pod
#### 拷贝文件
- 拷贝本地文件到 pod：
```
kubectl cp {本地文件路径} {pod 名字}:{pod 文件路径}
```
- 从 pod 拷贝文件到本地：
```
kubectl cp {pod 名字}:{pod 文件路径} {本地文件路径}
```

# Docker
- 清理构建缓存：
```
docker builder prune -f
```
