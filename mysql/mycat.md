## mycat目录详解

1、bin目录

bin目录存放的是mycat各种可执行命令



2、conf目录

conf目录存放的是mycat的各种配置文件，其中比较重要的有以下三个配置文件

* schema.xml：定义逻辑库，表、分片节点等内容
* rule.xml：定义分片规则
* server.xml：定义用户以及系统相关变量，如端口等

