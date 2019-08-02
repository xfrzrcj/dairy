# **Hadoop**

## 1、hue下 sqoop使用query报错

最近因工作需求，需要用hue编排任务，利用sqoop导入数据。导入脚本语句如下

```cmd
sqoop import --connect jdbc:oracle:thin:@ip:port/db --username user --password pwd --query "select col1,col2 from db.table where \$CONDITIONS" --target-dir /user/kjxydata/src/LT_READER_${date_time} --delete-target-dir -m 1 --null-string '\\N' --null-non-string '\\N' --as-textfile --fields-terminated-by "\t" --hive-drop-import-delims
```

但运行时错误。

在用hue写sqoop导入语句时，有几个坑。

+ 1、在command窗口中不要加 sqoop，直接从import开始。

+ 2、command窗口中使用query是有问题的。对于query后的sql，由于hue调用oozie，oozie在解析命令时会将sql拆解成多个参数，而不是当成一个参数，导致运行时会无法解析命令。

针对第二个问题，目前茶到两种解决方案：

+ 1、直接在hue中利用ssh运行脚本
+ 2、空出command命令框，而在参数框中打入命令

为保持所有sqoop形式命令一致，个人采用第二种方式。具体解决如图：

![图片示例](../pic/1ab5dc57-1361-3727-8950-28e220affd7a.png)

，注意将query语句写在一个arg中。

另外还有一点，注意`select col1,col2 from db.table where \$CONDITIONS`，在sqoop中如果用了query需要加`where $CONDITIONS`，如果是脚本中用记得加`\`，但是在参数窗口中不要加`\`。

详情还可以参考这个[cloudera的提问](https://issues.cloudera.org/browse/HUE-6717)。xml以后的内容就比较明晰了。

