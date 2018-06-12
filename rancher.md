
# rancher 那些坑
## 将服务器同时设置为分机注意事项

如果您已将主机添加到与Rancher服务器相同的主机上，请注意，您将无法在绑定到端口8080的主机上创建任何容器。 由于Rancher服务器的UI依赖于8080端口，因此将存在端口冲突，并且Rancher将停止工作。 如果您需要为您的容器使用端口8080 ，则可以使用其他端口启动Rancher服务器。如果您已将主机添加到与Rancher服务器相同的主机上，请注意，您将无法在绑定到端口8080的主机上创建任何容器。 由于Rancher服务器的UI依赖于8080端口，因此将存在端口冲突，并且Rancher将停止工作。 如果您需要为您的容器使用端口8080 ，则可以使用其他端口启动Rancher服务器。

## 主机标签问题

通过每台主机，您都可以添加标签来帮助您组织主机。启动牧场主/代理容器时，标签将作为环境变量添加。 UI中的主机标签将是一个键/值对，并且这些键必须是唯一标识符。如果您添加了两个具有不同值的键，我们会将最后输入的值用作键/值对。 

我们开始碰到label在rancher中无法起作用的问题，进入容器后，打个比方为，查询，mypostgres = true
```
use cattle;

select id, quote(`key`),quote(value) from label where type='host' AND state='created' AND value like ' %' OR value like '% ';

select * from label where type='host';
```
经过比较后发现，删除label后状态仍然是created,进入API查看，只显示原始的四个，例如

```
+-----+------+------------+-------+--------------------------------------+-------------+---------+---------------------+---------+-------------+------+------+--------------------------------------+------------------------------------------------+
|   1 | NULL |          5 | label | 801e8199-11de-45b34f3cc8f79 | NULL        | created | 2018-06-12 11:30:31 | NULL    | NULL        | {}   | host | io.rancher.host.agent_image          | rancher/agent:v1.2.10                          |
|   2 | NULL |          5 | label | d6f1e79d-1d18-a1b4-399db4b8a341 | NULL        | created | 2018-06-12 11:30:31 | NULL    | NULL        | {}   | host | io.rancher.host.docker_version       | 18.03                                          |
|   3 | NULL |          5 | label | 2b8f6d60-17b5-e60025ea7d41 | NULL        | created | 2018-06-12 11:30:31 | NULL    | NULL        | {}   | host | io.rancher.host.linux_kernel_version | 3.13                                           |
|   4 | NULL |          5 | label | 4fd42964-d48dcf970740 | NULL        | created | 2018-06-12 11:30:31 | NULL    | NULL        | {}   | host | io.rancher.host.os                   | linux  

``` 

### 碰到死进程， 例如host.activate出现异常怎么办？

```
UPDATE process_instance SET exit_reason='DONE', end_time=NOW()  WHERE exit_reason = 'UNKNOWN_EXCEPTION'
##我们最好先查看再执行

select * from process_instance WHERE exit_reason = 'UNKNOWN_EXCEPTION';
select * from label where type='host';
UPDATE process_instance SET exit_reason='DONE', end_time=NOW()  WHERE exit_reason = 'UNKNOWN_EXCEPTION';
UPDATE label SET state='created', removed=NULL ,remove_time=NULL  WHERE id = '332';

```
以上的思路是如果是因为主机标签引起来的问题，则进入数据库查看和更改标签状态，没问题了再重新激活主机。

### 是否可以删除不需要的记录

例如我们想删除label，有些label确实不存在了，还显示created状态，

