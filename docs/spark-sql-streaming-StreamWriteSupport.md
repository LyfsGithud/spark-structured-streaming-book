== [[StreamWriteSupport]] StreamWriteSupport Contract -- Writable Streaming Data Sources

`StreamWriteSupport` is the <<contract, abstraction>> of <<implementations, DataSourceV2 sinks>> that <<createStreamWriter, create StreamWriters>> for streaming write (when used in streaming queries in <<spark-sql-streaming-MicroBatchExecution.adoc#, MicroBatchExecution>> and <<spark-sql-streaming-ContinuousExecution.adoc#, ContinuousExecution>>).

[[contract]][[createStreamWriter]]
[source, java]
----
StreamWriter createStreamWriter(
  String queryId,
  StructType schema,
  OutputMode mode,
  DataSourceOptions options)
----

`createStreamWriter` creates a <<spark-sql-streaming-StreamWriter.adoc#, StreamWriter>> for streaming write and is used when the <<spark-sql-streaming-StreamExecution.adoc#queryExecutionThread, stream execution thread for a streaming query>> is <<spark-sql-streaming-StreamExecution.adoc#start, started>> and requests the stream execution engines to start, i.e.

* `ContinuousExecution` is requested to <<spark-sql-streaming-ContinuousExecution.adoc#runContinuous, runContinuous>>

* `MicroBatchExecution` is requested to <<spark-sql-streaming-MicroBatchExecution.adoc#runBatch, run a single streaming batch>>

[[implementations]]
.StreamWriteSupports
[cols="1,2",options="header",width="100%"]
|===
| StreamWriteSupport
| Description

| <<spark-sql-streaming-ConsoleSinkProvider.adoc#, ConsoleSinkProvider>>
| [[ConsoleSinkProvider]] Streaming sink for `console` data source format

| <<spark-sql-streaming-ForeachWriterProvider.adoc#, ForeachWriterProvider>>
| [[ForeachWriterProvider]]

| <<spark-sql-streaming-KafkaSourceProvider.adoc#, KafkaSourceProvider>>
| [[KafkaSourceProvider]]

| <<spark-sql-streaming-MemorySinkV2.adoc#, MemorySinkV2>>
| [[MemorySinkV2]]
|===
