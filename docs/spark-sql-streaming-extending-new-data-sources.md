== Extending Structured Streaming with New Data Sources

Spark Structured Streaming uses Spark SQL for planning streaming queries (_preparing for execution_).

Spark SQL is migrating from the former Data Source API V1 to a new Data Source API V2, and so is Structured Streaming. That is exactly the reason for <<spark-sql-streaming-BaseStreamingSource.md#, BaseStreamingSource>> and <<spark-sql-streaming-BaseStreamingSink.md#, BaseStreamingSink>> APIs for the two different Data Source API's class hierarchies, for streaming sources and sinks, respectively.

Structured Streaming supports two <<spark-sql-streaming-StreamExecution.md#, stream execution engines>> (i.e. <<spark-sql-streaming-micro-batch-stream-processing.md#, Micro-Batch>> and <<spark-sql-streaming-continuous-stream-processing.md#, Continuous>>) with their own APIs.

<<spark-sql-streaming-micro-batch-stream-processing.md#, Micro-Batch Stream Processing>> supports the old Data Source API V1 and the new modern Data Source API V2 with micro-batch-specific APIs for streaming sources and sinks.

<<spark-sql-streaming-continuous-stream-processing.md#, Continuous Stream Processing>> supports the new modern Data Source API V2 only with continuous-specific APIs for streaming sources and sinks.

The following are the questions to think of (and answer) while considering development of a new data source for Structured Streaming. They are supposed to give you a sense of how much work and time it takes as well as what Spark version to support (e.g. 2.2 vs 2.4).

* Data Source API V1
* Data Source API V2
* <<spark-sql-streaming-micro-batch-stream-processing.md#, Micro-Batch Stream Processing>> (_Structured Streaming V1_)
* <<spark-sql-streaming-continuous-stream-processing.md#, Continuous Stream Processing>> (_Structured Streaming V2_)
* Read side (<<spark-sql-streaming-BaseStreamingSource.md#, BaseStreamingSource>>)
* Write side (<<spark-sql-streaming-BaseStreamingSink.md#, BaseStreamingSink>>)
