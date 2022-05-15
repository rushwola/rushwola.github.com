hive数据仓库。

# 集群环境

## zk
zk集群：五个节点
m7-qding-bd-242:12181,
m7-qding-bd-245:12181,
BJ-HOST-113:12181
BJ-HOST-138:12181,
m7-qding-bd-244:12181

## hdfs
datanode 16个,每个节点14T。每个节点内存48G,cpu 24核。
双namenode，

##yarn

nodeManage:16
m7-qding-bd-248:8041
m7-qding-bd-247:8041
m7-qding-bd-246:8041
m7-qding-bd-245:8041
m7-qding-bd-244:8041
m7-qding-bd-243:8041
m7-qding-bd-242:8041
BJ-HOST-142:8041
BJ-HOST-141:8041
BJ-HOST-140:8041
BJ-HOST-139:8041
BJ-HOST-138:8041
BJ-HOST-137:8041
BJ-HOST-115:8041
BJ-HOST-114:8041
BJ-HOST-113:8041
每个节点内存48G,24核。

##高可用
双namenode,双resourcesmanage高可用。
