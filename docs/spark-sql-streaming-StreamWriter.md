== [[StreamWriter]] StreamWriter Contract

`StreamWriter` is the <<contract, extension>> of the `DataSourceWriter` contract to support epochs, i.e. <<implementations, streaming writers>> that can <<abort, abort>> and <<commit, commit>> writing jobs for a specified epoch.

TIP: Read up on https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-DataSourceWriter.html[DataSourceWriter] in https://bit.ly/spark-sql-internals[The Internals of Spark SQL] book.

[[contract]]
.StreamWriter Contract
[cols="1m,3",options="header",width="100%"]
|===
| Method
| Description

| abort
a| [[abort]]

[source, java]
----
void abort(
  long epochId,
  WriterCommitMessage[] messages)
----

Aborts the writing job for a specified `epochId` and `WriterCommitMessages`

Used exclusively when `MicroBatchWriter` is requested to <<spark-sql-streaming-MicroBatchWriter.adoc#abort, abort>>

| commit
a| [[commit]]

[source, java]
----
void commit(
  long epochId,
  WriterCommitMessage[] messages)
----

Commits the writing job for a specified `epochId` and `WriterCommitMessages`

Used when:

* `EpochCoordinator` is requested to <<spark-sql-streaming-EpochCoordinator.adoc#commitEpoch, commitEpoch>>

* `MicroBatchWriter` is requested to <<spark-sql-streaming-MicroBatchWriter.adoc#commit, commit>>

|===

[[implementations]]
.StreamWriters
[cols="1,3",options="header",width="100%"]
|===
| StreamWriter
| Description

| <<spark-sql-streaming-ForeachWriterProvider.adoc#, ForeachWriterProvider>>
| [[ForeachWriterProvider]] *foreachWriter* data source

| <<spark-sql-streaming-ConsoleWriter.adoc#, ConsoleWriter>>
| [[ConsoleWriter]] *console* data source

| KafkaStreamWriter
| [[KafkaStreamWriter]] *kafka* data source

| MemoryStreamWriter
| [[MemoryStreamWriter]] *memory* data source

|===
