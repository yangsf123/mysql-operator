# mysql-operator #

> reference oracle docs : https://blogs.oracle.com/developers/introducing-the-oracle-mysql-operator-for-kubernetes



## 特性 ##

* 集群配置
  * 单主模式 - the group has a single-primary server read-write mode, other members in the group are read-only mode
  * 多主模式

* 集群管理
  * 在kubernetes上使用Innodb和Group Replication创建和扩容MYSQL集群
  * 当集群实例die，MYSQL Operator将会自动重新加入到集群
  * 使用Kubernetes持久卷声明存储到本地磁盘或网络附加存储的数据
  
* 备份和恢复
  * 创建按需备份
  * 创建备份定时任务自动备份数据到 S3
  * 从现有备份还原数据库
  
* 操作
  * 运行在任意云环境Kubernets集群
  * Prometheus预警和监控指标
  * self healing clusters -- 自我治愈
  
  
## 高可用／生产准备MYSQL集群 ##


## Examples ##

  * 创建一个MYSQL集群
    定义一个yaml文件，通过kubectl直接提交到k8s,MYSQL操作员监视MySQLCluster资源，并将启动一个MySQL集群来采取行动
      `
        apiVersion: "mysql.oracle.com/v1"
        kind: MYSQLCluster
        metadata:
          name: mysql-cluster-with-3-replicas
        spec:
          replicas: 3
      `
   * 创建一个随需应变的备份
     创建一个随需应变的数据库备份并且上传到对象存储器
       `
         apiVersion: "mysql.oracle.com/v1"
         kind: MYSQLBackup
         metadata:
          name: mysql-backup
         spec:
          executor:
            provider: mysqldump
            databases:
              - test
          storage:
            provider: s3
            secretRef:
              name: s3-credentials
            config:
              endpoint: x.compat.objectstorage.y.oraclecloud.com
              region: ociregion
              bucket: mybucket
           clusterRef:
            name: mysql-cluster
          `
          
       `kubectl get mysqlbackups`
       `kubectl get mysqlbackup api-production-snapshot-151220170858 -o yaml`

    * 创建一个备份定时任务
        `
          apiVersion: "mysql.oracle.com/v1"
          kind: MySQLBackupSchedule
          metadata:
            name: mysql-backup-schedule
          spec:
            schedule: '30 * * * *'
            backupTemplate:
              executor:
                provider: mysqldump
                databases:
                  - test
                storage:
                  provider: s3  
                  secretRef:
                    name: s3-credentials
                  config:
                    endpoint: x.compat.objectstorage.y.oraclecloud.com
                    region: ociregion
                    bucket: mybucket
                clusterRef:
                  name: mysql-cluster
          `
