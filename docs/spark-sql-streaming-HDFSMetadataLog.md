== [[HDFSMetadataLog]] HDFSMetadataLog -- Hadoop DFS-based Metadata Storage

`HDFSMetadataLog` is a concrete <<spark-sql-streaming-MetadataLog.adoc#, metadata storage>> (of type `T`) that uses Hadoop DFS for fault-tolerance and reliability.

[[metadataPath]]
`HDFSMetadataLog` uses the given <<path, path>> as the *metadata directory* with metadata logs. The path is immediately converted to a Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/fs/Path.html[Path] for file management.

[[formats]]
`HDFSMetadataLog` uses http://json4s.org/[Json4s] with the https://github.com/FasterXML/jackson-databind[Jackson] binding for metadata <<serialize, serialization>> and <<deserialize, deserialization>> (to and from JSON format).

`HDFSMetadataLog` is further customized by the <<extensions, extensions>>.

[[extensions]]
.HDFSMetadataLogs (Direct Extensions Only)
[cols="30,70",options="header",width="100%"]
|===
| HDFSMetadataLog
| Description

| _Anonymous_
| [[KafkaSource]] `HDFSMetadataLog` of <<spark-sql-streaming-KafkaSourceOffset.adoc#, KafkaSourceOffsets>> for <<spark-sql-streaming-KafkaSource.adoc#, KafkaSource>>

| _Anonymous_
| [[RateStreamMicroBatchReader]] `HDFSMetadataLog` of <<spark-sql-streaming-Offset.adoc#LongOffset, LongOffsets>> for <<spark-sql-streaming-RateStreamMicroBatchReader.adoc#, RateStreamMicroBatchReader>>

| <<spark-sql-streaming-CommitLog.adoc#, CommitLog>>
| [[CommitLog]] <<spark-sql-streaming-StreamExecution.adoc#commitLog, Offset commit log>> of <<spark-sql-streaming-StreamExecution.adoc#, streaming query execution engines>>

| <<spark-sql-streaming-CompactibleFileStreamLog.adoc#, CompactibleFileStreamLog>>
| [[CompactibleFileStreamLog]] Compactible metadata logs (that compact logs at regular interval)

| <<spark-sql-streaming-KafkaSourceInitialOffsetWriter.adoc#, KafkaSourceInitialOffsetWriter>>
| [[KafkaSourceInitialOffsetWriter]] `HDFSMetadataLog` of <<spark-sql-streaming-KafkaSourceOffset.adoc#, KafkaSourceOffsets>> for <<spark-sql-streaming-KafkaSource.adoc#, KafkaSource>>

| <<spark-sql-streaming-OffsetSeqLog.adoc#, OffsetSeqLog>>
| [[OffsetSeqLog]] <<spark-sql-streaming-StreamExecution.adoc#offsetLog, Write-Ahead Log (WAL)>> of <<spark-sql-streaming-StreamExecution.adoc#, stream execution engines>>

|===

=== [[creating-instance]] Creating HDFSMetadataLog Instance

`HDFSMetadataLog` takes the following to be created:

* [[sparkSession]] `SparkSession`
* [[path]] Path of the metadata log directory

While being <<creating-instance, created>> `HDFSMetadataLog` creates the <<path, path>> unless exists already.

=== [[serialize]] Serializing Metadata (Writing Metadata in Serialized Format) -- `serialize` Method

[source, scala]
----
serialize(
  metadata: T,
  out: OutputStream): Unit
----

`serialize` simply writes the log data (serialized using <<formats, Json4s (with Jackson binding)>> library).

NOTE: `serialize` is used exclusively when `HDFSMetadataLog` is requested to <<writeBatchToFile, write metadata of a streaming batch to a file (metadata log)>> (when <<add, storing metadata of a streaming batch>>).

=== [[deserialize]] Deserializing Metadata (Reading Metadata from Serialized Format) -- `deserialize` Method

[source, scala]
----
deserialize(in: InputStream): T
----

`deserialize` deserializes a metadata (of type `T`) from a given `InputStream`.

NOTE: `deserialize` is used exclusively when `HDFSMetadataLog` is requested to <<get, retrieve metadata of a batch>>.

=== [[get]][[get-batchId]] Retrieving Metadata Of Streaming Batch -- `get` Method

[source, scala]
----
get(batchId: Long): Option[T]
----

NOTE: `get` is part of the <<spark-sql-streaming-MetadataLog.adoc#get, MetadataLog Contract>> to get metadata of a batch.

`get`...FIXME

=== [[get-range]] Retrieving Metadata of Range of Batches -- `get` Method

[source, scala]
----
get(
  startId: Option[Long],
  endId: Option[Long]): Array[(Long, T)]
----

NOTE: `get` is part of the <<spark-sql-streaming-MetadataLog.adoc#get, MetadataLog Contract>> to get metadata of range of batches.

`get`...FIXME

=== [[add]] Persisting Metadata of Streaming Micro-Batch -- `add` Method

[source, scala]
----
add(
  batchId: Long,
  metadata: T): Boolean
----

NOTE: `add` is part of the <<spark-sql-streaming-MetadataLog.adoc#add, MetadataLog Contract>> to persist metadata of a streaming batch.

`add` return `true` when the metadata of the streaming batch was not available and persisted successfully. Otherwise, `add` returns `false`.

Internally, `add` <<get, looks up metadata of the given streaming batch>> (`batchId`) and returns `false` when found.

Otherwise, when not found, `add` <<batchIdToPath, creates a metadata log file>> for the given `batchId` and <<writeBatchToFile, writes metadata to the file>>. `add` returns `true` if successful.

=== [[getLatest]] Latest Committed Batch Id with Metadata (When Available) -- `getLatest` Method

[source, scala]
----
getLatest(): Option[(Long, T)]
----

NOTE: `getLatest` is a part of link:spark-sql-streaming-MetadataLog.adoc#getLatest[MetadataLog Contract] to retrieve the recently-committed batch id and the corresponding metadata if available in the metadata storage.

`getLatest` requests the internal <<fileManager, FileManager>> for the files in <<metadataPath, metadata directory>> that match <<batchFilesFilter, batch file filter>>.

`getLatest` takes the batch ids (the batch files correspond to) and sorts the ids in reverse order.

`getLatest` gives the first batch id with the metadata which <<get, could be found in the metadata storage>>.

NOTE: It is possible that the batch id could be in the metadata storage, but not available for retrieval.

=== [[purge]] Removing Expired Metadata (Purging) -- `purge` Method

[source, scala]
----
purge(thresholdBatchId: Long): Unit
----

NOTE: `purge` is part of the <<spark-sql-streaming-MetadataLog.adoc#purge, MetadataLog Contract>> to...FIXME.

`purge`...FIXME

=== [[batchIdToPath]] Creating Batch Metadata File -- `batchIdToPath` Method

[source, scala]
----
batchIdToPath(batchId: Long): Path
----

`batchIdToPath` simply creates a Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/fs/Path.html[Path] for the file called by the specified `batchId` under the <<metadataPath, metadata directory>>.

[NOTE]
====
`batchIdToPath` is used when:

* `CompactibleFileStreamLog` is requested to <<spark-sql-streaming-CompactibleFileStreamLog.adoc#compact, compact>> and <<spark-sql-streaming-CompactibleFileStreamLog.adoc#allFiles, allFiles>>

* `HDFSMetadataLog` is requested to <<add, add>>, <<get, get>>, <<purge, purge>>, and <<purgeAfter, purgeAfter>>
====

=== [[isBatchFile]] `isBatchFile` Method

[source, scala]
----
isBatchFile(path: Path): Boolean
----

`isBatchFile`...FIXME

NOTE: `isBatchFile` is used exclusively when `HDFSMetadataLog` is requested for the <<batchFilesFilter, PathFilter of batch files>>.

=== [[pathToBatchId]] `pathToBatchId` Method

[source, scala]
----
pathToBatchId(path: Path): Long
----

`pathToBatchId`...FIXME

[NOTE]
====
`pathToBatchId` is used when:

* `CompactibleFileStreamLog` is requested for the <<spark-sql-streaming-CompactibleFileStreamLog.adoc#compactInterval, compact interval>>

* `HDFSMetadataLog` is requested to <<isBatchFile, isBatchFile>>, <<get-range, get metadata of a range of batches>>, <<getLatest, getLatest>>, <<getOrderedBatchFiles, getOrderedBatchFiles>>, <<purge, purge>>, and <<purgeAfter, purgeAfter>>
====

=== [[verifyBatchIds]] `verifyBatchIds` Object Method

[source, scala]
----
verifyBatchIds(
  batchIds: Seq[Long],
  startId: Option[Long],
  endId: Option[Long]): Unit
----

`verifyBatchIds`...FIXME

[NOTE]
====
`verifyBatchIds` is used when:

* `FileStreamSourceLog` is requested to <<spark-sql-streaming-FileStreamSourceLog.adoc#get, get>>

* `HDFSMetadataLog` is requested to <<get-range, get>>
====

=== [[parseVersion]] Retrieving Version (From Text Line) -- `parseVersion` Internal Method

[source, scala]
----
parseVersion(
  text: String,
  maxSupportedVersion: Int): Int
----

`parseVersion`...FIXME

[NOTE]
====
`parseVersion` is used when:

* `KafkaSourceInitialOffsetWriter` is requested to <<spark-sql-streaming-KafkaSourceInitialOffsetWriter.adoc#deserialize, deserialize metadata>>

* `KafkaSource` is requested for the <<spark-sql-streaming-KafkaSource.adoc#initialPartitionOffsets, initial partition offsets>>

* `CommitLog` is requested to <<spark-sql-streaming-CommitLog.adoc#deserialize, deserialize metadata>>

* `CompactibleFileStreamLog` is requested to <<spark-sql-streaming-CompactibleFileStreamLog.adoc#deserialize, deserialize metadata>>

* `OffsetSeqLog` is requested to <<spark-sql-streaming-OffsetSeqLog.adoc#deserialize, deserialize metadata>>

* `RateStreamMicroBatchReader` is requested to <<spark-sql-streaming-RateStreamMicroBatchReader.adoc#deserialize, deserialize metadata>>
====

=== [[purgeAfter]] `purgeAfter` Method

[source, scala]
----
purgeAfter(thresholdBatchId: Long): Unit
----

`purgeAfter`...FIXME

NOTE: `purgeAfter` seems to be used exclusively in tests.

=== [[writeBatchToFile]] Writing Batch Metadata to File (Metadata Log) -- `writeBatchToFile` Internal Method

[source, scala]
----
writeBatchToFile(
  metadata: T,
  path: Path): Unit
----

`writeBatchToFile` requests the <<fileManager, CheckpointFileManager>> to <<spark-sql-streaming-CheckpointFileManager.adoc#createAtomic, createAtomic>> (for the specified `path` and the `overwriteIfPossible` flag disabled).

`writeBatchToFile` then <<serialize, serializes the metadata>> (to the `CancellableFSDataOutputStream` output stream) and closes the stream.

In case of an exception, `writeBatchToFile` simply requests the `CancellableFSDataOutputStream` output stream to `cancel` (so that the output file is not generated) and re-throws the exception.

NOTE: `writeBatchToFile` is used exclusively when `HDFSMetadataLog` is requested to <<add, store (persist) metadata of a streaming batch>>.

=== [[getOrderedBatchFiles]] Retrieving Ordered Batch Metadata Files -- `getOrderedBatchFiles` Method

[source, scala]
----
getOrderedBatchFiles(): Array[FileStatus]
----

`getOrderedBatchFiles`...FIXME

NOTE: `getOrderedBatchFiles` does not seem to be used at all.

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| batchFilesFilter
a| [[batchFilesFilter]] Hadoop's https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/fs/PathFilter.html[PathFilter] of <<isBatchFile, batch files>> (with names being long numbers)

Used when:

* `CompactibleFileStreamLog` is requested for the <<spark-sql-streaming-CompactibleFileStreamLog.adoc#compactInterval, compactInterval>>

* `HDFSMetadataLog` is requested to <<get, get batch metadata>>, <<getLatest, getLatest>>, <<getOrderedBatchFiles, getOrderedBatchFiles>>, <<purge, purge>>, and <<purgeAfter, purgeAfter>>

| fileManager
a| [[fileManager]] <<spark-sql-streaming-CheckpointFileManager.adoc#, CheckpointFileManager>>

Used when...FIXME

|===
