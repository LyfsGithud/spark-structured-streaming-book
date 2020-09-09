== [[Offset]] Offset -- Read Position of Streaming Query

`Offset` is the <<contract, base>> of <<extensions, stream positions>> that represent progress of a streaming query in <<json, json>> format.

[[contract]]
.Offset Contract (Abstract Methods Only)
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| json
a| [[json]]

[source, java]
----
String json()
----

Converts the offset to JSON format (JSON-encoded offset)

Used when:

* `MicroBatchExecution` stream execution engine is requested to <<spark-sql-streaming-MicroBatchExecution.adoc#constructNextBatch, construct the next streaming micro-batch>> and <<spark-sql-streaming-MicroBatchExecution.adoc#runBatch, run a streaming micro-batch>> (with <<spark-sql-streaming-MicroBatchReader.adoc#, MicroBatchReader>> sources)

* `OffsetSeq` is requested for the <<spark-sql-streaming-OffsetSeq.adoc#toString, textual representation>>

* `OffsetSeqLog` is requested to <<spark-sql-streaming-OffsetSeqLog.adoc#serialize, serialize metadata (write metadata in serialized format)>>

* `ProgressReporter` is requested to <<spark-sql-streaming-ProgressReporter.adoc#recordTriggerOffsets, record trigger offsets>>

* `ContinuousExecution` stream execution engine is requested to <<spark-sql-streaming-ContinuousExecution.adoc#runContinuous, run a streaming query in continuous mode>> and <<spark-sql-streaming-ContinuousExecution.adoc#commit, commit an epoch>>

|===

[[extensions]]
.Offsets
[cols="30,70",options="header",width="100%"]
|===
| Offset
| Description

| ContinuousMemoryStreamOffset
| [[ContinuousMemoryStreamOffset]]

| FileStreamSourceOffset
| [[FileStreamSourceOffset]]

| <<spark-sql-streaming-KafkaSourceOffset.adoc#, KafkaSourceOffset>>
| [[KafkaSourceOffset]]

| LongOffset
| [[LongOffset]]

| RateStreamOffset
| [[RateStreamOffset]]

| SerializedOffset
| [[SerializedOffset]] JSON-encoded offset that is used when loading an offset from an external storage, e.g. from <<spark-sql-streaming-offsets-and-metadata-checkpointing.adoc#, checkpoint>> after restart

| TextSocketOffset
| [[TextSocketOffset]]

|===
