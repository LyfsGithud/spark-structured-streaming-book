== Spark Structured Streaming and Streaming Queries

*Spark Structured Streaming* (aka _Structured Streaming_ or _Spark Streams_) is the module of Apache Spark for stream processing using *streaming queries*.

Streaming queries can be expressed using a <<spark-sql-streaming-Dataset-operators.adoc#, high-level declarative streaming API>> (_Dataset API_) or good ol' SQL (_SQL over stream_ / _streaming SQL_). The declarative streaming Dataset API and SQL are executed on the underlying highly-optimized Spark SQL engine.

The semantics of the Structured Streaming model is as follows (see the article https://databricks.com/blog/2016/07/28/structured-streaming-in-apache-spark.html[Structured Streaming In Apache Spark]):

> At any time, the output of a continuous application is equivalent to executing a batch job on a prefix of the data.

NOTE: As of Spark 2.2.0, Structured Streaming has been marked stable and ready for production use. With that the other older streaming module *Spark Streaming* is considered obsolete and not used for developing new streaming applications with Apache Spark.

Spark Structured Streaming comes with two <<spark-sql-streaming-StreamExecution.adoc#, stream execution engines>> for executing streaming queries:

* <<spark-sql-streaming-MicroBatchExecution.adoc#, MicroBatchExecution>> for <<spark-sql-streaming-micro-batch-stream-processing.adoc#, Micro-Batch Stream Processing>>

* <<spark-sql-streaming-ContinuousExecution.adoc#, ContinuousExecution>> for <<spark-sql-streaming-continuous-stream-processing.adoc#, Continuous Stream Processing>>

The goal of Spark Structured Streaming is to unify streaming, interactive, and batch queries over structured datasets for developing end-to-end stream processing applications dubbed *continuous applications* using Spark SQL's Datasets API with additional support for the following features:

* <<spark-sql-streaming-aggregation.adoc#, Streaming Aggregation>>

* <<spark-sql-streaming-join.adoc#, Streaming Join>>

* <<spark-sql-streaming-watermark.adoc#, Streaming Watermark>>

* <<spark-sql-arbitrary-stateful-streaming-aggregation.adoc#, Arbitrary Stateful Streaming Aggregation>>

* <<spark-sql-streaming-stateful-stream-processing.adoc#, Stateful Stream Processing>>

In Structured Streaming, Spark developers describe custom streaming computations in the same way as with Spark SQL. Internally, Structured Streaming applies the user-defined structured query to the continuously and indefinitely arriving data to analyze real-time streaming data.

Structured Streaming introduces the concept of *streaming datasets* that are _infinite datasets_ with primitives like input link:spark-sql-streaming-Source.adoc[streaming data sources] and output link:spark-sql-streaming-Sink.adoc[streaming data sinks].

[TIP]
====
A `Dataset` is *streaming* when its logical plan is streaming.

[source, scala]
----
val batchQuery = spark.
  read. // <-- batch non-streaming query
  csv("sales")

assert(batchQuery.isStreaming == false)

val streamingQuery = spark.
  readStream. // <-- streaming query
  format("rate").
  load

assert(streamingQuery.isStreaming)
----

Read up on Spark SQL, Datasets and logical plans in https://bit.ly/spark-sql-internals[The Internals of Spark SQL] book.
====

Structured Streaming models a stream of data as an infinite (and hence continuous) table that could be changed every streaming batch.

You can specify link:spark-sql-streaming-OutputMode.adoc[output mode] of a streaming dataset which is what gets written to a streaming sink (i.e. the infinite result table) when there is a new data available.

Streaming Datasets use *streaming query plans* (as opposed to regular batch Datasets that are based on batch query plans).

[NOTE]
====
From this perspective, batch queries can be considered streaming Datasets executed once only (and is why some batch queries, e.g. link:spark-sql-streaming-KafkaSource.adoc[KafkaSource], can easily work in batch mode).

[source, scala]
----
val batchQuery = spark.read.format("rate").load

assert(batchQuery.isStreaming == false)

val streamingQuery = spark.readStream.format("rate").load

assert(streamingQuery.isStreaming)
----
====

With Structured Streaming, Spark 2 aims at simplifying *streaming analytics* with little to no need to reason about effective data streaming (trying to hide the unnecessary complexity in your streaming analytics architectures).

Structured streaming is defined by the following data abstractions in `org.apache.spark.sql.streaming` package:

* link:spark-sql-streaming-StreamingQuery.adoc[StreamingQuery]
* link:spark-sql-streaming-Source.adoc[Streaming Source]
* link:spark-sql-streaming-Sink.adoc[Streaming Sink]
* link:spark-sql-streaming-StreamingQueryManager.adoc[StreamingQueryManager]

Structured Streaming follows micro-batch model and periodically fetches data from the data source (and uses the `DataFrame` data abstraction to represent the fetched data for a certain batch).

With Datasets as Spark SQL's view of structured data, structured streaming checks input sources for new data every link:spark-sql-streaming-Trigger.adoc[trigger] (time) and executes the (continuous) queries.

NOTE: The feature has also been called *Streaming Spark SQL Query*, *Streaming DataFrames*, *Continuous DataFrame* or *Continuous Query*. There have been lots of names before the Spark project settled on Structured Streaming.

=== [[i-want-more]] Further Reading Or Watching

* https://issues.apache.org/jira/browse/SPARK-8360[SPARK-8360 Structured Streaming (aka Streaming DataFrames)]

* The official http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html[Structured Streaming Programming Guide]

* (article) https://databricks.com/blog/2016/07/28/structured-streaming-in-apache-spark.html[Structured Streaming In Apache Spark]

* (video) https://youtu.be/oXkxXDG0gNk[The Future of Real Time in Spark] from Spark Summit East 2016 in which Reynold Xin presents the concept of *Streaming DataFrames*

* (video) https://youtu.be/i7l3JQRx7Qw?t=19m15s[Structuring Spark: DataFrames, Datasets, and Streaming]

* (article) http://www.infoworld.com/article/3052924/analytics/what-sparks-structured-streaming-really-means.html[What Spark's Structured Streaming really means]

* (video) https://youtu.be/rl8dIzTpxrI[A Deep Dive Into Structured Streaming] by Tathagata "TD" Das from Spark Summit 2016

* (video) https://youtu.be/rl8dIzTpxrI[Arbitrary Stateful Aggregations in Structured Streaming in Apache Spark] by Burak Yavuz
