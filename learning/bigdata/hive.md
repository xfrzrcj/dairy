# **HIVE**

## 一、hive文件格式

hive文件格式包括以下4种

1、TEXTFILE

2、SEQUENCEFILE

3、RCFILE

4、ORCFILE(0.11以后出现)

5、PARQUET(0.13开始出现)

textfile为默认格式，sequencefile,rcfile,orcfile,无法直接从本地导入数据，需要先导入到textfile的表中，再insert到对应的表内。

+ TEXTFILE：默认格式，数据不做压缩，磁盘开销大，数据解析开销大。可结合Gzip、Bzip2使用(系统自动检查，执行查询时自动解压)，但使用这种方式，hive不会对数据进行切分，从而无法对数据进行并行操作。示例

  ```sql
  create table if not exists textfile_table(
  site string,
  url  string,
  pv   bigint,
  label string)
  row format delimited
  fields terminated by '\t'
  stored as textfile;
  
  插入数据操作：
  set hive.exec.compress.output=true;  
  set mapred.output.compress=true;  
  set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;  
  set io.compression.codecs=org.apache.hadoop.io.compress.GzipCodec;  
  insert overwrite table textfile_table select * from textfile_table;
  ```

  

+ SEQUENCEFILE：SequenceFile是Hadoop API提供的一种二进制文件支持，其具有使用方便、可分割、可压缩的特点。SequenceFile支持三种压缩选择：NONE，RECORD，BLOCK。Record压缩率低，一般建议使用BLOCK压缩。示例

  ```sql
  create table if not exists seqfile_table(
  site string,
  url  string,
  pv   bigint,
  label string)
  row format delimited
  fields terminated by '\t'
  stored as sequencefile;
  
  插入数据操作：
  set hive.exec.compress.output=true;  
  set mapred.output.compress=true;  
  set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;  
  set io.compression.codecs=org.apache.hadoop.io.compress.GzipCodec;  
  SET mapred.output.compression.type=BLOCK;
  insert overwrite table seqfile_table select * from textfile_table;
  ```

+ RCFILE：RCFILE是一种行列存储相结合的存储方式。首先，其将数据按行分块，保证同一个record在一个块上，避免读一个记录需要读取多个block。其次，块数据列式存储，有利于数据压缩和快速的列存取。
  RCFILE文件示例：

  ```sql
  create table if not exists rcfile_table(
  site string,
  url  string,
  pv   bigint,
  label string)
  row format delimited
  fields terminated by '\t'
  stored as rcfile;
  
  插入数据操作：
  set hive.exec.compress.output=true;  
  set mapred.output.compress=true;  
  set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;  
  set io.compression.codecs=org.apache.hadoop.io.compress.GzipCodec;  
  insert overwrite table rcfile_table select * from textfile_table;
  ```

+ ORCFILE：优化的RCfile

  和RCFile格式相比，ORCFile格式有以下优点：
  
  　　(1)、每个task只输出单个文件，这样可以减少NameNode的负载；
  　　(2)、支持各种复杂的数据类型，比如： datetime, decimal, 以及一些复杂类型(struct, list, map, and union)；
  　　(3)、在文件中存储了一些轻量级的索引数据；
  　　(4)、基于数据类型的块模式压缩：a、integer类型的列用行程长度编码(run-length encoding);b、String类型的列用字典编码(dictionary encoding)；
  　　(5)、用多个互相独立的RecordReaders并行读相同的文件；
  　　(6)、无需扫描markers就可以分割文件；
  　　(7)、绑定读写所需要的内存；
  　　(8)、metadata的存储是用 Protocol Buffers的，所以它支持添加和删除一些列。
  
+ PARQUET：和ORCFile类似，不展开。

## 参考：

[Hive学习笔记 --- ORCFile介绍](https://blog.csdn.net/u012965373/article/details/72834392)

[Hive文件格式](https://www.cnblogs.com/Richardzhu/p/3613661.html)

