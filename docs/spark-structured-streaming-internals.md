== Internals of Streaming Queries

NOTE: The page is to keep notes about how to guide readers through the codebase and may disappear if merged with the other pages or become an intro page.

. <<DataStreamReader, DataStreamReader and Streaming Data Source>>
. <<data-source-resolution, Data Source Resolution, Streaming Dataset and Logical Query Plan>>
. <<dataset, Dataset API -- High-Level DSL to Build Logical Query Plan>>
. <<DataStreamWriter, DataStreamWriter and Streaming Data Sink>>
. <<StreamingQuery, StreamingQuery>>
. <<StreamingQueryManager, StreamingQueryManager>>

=== [[DataStreamReader]] DataStreamReader and Streaming Data Source

It all starts with `SparkSession.readStream` method which lets you define a <<spark-sql-streaming-Source.adoc#, streaming source>> in a *stream processing pipeline* (aka _streaming processing graph_ or _dataflow graph_).

[source, scala]
----
import org.apache.spark.sql.SparkSession
assert(spark.isInstanceOf[SparkSession])

val reader = spark.readStream

import org.apache.spark.sql.streaming.DataStreamReader
assert(reader.isInstanceOf[DataStreamReader])
----

`SparkSession.readStream` method creates a <<spark-sql-streaming-DataStreamReader.adoc#, DataStreamReader>>.

The fluent API of `DataStreamReader` allows you to describe the input data source (e.g. <<spark-sql-streaming-DataStreamReader.adoc#format, DataStreamReader.format>> and <<spark-sql-streaming-DataStreamReader.adoc#options, DataStreamReader.options>>) using method chaining (with the goal of making the readability of the source code close to that of ordinary written prose, essentially creating a domain-specific language within the interface. See https://en.wikipedia.org/wiki/Fluent_interface[Fluent interface] article in Wikipedia).

[source, scala]
----
reader
  .format("csv")
  .option("delimiter", "|")
----

There are a couple of built-in data source formats. Their names are the names of the corresponding `DataStreamReader` methods and so act like shortcuts of `DataStreamReader.format` (where you have to specify the format by name), i.e. <<spark-sql-streaming-DataStreamReader.adoc#csv, csv>>, <<spark-sql-streaming-DataStreamReader.adoc#json, json>>, <<spark-sql-streaming-DataStreamReader.adoc#orc, orc>>, <<spark-sql-streaming-DataStreamReader.adoc#parquet, parquet>> and <<spark-sql-streaming-DataStreamReader.adoc#text, text>>, followed by <<spark-sql-streaming-DataStreamReader.adoc#load, DataStreamReader.load>>.

You may also want to use <<spark-sql-streaming-DataStreamReader.adoc#schema, DataStreamReader.schema>> method to specify the schema of the streaming data source.

[source, scala]
----
reader.schema("a INT, b STRING")
----

In the end, you use <<spark-sql-streaming-DataStreamReader.adoc#load, DataStreamReader.load>> method that simply creates a streaming Dataset (the good ol' Dataset that you may have already used in Spark SQL).

[source, scala]
----
val input = reader
  .format("csv")
  .option("delimiter", "\t")
  .schema("word STRING, num INT")
  .load("data/streaming")

import org.apache.spark.sql.DataFrame
assert(input.isInstanceOf[DataFrame])
----

The Dataset has the `isStreaming` property enabled that is basically the only way you could distinguish streaming Datasets from regular, batch Datasets.

[source, scala]
----
assert(input.isStreaming)
----

In other words, Spark Structured Streaming is designed to extend the features of Spark SQL and let your structured queries be streaming queries.

=== [[data-source-resolution]] Data Source Resolution, Streaming Dataset and Logical Query Plan

Being curious about the internals of streaming Datasets is where you start...seeing numbers not humans (sorry, couldn't resist drawing the comparison between https://en.wikipedia.org/wiki/The_Matrix[Matrix the movie] and the internals of Spark Structured Streaming).

Whenever you create a Dataset (be it batch in Spark SQL or streaming in Spark Structured Streaming) is when you create a *logical query plan* using the *high-level Dataset DSL*.

A logical query plan is made up of logical operators.

Spark Structured Streaming gives you two logical operators to represent streaming sources, i.e. <<spark-sql-streaming-StreamingRelationV2.adoc#, StreamingRelationV2>> and <<spark-sql-streaming-StreamingRelation.adoc#, StreamingRelation>>.

When <<spark-sql-streaming-DataStreamReader.adoc#load, DataStreamReader.load>> method is executed, `load` first looks up the requested data source (that you specified using <<spark-sql-streaming-DataStreamReader.adoc#format, DataStreamReader.format>>) and creates an instance of it (_instantiation_). That'd be *data source resolution* step (that I described in...FIXME).

`DataStreamReader.load` is where you can find the intersection of the former <<spark-sql-streaming-micro-batch-stream-processing.adoc#, Micro-Batch Stream Processing>> V1 API with the new <<spark-sql-streaming-continuous-stream-processing.adoc#, Continuous Stream Processing>> V2 API.

For <<spark-sql-streaming-MicroBatchReadSupport.adoc#, MicroBatchReadSupport>> or <<spark-sql-streaming-ContinuousReadSupport.adoc#, ContinuousReadSupport>> data sources, `DataStreamReader.load` creates a logical query plan with a <<spark-sql-streaming-StreamingRelationV2.adoc#, StreamingRelationV2>> leaf logical operator. That is the new *V2 code path*.

.StreamingRelationV2 Logical Operator for Data Source V2
[source, scala]
----
// rate data source is V2
val rates = spark.readStream.format("rate").load
val plan = rates.queryExecution.logical
scala> println(plan.numberedTreeString)
00 StreamingRelationV2 org.apache.spark.sql.execution.streaming.sources.RateStreamProvider@2ed03b1a, rate, [timestamp#12, value#13L]
----

For all other types of streaming data sources, `DataStreamReader.load` creates a logical query plan with a <<spark-sql-streaming-StreamingRelation.adoc#, StreamingRelation>> leaf logical operator. That is the former *V1 code path*.

.StreamingRelation Logical Operator for Data Source V1
[source, scala]
----
// text data source is V1
val texts = spark.readStream.format("text").load("data/streaming")
val plan = texts.queryExecution.logical
scala> println(plan.numberedTreeString)
00 StreamingRelation DataSource(org.apache.spark.sql.SparkSession@35edd886,text,List(),None,List(),None,Map(path -> data/streaming),None), FileSource[data/streaming], [value#18]
----

=== [[dataset]] Dataset API -- High-Level DSL to Build Logical Query Plan

With a streaming Dataset created, you can now use all the methods of `Dataset` API, including but not limited to the following operators:

* <<spark-sql-streaming-Dataset-operators.adoc#dropDuplicates, Dataset.dropDuplicates>> for streaming deduplication

* <<spark-sql-streaming-Dataset-operators.adoc#groupBy, Dataset.groupBy>> and <<spark-sql-streaming-Dataset-operators.adoc#groupByKey, Dataset.groupByKey>> for streaming aggregation

* <<spark-sql-streaming-Dataset-operators.adoc#withWatermark, Dataset.withWatermark>> for event time watermark

Please note that a streaming Dataset is a regular Dataset (_with some streaming-related limitations_).

[source, scala]
----
val rates = spark
  .readStream
  .format("rate")
  .load
val countByTime = rates
  .withWatermark("timestamp", "10 seconds")
  .groupBy($"timestamp")
  .agg(count("*") as "count")

import org.apache.spark.sql.Dataset
assert(countByTime.isInstanceOf[Dataset[_]])
----

The point is to understand that the Dataset API is a domain-specific language (DSL) to build a more sophisticated stream processing pipeline that you could also build using the low-level logical operators directly.

Use <<spark-sql-streaming-Dataset-operators.adoc#explain, Dataset.explain>> to learn the underlying logical and physical query plans.

[source, scala]
----
assert(countByTime.isStreaming)

scala> countByTime.explain(extended = true)
== Parsed Logical Plan ==
'Aggregate ['timestamp], [unresolvedalias('timestamp, None), count(1) AS count#131L]
+- EventTimeWatermark timestamp#88: timestamp, interval 10 seconds
   +- StreamingRelationV2 org.apache.spark.sql.execution.streaming.sources.RateStreamProvider@2fcb3082, rate, [timestamp#88, value#89L]

== Analyzed Logical Plan ==
timestamp: timestamp, count: bigint
Aggregate [timestamp#88-T10000ms], [timestamp#88-T10000ms, count(1) AS count#131L]
+- EventTimeWatermark timestamp#88: timestamp, interval 10 seconds
   +- StreamingRelationV2 org.apache.spark.sql.execution.streaming.sources.RateStreamProvider@2fcb3082, rate, [timestamp#88, value#89L]

== Optimized Logical Plan ==
Aggregate [timestamp#88-T10000ms], [timestamp#88-T10000ms, count(1) AS count#131L]
+- EventTimeWatermark timestamp#88: timestamp, interval 10 seconds
   +- Project [timestamp#88]
      +- StreamingRelationV2 org.apache.spark.sql.execution.streaming.sources.RateStreamProvider@2fcb3082, rate, [timestamp#88, value#89L]

== Physical Plan ==
*(5) HashAggregate(keys=[timestamp#88-T10000ms], functions=[count(1)], output=[timestamp#88-T10000ms, count#131L])
+- StateStoreSave [timestamp#88-T10000ms], state info [ checkpoint = <unknown>, runId = 28606ba5-9c7f-4f1f-ae41-e28d75c4d948, opId = 0, ver = 0, numPartitions = 200], Append, 0, 2
   +- *(4) HashAggregate(keys=[timestamp#88-T10000ms], functions=[merge_count(1)], output=[timestamp#88-T10000ms, count#136L])
      +- StateStoreRestore [timestamp#88-T10000ms], state info [ checkpoint = <unknown>, runId = 28606ba5-9c7f-4f1f-ae41-e28d75c4d948, opId = 0, ver = 0, numPartitions = 200], 2
         +- *(3) HashAggregate(keys=[timestamp#88-T10000ms], functions=[merge_count(1)], output=[timestamp#88-T10000ms, count#136L])
            +- Exchange hashpartitioning(timestamp#88-T10000ms, 200)
               +- *(2) HashAggregate(keys=[timestamp#88-T10000ms], functions=[partial_count(1)], output=[timestamp#88-T10000ms, count#136L])
                  +- EventTimeWatermark timestamp#88: timestamp, interval 10 seconds
                     +- *(1) Project [timestamp#88]
                        +- StreamingRelation rate, [timestamp#88, value#89L]
----

Or go pro and talk to `QueryExecution` directly.

[source, scala]
----
val plan = countByTime.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'Aggregate ['timestamp], [unresolvedalias('timestamp, None), count(1) AS count#131L]
01 +- EventTimeWatermark timestamp#88: timestamp, interval 10 seconds
02    +- StreamingRelationV2 org.apache.spark.sql.execution.streaming.sources.RateStreamProvider@2fcb3082, rate, [timestamp#88, value#89L]
----

Please note that most of the stream processing operators you may also have used in batch structured queries in Spark SQL. Again, the distinction between Spark SQL and Spark Structured Streaming is very thin from a developer's point of view.

=== [[DataStreamWriter]] DataStreamWriter and Streaming Data Sink

Once you're satisfied with building a stream processing pipeline (using the APIs of <<DataStreamReader, DataStreamReader>>, <<spark-sql-streaming-Dataset-operators.adoc#, Dataset>>, `RelationalGroupedDataset` and `KeyValueGroupedDataset`), you should define how and when the result of the streaming query is persisted in (_sent out to_) an external data system using a <<spark-sql-streaming-Sink.adoc#, streaming sink>>.

TIP: Read up on the APIs of https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-dataset-operators.html[Dataset], https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-RelationalGroupedDataset.html[RelationalGroupedDataset] and https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-KeyValueGroupedDataset.html[KeyValueGroupedDataset] in https://bit.ly/spark-sql-internals[The Internals of Spark SQL] book.

You should use <<spark-sql-streaming-Dataset-operators.adoc#writeStream, Dataset.writeStream>> method that simply creates a <<spark-sql-streaming-DataStreamWriter.adoc#, DataStreamWriter>>.

[source, scala]
----
// Not only is this a Dataset, but it is also streaming
assert(countByTime.isStreaming)

val writer = countByTime.writeStream

import org.apache.spark.sql.streaming.DataStreamWriter
assert(writer.isInstanceOf[DataStreamWriter[_]])
----

The fluent API of `DataStreamWriter` allows you to describe the output data sink (e.g. <<spark-sql-streaming-DataStreamWriter.adoc#format, DataStreamWriter.format>> and <<spark-sql-streaming-DataStreamWriter.adoc#options, DataStreamWriter.options>>) using method chaining (with the goal of making the readability of the source code close to that of ordinary written prose, essentially creating a domain-specific language within the interface. See https://en.wikipedia.org/wiki/Fluent_interface[Fluent interface] article in Wikipedia).

[source, scala]
----
writer
  .format("csv")
  .option("delimiter", "\t")
----

Like in <<DataStreamReader, DataStreamReader>> data source formats, there are a couple of built-in data sink formats. Unlike data source formats, their names do not have corresponding `DataStreamWriter` methods. The reason is that you will use <<spark-sql-streaming-DataStreamWriter.adoc#start, DataStreamWriter.start>> to create and immediately start a <<spark-sql-streaming-StreamingQuery.adoc#, StreamingQuery>>.

There are however two special output formats that do have corresponding `DataStreamWriter` methods, i.e. <<spark-sql-streaming-DataStreamWriter.adoc#foreach, DataStreamWriter.foreach>> and <<spark-sql-streaming-DataStreamWriter.adoc#foreachBatch, DataStreamWriter.foreachBatch>>, that allow for persisting query results to external data systems that do not have streaming sinks available. They give you a trade-off between developing a full-blown streaming sink and simply using the methods (that lay the basis of what a custom sink would have to do anyway).

`DataStreamWriter` API defines two new concepts (that are not available in the "base" Spark SQL):

* <<spark-sql-streaming-OutputMode.adoc#, OutputMode>> that you specify using <<spark-sql-streaming-DataStreamWriter.adoc#outputMode, DataStreamWriter.outputMode>> method

* <<spark-sql-streaming-Trigger.adoc#, Trigger>> that you specify using <<spark-sql-streaming-DataStreamWriter.adoc#trigger, DataStreamWriter.trigger>> method

You may also want to give a streaming query a name using <<spark-sql-streaming-DataStreamWriter.adoc#queryName, DataStreamWriter.queryName>> method.

In the end, you use <<spark-sql-streaming-DataStreamWriter.adoc#start, DataStreamWriter.start>> method to create and immediately start a <<spark-sql-streaming-StreamingQuery.adoc#, StreamingQuery>>.

[source, scala]
----
import org.apache.spark.sql.streaming.OutputMode
import org.apache.spark.sql.streaming.Trigger
import scala.concurrent.duration._
val sq = writer
  .format("console")
  .option("truncate", false)
  .option("checkpointLocation", "/tmp/csv-to-csv-checkpoint")
  .outputMode(OutputMode.Append)
  .trigger(Trigger.ProcessingTime(30.seconds))
  .queryName("csv-to-csv")
  .start("/tmp")

import org.apache.spark.sql.streaming.StreamingQuery
assert(sq.isInstanceOf[StreamingQuery])
----

When `DataStreamWriter` is requested to <<spark-sql-streaming-DataStreamWriter.adoc#start, start a streaming query>>, it allows for the following data source formats:

* *memory* with <<spark-sql-streaming-MemorySinkV2.adoc#, MemorySinkV2>> (with <<spark-sql-streaming-Trigger.adoc#ContinuousTrigger, ContinuousTrigger>>) or <<spark-sql-streaming-MemorySink.adoc#, MemorySink>>

* *foreach* with <<spark-sql-streaming-ForeachWriterProvider.adoc#, ForeachWriterProvider>> sink

* *foreachBatch* with <<spark-sql-streaming-ForeachBatchSink.adoc#, ForeachBatchSink>> sink (that does not support <<spark-sql-streaming-Trigger.adoc#ContinuousTrigger, ContinuousTrigger>>)

* Any `DataSourceRegister` data source

* Custom data sources specified by their fully-qualified class names or *[name].DefaultSource*

* *avro*, *kafka* _and some others_ (see `DataSource.lookupDataSource` object method)

* <<spark-sql-streaming-StreamWriteSupport.adoc#, StreamWriteSupport>>

* `DataSource` is requested to <<spark-sql-streaming-DataSource.adoc#createSink, create a streaming sink>> that accepts <<spark-sql-streaming-StreamSinkProvider.adoc#, StreamSinkProvider>> or `FileFormat` data sources only

With a streaming sink, `DataStreamWriter` requests the <<StreamingQueryManager, StreamingQueryManager>> to <<spark-sql-streaming-StreamingQueryManager.adoc#startQuery, start a streaming query>>.

=== [[StreamingQuery]] StreamingQuery

When a stream processing pipeline is started (using <<spark-sql-streaming-DataStreamWriter.adoc#start, DataStreamWriter.start>> method), `DataStreamWriter` creates a <<spark-sql-streaming-StreamingQuery.adoc#, StreamingQuery>> and requests the <<StreamingQueryManager, StreamingQueryManager>> to <<spark-sql-streaming-StreamingQueryManager.adoc#startQuery, start a streaming query>>.

=== [[StreamingQueryManager]] StreamingQueryManager

<<spark-sql-streaming-StreamingQueryManager.adoc#, StreamingQueryManager>> is used to manage streaming queries.
