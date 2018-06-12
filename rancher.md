

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


