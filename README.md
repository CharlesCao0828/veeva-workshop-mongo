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
## Lab3、通过Helm安装类型为Sharded的MongoDB集群
### 步骤一：通过Helm安装类型为Sharded的MongoDB集群。
```
helm install mongo-sharded bitnami/mongodb-sharded --set shards=2,mongos.replicas=1,mongodbRootPassword=Mongo@12345,shardsvr.persistence.size=20
```
### 步骤二：查看Kubernetes资源与MongoDB集群创建状态。
```
## 查看pvc创建
kubectl get pvc
## 查看Mongodb节点
kubectl get pod
## 查看kubernetes service
kubectl get svc
```
### 步骤三：待集群节点全部正常运行后， 创建MongoDB客户端，并登录MongoDB集群。
```
## 创建MongoDB客户端
kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il
## 登录MongoDB集群
mongo admin --host mongo-sharded-mongodb-sharded --authenticationDatabase admin  -u root -p Mongo@12345
```
### 步骤四：查看MongoDB集群shard信息
```
# 在MongoDB shell里执行.
sh.status()
```
### 步骤五：尝试更改MongoDB Shard的Replica数量，提升MongoDB集群的高可用性。
```
helm upgrade mongo-sharded bitnami/mongodb-sharded --set shards=2,shardsvr.dataNode.replicas=2,mongodbRootPassword=Mongo@12345,shardsvr.persistence.size=20 
```
### 步骤六：查看Kubernetes资源，确认MongoDB新的Shard节点与存储卷被创建。
```
kubectl get pvc
kubectl get pod 
```
### 步骤七：查看新创建的MongoDB节点日志。
```
kubectl logs mongo-sharded-mongodb-sharded-shard0-data-1  
# 登录MongoDB集群
mongo admin --host mongo-sharded-mongodb-sharded --authenticationDatabase admin  -u root -p Mongo@12345
# 查看MongoDB集群shard状态
sh.status()
```
## Lab4、利用MongoDB Enterprise Operator部署MongoDB
MongoDB Enterprise Operator简介：MongoDB Enterprise Operator (https://docs.mongodb.com/kubernetes-operator/master/)可用于在Kubernetes平台上管理MongoDB集群的生命周期，包括集群的创建、容量的调整、服务通信、数据备份、快照恢复等任务。MongoDB Enterprise Operator基于标准Kubernetes API开发而成，具有平台无关性，可以安装在各种云厂商以及软件发行商的提供的Kubernetes平台上。其基于标准Kubernetes API开发而成，并提供CRD，开发运维人员仅需要按照MongoDB Enterprise Operator提供的的CRD进行定义并提交任务即可，MongoDB Enterprise Operator会自动地创建Kubernetes Statefulset/PVC/ConfigMap等资源自动地完成MongoDB集群的创建和配置。

### 步骤一：安装MongoDB Enterprise Operator
#### 1.1、克隆MongoDB Enterprise Operator gihub链接。
```
git clone https://github.com/mongodb/mongodb-enterprise-kubernetes.git
```
#### 1.2、 创建Kubernetes Namespace，MongoDB Operator与MongoDB集群都将运行在此命名空间下。
```
kubectl create namespace mongodb
```
#### 1.3 、通过helm工具安装MongoDB Enterprise Operator。
```
cd mongodb-enterprise-kubernetes/
## 通过helm安装MongoDB Enterprise Operator
helm install mongodb helm_chart \
   --values helm_chart/values.yaml
```
#### 1.4 、运行命令查看MongoDB Enterprise Operator安装结果，MongoDB Enterprise Operator会以pod的形式运行在mongodb namespace下。
```
kubectl get pod -n mongodb 
```
### 步骤二、部署OpsManager实例
OpsManager可以为用户提供友好的图形化界面，高效地对多个MongoDB集群进行集中的管理、监控，同时实现一些高级的功能，如查询分析、性能优化以及备份和快照等。
在MongoDB Enterprise Operator的方案中，MongoDB Enterprise Operator首先会创建OpsManager应用实例，待实例创建成功之后，用户可以登录OpsManager进行查看。MongoDB Enterprise Operator为OpsManager自动创建多节点的MongoDB集群（OpsManager Applicaton Database），用于存放OpsManager的相关数据，此外，MongoDB Enterprise Operator支持OpsManager的集群化部署，用户可以根据实际应用场景灵活地设置OpsManager实例的部署数量。
#### 2.1、下载github连接。

#### 2.2、设置“kubectl context”信息，让kubectl默认工作在“mongodb”命名空间下。
```
kubectl config set-context $(kubectl config current-context) --namespace=mongodb
```
#### 2.3、创建Kubernetes Secret，新创建的两个Secret用于存放OpsManager用户名和密码，以及OpsManager Applicaton Database密码。当OpsManager成功创建后，用户可通过OpsManager的用户名和密码登录并OpsManager，可通过OpsManager Application Database的密码对OpsManager的Databse进行维护。
```
# 存放OpsManager用户名密码
kubectl create secret generic om-admin-secret \
--from-literal=Username=admin \
--from-literal=Password=Admin@123 \
--from-literal=FirstName=San \
--from-literal=LastName=Zhang
# 存放OpsManager Applicaton Database
kubectl create secret generic om-db-user-secret \
--from-literal=password=Admin@12345
```
#### 2.4、通过OpsManager yaml配置文件创建OpsManager实例。若需要进一步定制化配置OpsManager，请查看详细配置 (https://docs.mongodb.com/kubernetes-operator/master/reference/k8s-operator-om-specification/)。
```
kubectl apply -f kubernetes/mongodb-ops-manager.yml
```
#### 2.5、查看Kubernetes资源创建信息与OpsManager部署状态。
```
## 查看Kubernetes pvc创建状态（OpsManager Applicaton Database存储卷）
kubectl get pvc
## 查看pod创建状态（OpsManager应用实例）
kubectl get pod 
## 查看OpsManager状态
kubectl get om mongodb-ops-manager -o yaml
## 查看Kubernetes service
kubectl get svc
```
### 步骤三、部署类型为Standalone的MongoDB实例
#### 3.5、创建Kubernetes Secret
```
kubectl -n mongodb \
  create secret generic global-programmatic-key \
  --from-literal="<Public Key>" \
  --from-literal="publicApiKey=<private key>"
```
#### 3.7、利用3.6步获取的Organization ID创建Kubernetes Configmap。
```
kubectl create configmap myconfigmap --from-literal="baseUrl=http://mongodb-ops-manager-svc.mongodb.svc.cluster.local:8080" --from-literal="projectName=mongodb-ops-manager-db" --from-literal="orgId=<org-id>"
```
#### 3.8、利用yaml文件创建MongoDB Standalone实例。
```
kubectl apply -f mongodb-standalone.yml
```
#### 3.9、观察Kubernete资源与MongoDB实例创建状态。
```
## 查看Kubernetes PVC创建
kubectl get pvc |grep mongodb-standalone
## 查看Kubernetes pod创建
kubectl get pod |grep mongodb-standalone
## 查看Kubernetes svc创建
kubectl get svc |grep mongodb-standalone
## 查看MongoDB状态
kubectl get mongodb
```
#### 3.11、删除MongoDB Standalone实例。
```
kubectl delete -f mongodb-standalone.yml
kubectl delete pvc data-mongodb-standalone-0 
```
### 步骤四、部署类型为Replicaset的MongoDB实例
#### 4.1、创建MongoDB Replicaset集群
```
kubectl apply -f mongodb-replicaset.yml 
```
#### 4.2、查看Kubernetes资源创建状态与MongoDB部署状态
```
## 查看PVC部署状态
kubectl get pvc |grep mongo-cluster
## 查看pod部署状态
kubectl get pod |grep mongo-cluster
## 查看Kubernetes service部署状态
kubectl get svc |grep mongo-cluster
## 查看MongoDB集群部署状态
kubectl get mongodb
```
#### 4.4、删除OpsManager集群
```
kubectl delete -f mongodb-replicaset.yml 
kubectl delete pvc data-mongo-cluster-1-0    data-mongo-cluster-1-1   data-mongo-cluster-1-2
```

### 步骤五、部署类型为Sharded的MongoDB实例
#### 5.1、创建MongoDB sharded集群。
```
kubectl apply -f mongodb-sharded.yml 
```


