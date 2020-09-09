== [[StreamingAggregationStateManagerBaseImpl]] StreamingAggregationStateManagerBaseImpl -- Base State Manager for Streaming Aggregation

`StreamingAggregationStateManagerBaseImpl` is the base implementation of the <<spark-sql-streaming-StreamingAggregationStateManager.adoc#, StreamingAggregationStateManager contract>> for <<implementations, state managers for streaming aggregations>> that <<keyProjector, use UnsafeProjection to getKey>>.

[[keyProjector]]
`StreamingAggregationStateManagerBaseImpl` uses `UnsafeProjection` to <<getKey, getKey>>.

[[implementations]]
.StreamingAggregationStateManagerBaseImpls
[cols="1,2",options="header",width="100%"]
|===
| StreamingAggregationStateManagerBaseImpl
| Description

| <<spark-sql-streaming-StreamingAggregationStateManagerImplV1.adoc#, StreamingAggregationStateManagerImplV1>>
| [[StreamingAggregationStateManagerImplV1]] Legacy <<spark-sql-streaming-StreamingAggregationStateManager.adoc#, StreamingAggregationStateManager>> (used when <<spark-sql-streaming-properties.adoc#spark.sql.streaming.aggregation.stateFormatVersion, spark.sql.streaming.aggregation.stateFormatVersion>> configuration property is `1`)

| <<spark-sql-streaming-StreamingAggregationStateManagerImplV2.adoc#, StreamingAggregationStateManagerImplV2>>
| [[StreamingAggregationStateManagerImplV2]] Default <<spark-sql-streaming-StreamingAggregationStateManager.adoc#, StreamingAggregationStateManager>> (used when <<spark-sql-streaming-properties.adoc#spark.sql.streaming.aggregation.stateFormatVersion, spark.sql.streaming.aggregation.stateFormatVersion>> configuration property is `2`)
|===

[[creating-instance]]
`StreamingAggregationStateManagerBaseImpl` takes the following to be created:

* [[keyExpressions]] Catalyst expressions for the keys (`Seq[Attribute]`)
* [[inputRowAttributes]] Catalyst expressions for the input rows (`Seq[Attribute]`)

NOTE: `StreamingAggregationStateManagerBaseImpl` is a Scala abstract class and cannot be <<creating-instance, created>> directly. It is created indirectly for the <<implementations, concrete StreamingAggregationStateManagerBaseImpls>>.

=== [[commit]] Committing (Changes to) State Store -- `commit` Method

[source, scala]
----
commit(
  store: StateStore): Long
----

NOTE: `commit` is part of the <<spark-sql-streaming-StreamingAggregationStateManager.adoc#commit, StreamingAggregationStateManager Contract>> to commit changes to a <<spark-sql-streaming-StateStore.adoc#, state store>>.

`commit` simply requests the <<spark-sql-streaming-StateStore.adoc#, state store>> to <<spark-sql-streaming-StateStore.adoc#commit, commit state changes>>.

=== [[remove]] Removing Key From State Store -- `remove` Method

[source, scala]
----
remove(store: StateStore, key: UnsafeRow): Unit
----

NOTE: `remove` is part of the <<spark-sql-streaming-StreamingAggregationStateManager.adoc#remove, StreamingAggregationStateManager Contract>> to remove a key from a state store.

`remove`...FIXME

=== [[getKey]] `getKey` Method

[source, scala]
----
getKey(row: UnsafeRow): UnsafeRow
----

NOTE: `getKey` is part of the <<spark-sql-streaming-StreamingAggregationStateManager.adoc#getKey, StreamingAggregationStateManager Contract>> to...FIXME

`getKey`...FIXME

=== [[keys]] Getting All Keys in State Store -- `keys` Method

[source, scala]
----
keys(store: StateStore): Iterator[UnsafeRow]
----

NOTE: `keys` is part of the <<spark-sql-streaming-StreamingAggregationStateManager.adoc#keys, StreamingAggregationStateManager Contract>> to get all keys in a state store (as an iterator).

`keys`...FIXME
