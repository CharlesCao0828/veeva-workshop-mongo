# veeva-workshop-mongo

## 实验操作

### 步骤一、添加镜像仓库
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
### 步骤二、安装类型为standalone的MongoDB数据库
```
helm install mongo-standalone bitnami/mongodb --set architecture=standalone,auth.rootPassword=Mongo@12345,persistence.size=20Gi
```
