# FileStreamSource

`FileStreamSource` is a [Source](Source.md) that reads text files from `path` directory as they appear. It uses `LongOffset` offsets.

NOTE: It is used by spark-sql-datasource.md#createSource[DataSource.createSource] for `FileFormat`.

You can provide the <<schema, schema>> of the data and `dataFrameBuilder` - the function to build a `DataFrame` in <<getBatch, getBatch>> at instantiation time.

[source, scala]
----
// NOTE The source directory must exist
// mkdir text-logs

val df = spark.readStream
  .format("text")
  .option("maxFilesPerTrigger", 1)
  .load("text-logs")

scala> df.printSchema
root
|-- value: string (nullable = true)
----

Batches are indexed.

It lives in `org.apache.spark.sql.execution.streaming` package.

[source, scala]
----
import org.apache.spark.sql.types._
val schema = StructType(
  StructField("id", LongType, nullable = false) ::
  StructField("name", StringType, nullable = false) ::
  StructField("score", DoubleType, nullable = false) :: Nil)

// You should have input-json directory available
val in = spark.readStream
  .format("json")
  .schema(schema)
  .load("input-json")

val input = in.transform { ds =>
  println("transform executed")  // <-- it's going to be executed once only
  ds
}

scala> input.isStreaming
res9: Boolean = true
----

It tracks already-processed files in `seenFiles` hash map.

[TIP]
====
Enable `DEBUG` or `TRACE` logging level for `org.apache.spark.sql.execution.streaming.FileStreamSource` to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.streaming.FileStreamSource=TRACE
```

Refer to spark-sql-streaming-logging.md[Logging].
====

=== [[creating-instance]] Creating FileStreamSource Instance

CAUTION: FIXME

=== [[options]] Options

==== [[maxFilesPerTrigger]] `maxFilesPerTrigger`

`maxFilesPerTrigger` option specifies the maximum number of files per trigger (batch). It limits the file stream source to read the `maxFilesPerTrigger` number of files specified at a time and hence enables rate limiting.

It allows for a static set of files be used like a stream for testing as the file set is processed `maxFilesPerTrigger` number of files at a time.

=== [[schema]] `schema`

If the schema is specified at instantiation time (using optional `dataSchema` constructor parameter) it is returned.

Otherwise, `fetchAllFiles` internal method is called to list all the files in a directory.

When there is at least one file the schema is calculated using `dataFrameBuilder` constructor parameter function. Else, an `IllegalArgumentException("No schema specified")` is thrown unless it is for *text* provider (as `providerName` constructor parameter) where the default schema with a single `value` column of type `StringType` is assumed.

NOTE: *text* as the value of `providerName` constructor parameter denotes *text file stream provider*.

=== [[getOffset]] `getOffset` Method

[source, scala]
----
getOffset: Option[Offset]
----

`getOffset`...FIXME

The maximum offset (`getOffset`) is calculated by fetching all the files in `path` excluding files that start with `_` (underscore).

When computing the maximum offset using `getOffset`, you should see the following DEBUG message in the logs:

```
Listed ${files.size} in ${(endTime.toDouble - startTime) / 1000000}ms
```

When computing the maximum offset using `getOffset`, it also filters out the files that were already seen (tracked in `seenFiles` internal registry).

You should see the following DEBUG message in the logs (depending on the status of a file):

```
new file: $file
// or
old file: $file
```

`getOffset` is part of the [Source](Source.md#getOffset) abstraction.

=== [[getBatch]] Generating DataFrame for Streaming Batch -- `getBatch` Method

`FileStreamSource.getBatch` asks <<metadataLog, metadataLog>> for the batch.

You should see the following INFO and DEBUG messages in the logs:

```
INFO Processing ${files.length} files from ${startId + 1}:$endId
DEBUG Streaming ${files.mkString(", ")}
```

The method to create a result batch is given at instantiation time (as `dataFrameBuilder` constructor parameter).

=== [[metadataLog]] `metadataLog`

`metadataLog` is a metadata storage using `metadataPath` path (which is a constructor parameter).

NOTE: It extends `HDFSMetadataLog[Seq[String]]`.

CAUTION: FIXME Review `HDFSMetadataLog`

=== [[fetchMaxOffset]] `fetchMaxOffset` Internal Method

[source, scala]
----
fetchMaxOffset(): FileStreamSourceOffset
----

`fetchMaxOffset`...FIXME

NOTE: `fetchMaxOffset` is used exclusively when `FileStreamSource` is requested to <<getOffset, getOffset>>.

=== [[fetchAllFiles]] `fetchAllFiles` Internal Method

[source, scala]
----
fetchAllFiles(): Seq[(String, Long)]
----

`fetchAllFiles`...FIXME

NOTE: `fetchAllFiles` is used exclusively when `FileStreamSource` is requested to <<fetchMaxOffset, fetchMaxOffset>>.

=== [[allFilesUsingMetadataLogFileIndex]] `allFilesUsingMetadataLogFileIndex` Internal Method

[source, scala]
----
allFilesUsingMetadataLogFileIndex(): Seq[FileStatus]
----

`allFilesUsingMetadataLogFileIndex` simply creates a new <<spark-sql-streaming-MetadataLogFileIndex.md#, MetadataLogFileIndex>> and requests it to `allFiles`.

NOTE: `allFilesUsingMetadataLogFileIndex` is used exclusively when `FileStreamSource` is requested to <<fetchAllFiles, fetchAllFiles>> (when requested for <<fetchMaxOffset, fetchMaxOffset>> when `FileStreamSource` is requested to <<getOffset, getOffset>>).
