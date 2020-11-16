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
### 步骤四：创建MongoDB客户端，并登录MongoDB集群。
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
# 查询文档
db.inventory.find({})
```
### 步骤五： 删除MongoDB standalone实例。
```
# 查看helm安装版本
helm ls
# 删除helm安装版本
helm uninstall mongo-standalone 
# 查询pod
kubectl get pod 
# 查询pvc
kubectl get pvc
```

## Lab2、通过Helm安装类型为Replicaset的MongoDB
### 步骤一：通过Helm安装MongoDB Replicaset集群。
```
helm install mongo-replicaset bitnami/mongodb --set architecture=replicaset,auth.rootPassword=Mongo@12345,persistence.size=20Gi,replicaCount=2
```
### 步骤二：查看Kubernetes资源与MongoDB集群创建状态。
```
# 查看pvc创建
kubectl get pvc
# 查看Mongodb节点
kubectl get pod
# 查看kubernetes service
kubectl get svc
```
### 步骤三：待集群节点全部正常运行后， 创建MongoDB客户端，并登录MongoDB集群。
```
# 创建MongoDB客户端
kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il
# 登录MongoDB集群
mongo admin --host "mongo-replicaset-mongodb-0.mongo-replicaset-mongodb-headless.default.svc.cluster.local,mongo-replicaset-mongodb-1.mongo-replicaset-mongodb-headless.default.svc.cluster.local," --authenticationDatabase admin -u root -p Mongo@12345
# 查看MongoDB Replicaset 状态
rs.status()
# 查看slave节点数量
rs.printSlaveReplicationInfo()
```
### 步骤四：调整MongoDB集群Replicas数量，对集群进行扩缩容。
```
# 退出mongo shell客户端
exit 
# 获取MONGODB_REPLICA_SET_KEY
kubectl get secret --namespace default mongo-replicaset-mongodb -o jsonpath="{.data.mongodb-replica-set-key}" | base64 --decode
# 更新helm，将MongoDB Replicaset的数量扩展为3个
helm upgrade  mongo-replicaset bitnami/mongodb --set architecture=replicaset,auth.rootPassword=Mongo@12345,persistence.size=20Gi,replicaCount=3,auth.replicaSetKey=<MONGODB_REPLICA_SET_KEY>
# 创建MongoDB客户端
kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il
# 登录MongoDB集群
mongo admin --host "mongo-replicaset-mongodb-0.mongo-replicaset-mongodb-headless.default.svc.cluster.local,mongo-replicaset-mongodb-1.mongo-replicaset-mongodb-headless.default.svc.cluster.local,mongo-replicaset-mongodb-2.mongo-replicaset-mongodb-headless.default.svc.cluster.local," --authenticationDatabase admin -u root -p Mongo@12345
# 查看MongoDB Replicaset 状态
rs.status()
# 查看MongoDB Slave节点数量
rs.printSlaveReplicationInfo()
```

