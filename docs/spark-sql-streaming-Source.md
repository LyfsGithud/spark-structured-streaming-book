== [[Source]] Source Contract -- Streaming Sources for Micro-Batch Stream Processing (Data Source API V1)

`Source` is the <<contract, extension>> of the <<spark-sql-streaming-BaseStreamingSource.adoc#, BaseStreamingSource contract>> for <<implementations, streaming sources>> that work with "continuous" stream of data identified by <<spark-sql-streaming-Offset.adoc#, offsets>>.

`Source` is part of Data Source API V1 and used in <<spark-sql-streaming-micro-batch-stream-processing.adoc#, Micro-Batch Stream Processing>> only.

For fault tolerance, `Source` must be able to replay an arbitrary sequence of past data in a stream using a range of offsets. This is the assumption so Structured Streaming can achieve end-to-end exactly-once guarantees.

[[contract]]
.Source Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| commit
a| [[commit]]

[source, scala]
----
commit(end: Offset): Unit
----

Commits data up to the end <<spark-sql-streaming-Offset.adoc#, offset>>, i.e. informs the source that Spark has completed processing all data for offsets less than or equal to the end offset and will only request offsets greater than the end offset in the future.

Used exclusively when <<spark-sql-streaming-MicroBatchExecution.adoc#, MicroBatchExecution>> stream execution engine (<<spark-sql-streaming-micro-batch-stream-processing.adoc#, Micro-Batch Stream Processing>>) is requested to <<spark-sql-streaming-MicroBatchExecution.adoc#constructNextBatch-walCommit, write offsets to a commit log (walCommit phase)>> while <<spark-sql-streaming-MicroBatchExecution.adoc#runActivatedStream, running an activated streaming query>>.

| getBatch
a| [[getBatch]]

[source, scala]
----
getBatch(
  start: Option[Offset],
  end: Offset): DataFrame
----

Generating a streaming `DataFrame` with data between the start and end <<spark-sql-streaming-Offset.adoc#, offsets>>

Start offset can be undefined (`None`) to indicate that the batch should begin with the first record

Used when <<spark-sql-streaming-MicroBatchExecution.adoc#, MicroBatchExecution>> stream execution engine (<<spark-sql-streaming-micro-batch-stream-processing.adoc#, Micro-Batch Stream Processing>>) is requested to <<spark-sql-streaming-MicroBatchExecution.adoc#runActivatedStream, run an activated streaming query>>, namely:

* <<spark-sql-streaming-MicroBatchExecution.adoc#populateStartOffsets, Populate start offsets from checkpoint (resuming from checkpoint)>>

* <<spark-sql-streaming-MicroBatchExecution.adoc#runBatch-getBatch, Request unprocessed data from all sources (getBatch phase)>>

| getOffset
a| [[getOffset]]

[source, scala]
----
getOffset: Option[Offset]
----

Latest (maximum) <<spark-sql-streaming-Offset.adoc#, offset>> of the source (or `None` to denote no data)

Used exclusively when <<spark-sql-streaming-MicroBatchExecution.adoc#, MicroBatchExecution>> stream execution engine (<<spark-sql-streaming-micro-batch-stream-processing.adoc#, Micro-Batch Stream Processing>>) is requested for <<spark-sql-streaming-MicroBatchExecution.adoc#constructNextBatch-getOffset, latest offsets of all sources (getOffset phase)>> while <<spark-sql-streaming-MicroBatchExecution.adoc#runActivatedStream, running activated streaming query>>.

| schema
a| [[schema]]

[source, scala]
----
schema: StructType
----

Schema of the source

NOTE: `schema` _seems_ to be used for tests only and a duplication of <<spark-sql-streaming-StreamSourceProvider.adoc#sourceSchema, StreamSourceProvider.sourceSchema>>.

|===

[[implementations]]
.Sources
[cols="30,70",options="header",width="100%"]
|===
| Source
| Description

| <<spark-sql-streaming-FileStreamSource.adoc#, FileStreamSource>>
| [[FileStreamSource]] Part of file-based data sources (`FileFormat`)

| <<spark-sql-streaming-KafkaSource.adoc#, KafkaSource>>
| [[KafkaSource]] Part of <<spark-sql-streaming-kafka-data-source.adoc#, kafka>> data source

|===
