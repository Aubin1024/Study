# replication原理
 - 主数据库更新数据，将更改写到二进制日志中
 - slave的I/O线程复制Master的二进制日志文件
 - Slave写入到中继日志总
 - Slave的SQL线程将日志文件转为SQL语句并执行，实现复制
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-5-28/90556947.jpg)

# 详细流程
1. Master的I/O线程连接上Master的slave服务线程后，请求从指定的日志文件读取指定的内容。
2. Master收到Slave的请求后，通过自身的I/O线程，根据Slave请求的位置之后的日志信息返还给Slave的I/O线程。返回的信息还包括Master端对应的日志文件名称与日志文件对应的位置方便下次读取。
3. Slave的I/O线程收到后将获取到的日志文件写在Slave端的RelayLog文件最后，并将Master端返回的日志文件名称与日志对应的位置保存至master-info的文件中，以便下一次读取迅速定位。
4. Slave的SQL线程检测到RelaLog文件增加了内容，马上将其转换为SQL语句并执行SQL语句。
5. 至此Master端与Slave端等同于执行了相同的操作，从而实现的SQL复制。
![](http://shuaiguoxia-img.oss-cn-beijing.aliyuncs.com/18-5-28/89840836.jpg)