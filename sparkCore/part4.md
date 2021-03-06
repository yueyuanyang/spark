## Spark算子：RDD行动Action操作(4)–saveAsNewAPIHadoopFile、saveAsNewAPIHadoopDataset

### saveAsNewAPIHadoopFile

def saveAsNewAPIHadoopFile[F <: OutputFormat[K, V]](path: String)(implicit fm: ClassTag[F]): Unit

def saveAsNewAPIHadoopFile(path: String, keyClass: Class[_], valueClass: Class[_], outputFormatClass: Class[_ <: OutputFormat[_, _]], conf: Configuration = self.context.hadoopConfiguration): Unit

saveAsNewAPIHadoopFile用于将RDD数据保存到HDFS上，使用新版本Hadoop API。

用法基本同saveAsHadoopFile。

```
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import SparkContext._
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat
import org.apache.hadoop.io.Text
import org.apache.hadoop.io.IntWritable
 
var rdd1 = sc.makeRDD(Array(("A",2),("A",1),("B",6),("B",3),("B",7)))
rdd1.saveAsNewAPIHadoopFile("/tmp/lxw1234/",classOf[Text],
classOf[IntWritable],classOf[TextOutputFormat[Text,IntWritable]])
```

### saveAsNewAPIHadoopDataset

def saveAsNewAPIHadoopDataset(conf: Configuration): Unit

作用同saveAsHadoopDataset,只不过采用新版本Hadoop API。

以写入HBase为例：

完整的Spark应用程序：

```
HBase建表：
create ‘lxw1234′,{NAME => ‘f1′,VERSIONS => 1},{NAME => ‘f2′,VERSIONS => 1},{NAME => ‘f3′,VERSIONS => 1}
package com.lxw1234.test
 
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import SparkContext._
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.mapreduce.Job
import org.apache.hadoop.hbase.mapreduce.TableOutputFormat
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.client.Result
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.client.Put
 
object Test {
  def main(args : Array[String]) {
   val sparkConf = new SparkConf().setMaster("spark://lxw1234.com:7077").setAppName("lxw1234.com")
   val sc = new SparkContext(sparkConf);
   var rdd1 = sc.makeRDD(Array(("A",2),("B",6),("C",7)))
   
    sc.hadoopConfiguration.set("hbase.zookeeper.quorum ","zkNode1,zkNode2,zkNode3")
    sc.hadoopConfiguration.set("zookeeper.znode.parent","/hbase")
    sc.hadoopConfiguration.set(TableOutputFormat.OUTPUT_TABLE,"lxw1234")
    var job = new Job(sc.hadoopConfiguration)
    job.setOutputKeyClass(classOf[ImmutableBytesWritable])
    job.setOutputValueClass(classOf[Result])
    job.setOutputFormatClass(classOf[TableOutputFormat[ImmutableBytesWritable]])
    
    rdd1.map(
      x => {
        var put = new Put(Bytes.toBytes(x._1))
        put.add(Bytes.toBytes("f1"), Bytes.toBytes("c1"), Bytes.toBytes(x._2))
        (new ImmutableBytesWritable,put)
      }    
    ).saveAsNewAPIHadoopDataset(job.getConfiguration)
    
    sc.stop()   
  }
}
 
```
