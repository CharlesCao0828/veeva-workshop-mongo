# veeva-workshop-mongo

## 实验操作
## Lab 1、通过Helm安装类型为Standalone的Mongodb

### 步骤一、添加镜像仓库
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
### 步骤二、安装类型为standalone的MongoDB数据库
```
helm install mongo-standalone bitnami/mongodb --set architecture=standalone,auth.rootPassword=Mongo@12345,persistence.size=20Gi
```
### 步骤三：查看安装状态与结果
```
# 查看pvc创建
kubectl get pvc
# 查看Mongodb节点
kubectl get pod
# 查看kubernetes service
kubectl get svc
```
### 创建MongoDB客户端，并登录MongoDB集群。
```
# 创建MongoDB客户端
kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il
# 登录MongoDB集群
mongo admin --host mongo-standalone-mongodb -u root -p Mongo@12345
# 创建数据库
use online-shop
# 创建集合并插入文档
db.inventory.insertMany([
   { item: "journal", qty: 25, status: "A", size: { h: 14, w: 21, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "notebook", qty: 50, status: "A", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank" ] },
   { item: "paper", qty: 10, status: "D", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank", "plain" ] },
   { item: "planner", qty: 0, status: "D", size: { h: 22.85, w: 30, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "postcard", qty: 45, status: "A", size: { h: 10, w: 15.25, uom: "cm" }, tags: [ "blue" ] }
]);
```
