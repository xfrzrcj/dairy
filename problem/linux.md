# **Linux**
## **1、crontab 如何以秒为单位执行任务**

conrtab 命令格式:

`f1 f2 f3 f4 f5 cmd`

* 其中 f1 是表示分钟，f2 表示小时，f3 表示一个月份中的第几日，f4 表示月份，f5 表示一个星期中的第几天。cmd 表示要执行的程序。
* 当 f1 为 * 时表示每分钟都要执行 program，f2 为 * 时表示每小时都要执行程序，以此类推
* 当 f1 为 a-b 时表示从第 a 分钟到第 b 分钟这段时间内要执行，f2 为 a-b 时表示从第 a 到第 b 小时都要执行，其馀类推
* 当 f1 为 \*/n 时表示每 n 分钟个时间间隔执行一次，f2 为 \*/n 表示每 n 小时个时间间隔执行一次，其馀类推
* 当 f1 为 a, b, c,... 时表示第 a, b, c,... 分钟要执行，f2 为 a, b, c,... 时表示第 a, b, c...个小时要执行，其馀类推

注意crontab最小时间单位是分，但有时需要几秒后执行。网上找到有两种办法，其中一种是编写一个脚本，间隔一定秒执行，执行1分钟。然后由 crontab 每分钟调用此脚本。脚本：
```shell
#!/bin/bash  

step=2 #间隔的秒数，不能大于60  

for (( i = 0; i < 60; i=(i+step) )); do  
    $(php '/home/fdipzone/php/crontab/tolog.php')  
    sleep $step  
done  

exit 0
```
参考网址[Linux crontab 实现秒级定时任务](https://www.cnblogs.com/handle/p/9246197.html)


---


## **2、linux下如何利用ps和kill杀死进程**
`ps -ef |grep python |grep -v grep |awk '{print $2}'|xargs kill -9 `
* 注意|grep -v grep可以排除当前查找python的进程
* |awk '{print $2}'为在ps查找的结果中获取第二列,即pid


参考网址[https://www.cnblogs.com/shanheyongmu/p/6001098.html](https://www.cnblogs.com/shanheyongmu/p/6001098.html)
