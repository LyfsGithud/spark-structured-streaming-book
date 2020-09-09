== [[StateStore]] StateStore Contract -- Kay-Value Store for Streaming State Data

`StateStore` is the <<contract, abstraction>> of <<implementations, key-value stores>> for managing state in <<spark-sql-streaming-stateful-stream-processing.adoc#, Stateful Stream Processing>> (e.g. for persisting running aggregates in <<spark-sql-streaming-aggregation.adoc#, Streaming Aggregation>>).

`StateStore` supports *incremental checkpointing* in which only the key-value "Row" pairs that changed are <<commit, committed>> or <<abort, aborted>> (without touching other key-value pairs).

`StateStore` is identified with the <<id, aggregating operator id and the partition id>> (among other properties for identification).

[[implementations]]
NOTE: <<spark-sql-streaming-HDFSBackedStateStore.adoc#, HDFSBackedStateStore>> is the default and only known implementation of the <<contract, StateStore Contract>> in Spark Structured Streaming.

[[contract]]
.StateStore Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| abort
a| [[abort]]

[source, scala]
----
abort(): Unit
----

Aborts (_discards_) changes to the state store

Used when:

* `StateStoreOps` implicit class is requested to <<spark-sql-streaming-StateStoreOps.adoc#mapPartitionsWithStateStore, mapPartitionsWithStateStore>> (when the state store has not been <<hasCommitted, committed>> for a task that finishes, possibly with an error)

* `StateStoreHandler` (of <<spark-sql-streaming-SymmetricHashJoinStateManager.adoc#, SymmetricHashJoinStateManager>>) is requested to <<spark-sql-streaming-StateStoreHandler.adoc#abortIfNeeded, abortIfNeeded>> (when the state store has not been <<hasCommitted, committed>> for a task that finishes, possibly with an error)

| commit
a| [[commit]]

[source, scala]
----
commit(): Long
----

Commits the changes to the state store (and returns the current version)

Used when:

* <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#, FlatMapGroupsWithStateExec>>, <<spark-sql-streaming-StreamingDeduplicateExec.adoc#, StreamingDeduplicateExec>> and <<spark-sql-streaming-StreamingGlobalLimitExec.adoc#, StreamingGlobalLimitExec>> physical operators are executed (right after all rows in a partition have been processed)

* `StreamingAggregationStateManagerBaseImpl` is requested to <<spark-sql-streaming-StreamingAggregationStateManagerBaseImpl.adoc#commit, commit (changes to) a state store>> (exclusively when <<spark-sql-streaming-StateStoreSaveExec.adoc#, StateStoreSaveExec>> physical operator is executed)

* `StateStoreHandler` (of <<spark-sql-streaming-SymmetricHashJoinStateManager.adoc#, SymmetricHashJoinStateManager>>) is requested to <<spark-sql-streaming-StateStoreHandler.adoc#commit, commit changes to a state store>>

| get
a| [[get]]

[source, scala]
----
get(key: UnsafeRow): UnsafeRow
----

Looks up (_gets_) the value of the given non-`null` key

Used when:

* <<spark-sql-streaming-StreamingDeduplicateExec.adoc#, StreamingDeduplicateExec>> and <<spark-sql-streaming-StreamingGlobalLimitExec.adoc#, StreamingGlobalLimitExec>> physical operators are executed

* `StateManagerImplBase` (of `FlatMapGroupsWithStateExecHelper`) is requested to `getState`

* <<spark-sql-streaming-StreamingAggregationStateManagerImplV1.adoc#get, StreamingAggregationStateManagerImplV1>> and <<spark-sql-streaming-StreamingAggregationStateManagerImplV2.adoc#get, StreamingAggregationStateManagerImplV2>> are requested to get the value of a non-null key

* `KeyToNumValuesStore` is requested to <<spark-sql-streaming-KeyToNumValuesStore.adoc#get, get>>

* KeyWithIndexToValueStore` is requested to <<spark-sql-streaming-KeyWithIndexToValueStore.adoc#get, get>> and <<spark-sql-streaming-KeyWithIndexToValueStore.adoc#getAll, getAll>>

| getRange
a| [[getRange]]

[source, scala]
----
getRange(
  start: Option[UnsafeRow],
  end: Option[UnsafeRow]): Iterator[UnsafeRowPair]
----

Gets the key-value pairs of `UnsafeRows` for the specified range (with optional approximate `start` and `end` extents)

Used when:

* `WatermarkSupport` is requested to <<spark-sql-streaming-WatermarkSupport.adoc#removeKeysOlderThanWatermark, removeKeysOlderThanWatermark>>

* `StateManagerImplBase` is requested to `getAllState`

* `StreamingAggregationStateManagerBaseImpl` is requested to <<spark-sql-streaming-StreamingAggregationStateManagerBaseImpl.adoc#keys, keys>>

* <<spark-sql-streaming-KeyToNumValuesStore.adoc#iterator, KeyToNumValuesStore>> and <<spark-sql-streaming-KeyWithIndexToValueStore.adoc#iterator, KeyWithIndexToValueStore>> are requested to `iterator`

NOTE: All the uses above assume the `start` and `end` as `None` that basically is <<iterator, iterator>>.

| hasCommitted
a| [[hasCommitted]]

[source, scala]
----
hasCommitted: Boolean
----

Flag to indicate whether state changes have been committed (`true`) or not (`false`)

Used when:

* `RDD` (via `StateStoreOps` implicit class) is requested to <<spark-sql-streaming-StateStoreOps.adoc#mapPartitionsWithStateStore, mapPartitionsWithStateStore>> (and a task finishes and may need to <<abort, abort state updates>>)

* `SymmetricHashJoinStateManager` is requested to <<spark-sql-streaming-SymmetricHashJoinStateManager.adoc#abortIfNeeded, abortIfNeeded>> (when a task finishes and may need to <<abort, abort state updates>>))

| id
a| [[id]]

[source, scala]
----
id: StateStoreId
----

The <<spark-sql-streaming-StateStoreId.adoc#, ID>> of the state store

Used when:

* `HDFSBackedStateStore` state store is requested for the <<spark-sql-streaming-HDFSBackedStateStore.adoc#toString, textual representation>>

* `StateStoreHandler` (of <<spark-sql-streaming-SymmetricHashJoinStateManager.adoc#, SymmetricHashJoinStateManager>>) is requested to <<spark-sql-streaming-StateStoreHandler.adoc#abortIfNeeded, abortIfNeeded>> and <<spark-sql-streaming-StateStoreHandler.adoc#getStateStore, getStateStore>>

| iterator
a| [[iterator]]

[source, scala]
----
iterator(): Iterator[UnsafeRowPair]
----

Returns an iterator with all the kay-value pairs in the state store

Used when:

* <<spark-sql-streaming-StateStoreRestoreExec.adoc#, StateStoreRestoreExec>> physical operator is requested to execute

* <<spark-sql-streaming-HDFSBackedStateStore.adoc#getRange, HDFSBackedStateStore>> state store in particular and any <<getRange, StateStore>> in general are requested to `getRange`

* `StreamingAggregationStateManagerImplV1` state manager is requested for the <<spark-sql-streaming-StreamingAggregationStateManagerImplV1.adoc#iterator, iterator>> and <<spark-sql-streaming-StreamingAggregationStateManagerImplV1.adoc#values, values>>

* `StreamingAggregationStateManagerImplV2` state manager is requested to <<spark-sql-streaming-StreamingAggregationStateManagerImplV2.adoc#iterator, iterator>> and <<spark-sql-streaming-StreamingAggregationStateManagerImplV2.adoc#values, values>>

| metrics
a| [[metrics]]

[source, scala]
----
metrics: StateStoreMetrics
----

<<spark-sql-streaming-StateStoreMetrics.adoc#, StateStoreMetrics>> of the state store

Used when:

* `StateStoreWriter` stateful physical operator is requested to <<spark-sql-streaming-StateStoreWriter.adoc#setStoreMetrics, setStoreMetrics>>

* `StateStoreHandler` (of <<spark-sql-streaming-SymmetricHashJoinStateManager.adoc#, SymmetricHashJoinStateManager>>) is requested to <<spark-sql-streaming-StateStoreHandler.adoc#commit, commit>> and for the <<spark-sql-streaming-StateStoreHandler.adoc#metrics, metrics>>

| put
a| [[put]]

[source, scala]
----
put(
  key: UnsafeRow,
  value: UnsafeRow): Unit
----

Stores (_puts_) the value for the (non-null) key

Used when:

* <<spark-sql-streaming-StreamingDeduplicateExec.adoc#, StreamingDeduplicateExec>> and <<spark-sql-streaming-StreamingGlobalLimitExec.adoc#, StreamingGlobalLimitExec>> physical operators are executed

* `StateManagerImplBase` is requested to `putState`

* <<spark-sql-streaming-StreamingAggregationStateManagerImplV1.adoc#put, StreamingAggregationStateManagerImplV1>> and <<spark-sql-streaming-StreamingAggregationStateManagerImplV2.adoc#put, StreamingAggregationStateManagerImplV2>> are requested to store a row in a state store

* <<spark-sql-streaming-KeyToNumValuesStore.adoc#put, KeyToNumValuesStore>> and <<spark-sql-streaming-KeyWithIndexToValueStore.adoc#put, KeyWithIndexToValueStore>> are requested to store a new value for a given key

| remove
a| [[remove]]

[source, scala]
----
remove(key: UnsafeRow): Unit
----

Removes the (non-null) key from the state store

Used when:

* Physical operators with `WatermarkSupport` are requested to <<spark-sql-streaming-WatermarkSupport.adoc#removeKeysOlderThanWatermark, removeKeysOlderThanWatermark>>

* `StateManagerImplBase` is requested to `removeState`

* `StreamingAggregationStateManagerBaseImpl` is requested to <<spark-sql-streaming-StreamingAggregationStateManagerBaseImpl.adoc#remove, remove a key from a state store>>

* `KeyToNumValuesStore` is requested to <<spark-sql-streaming-KeyToNumValuesStore.adoc#remove, remove a key>>

* `KeyWithIndexToValueStore` is requested to <<spark-sql-streaming-KeyWithIndexToValueStore.adoc#remove, remove a key>> and <<spark-sql-streaming-KeyWithIndexToValueStore.adoc#removeAllValues, removeAllValues>>

| version
a| [[version]]

[source, scala]
----
version: Long
----

Version of the state store

Used exclusively when `HDFSBackedStateStore` state store is requested for a <<spark-sql-streaming-HDFSBackedStateStore.adoc#newVersion, new version>> (that simply the current version incremented)

|===

[NOTE]
====
`StateStore` was introduced in https://github.com/apache/spark/commit/8c826880f5eaa3221c4e9e7d3fece54e821a0b98[[SPARK-13809\][SQL\] State store for streaming aggregations].

Read the motivation and design in https://docs.google.com/document/d/1-ncawFx8JS5Zyfq1HAEGBx56RDet9wfVp_hDM8ZL254/edit[State Store for Streaming Aggregations].
====

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.streaming.state.StateStore$` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.streaming.state.StateStore$=ALL
```

Refer to <<spark-sql-streaming-logging.adoc#, Logging>>.
====

=== [[coordinatorRef]] Creating (and Caching) RPC Endpoint Reference to StateStoreCoordinator for Executors -- `coordinatorRef` Internal Object Method

[source, scala]
----
coordinatorRef: Option[StateStoreCoordinatorRef]
----

`coordinatorRef` requests the `SparkEnv` helper object for the current `SparkEnv`.

If the `SparkEnv` is available and the <<_coordRef, _coordRef>> is not assigned yet, `coordinatorRef` prints out the following DEBUG message to the logs followed by requesting the `StateStoreCoordinatorRef` for the <<spark-sql-streaming-StateStoreCoordinatorRef.adoc#forExecutor, StateStoreCoordinator endpoint>>.

```
Getting StateStoreCoordinatorRef
```

If the `SparkEnv` is available, `coordinatorRef` prints out the following INFO message to the logs:

```
Retrieved reference to StateStoreCoordinator: [_coordRef]
```

NOTE: `coordinatorRef` is used when `StateStore` helper object is requested to <<reportActiveStoreInstance, reportActiveStoreInstance>> (when `StateStore` object helper is requested to <<get-StateStore, find the StateStore by StateStoreProviderId>>) and <<verifyIfStoreInstanceActive, verifyIfStoreInstanceActive>> (when `StateStore` object helper is requested to <<doMaintenance, doMaintenance>>).

=== [[unload]] Unloading State Store Provider -- `unload` Method

[source, scala]
----
unload(storeProviderId: StateStoreProviderId): Unit
----

`unload`...FIXME

NOTE: `unload` is used when `StateStore` helper object is requested to <<stop, stop>> and <<doMaintenance, doMaintenance>>.

=== [[stop]] `stop` Object Method

[source, scala]
----
stop(): Unit
----

`stop`...FIXME

NOTE: `stop` seems only be used in tests.

=== [[reportActiveStoreInstance]] Announcing New StateStoreProvider -- `reportActiveStoreInstance` Internal Object Method

[source, scala]
----
reportActiveStoreInstance(
  storeProviderId: StateStoreProviderId): Unit
----

`reportActiveStoreInstance` takes the current host and `executorId` (from the `BlockManager` on the Spark executor) and requests the <<coordinatorRef, StateStoreCoordinatorRef>> to <<spark-sql-streaming-StateStoreCoordinatorRef.adoc#reportActiveInstance, reportActiveInstance>>.

NOTE: `reportActiveStoreInstance` uses `SparkEnv` to access the `BlockManager`.

In the end, `reportActiveStoreInstance` prints out the following INFO message to the logs:

```
Reported that the loaded instance [storeProviderId] is active
```

NOTE: `reportActiveStoreInstance` is used exclusively when `StateStore` utility is requested to <<get-StateStore, find the StateStore by StateStoreProviderId>>.

=== [[MaintenanceTask]] `MaintenanceTask` Daemon Thread

`MaintenanceTask` is a daemon thread that <<doMaintenance, triggers maintenance work of registered StateStoreProviders>>.

When an error occurs, `MaintenanceTask` clears <<loadedProviders, loadedProviders>> internal registry.

`MaintenanceTask` is scheduled on *state-store-maintenance-task* thread pool that runs periodically every <<spark-sql-streaming-properties.adoc#spark.sql.streaming.stateStore.maintenanceInterval, spark.sql.streaming.stateStore.maintenanceInterval>> (default: `60s`).

=== [[get-StateStore]] Looking Up StateStore by Provider ID -- `get` Object Method

[source, scala]
----
get(
  storeProviderId: StateStoreProviderId,
  keySchema: StructType,
  valueSchema: StructType,
  indexOrdinal: Option[Int],
  version: Long,
  storeConf: StateStoreConf,
  hadoopConf: Configuration): StateStore
----

`get` finds `StateStore` for the specified <<spark-sql-streaming-StateStoreProviderId.adoc#, StateStoreProviderId>> and version.

NOTE: The version is either the <<spark-sql-streaming-EpochTracker.adoc#getCurrentEpoch, current epoch>> (in <<spark-sql-streaming-continuous-stream-processing.adoc#, Continuous Stream Processing>>) or the <<spark-sql-streaming-StatefulOperatorStateInfo.adoc#storeVersion, current batch ID>> (in <<spark-sql-streaming-micro-batch-stream-processing.adoc#, Micro-Batch Stream Processing>>).

Internally, `get` looks up the <<spark-sql-streaming-StateStoreProvider.adoc#, StateStoreProvider>> (by `storeProviderId`) in the <<loadedProviders, loadedProviders>> internal cache. If unavailable, `get` uses the `StateStoreProvider` utility to <<spark-sql-streaming-StateStoreProvider.adoc#createAndInit, create and initialize one>>.

`get` will also <<startMaintenanceIfNeeded, start the periodic maintenance task>> (unless already started) and <<reportActiveStoreInstance, announce the new StateStoreProvider>>.

In the end, `get` requests the `StateStoreProvider` to <<spark-sql-streaming-StateStoreProvider.adoc#getStore, look up the StateStore by the specified version>>.

[NOTE]
====
`get` is used when:

* `StateStoreRDD` is requested to <<spark-sql-streaming-StateStoreRDD.adoc#compute, compute a partition>>

* `StateStoreHandler` (of <<spark-sql-streaming-SymmetricHashJoinStateManager.adoc#, SymmetricHashJoinStateManager>>) is requested to <<spark-sql-streaming-StateStoreHandler.adoc#getStateStore, look up a StateStore (by key and value schemas)>>
====

==== [[startMaintenanceIfNeeded]] Starting Periodic Maintenance Task (Unless Already Started) -- `startMaintenanceIfNeeded` Internal Object Method

[source, scala]
----
startMaintenanceIfNeeded(): Unit
----

`startMaintenanceIfNeeded` schedules <<MaintenanceTask, MaintenanceTask>> to start after and every link:spark-sql-streaming-properties.adoc#spark.sql.streaming.stateStore.maintenanceInterval[spark.sql.streaming.stateStore.maintenanceInterval] (defaults to `60s`).

NOTE: `startMaintenanceIfNeeded` does nothing when the maintenance task has already been started and is still running.

NOTE: `startMaintenanceIfNeeded` is used exclusively when `StateStore` is requested to <<get, find the StateStore by StateStoreProviderId>>.

==== [[doMaintenance]] Doing State Maintenance of Registered State Store Providers -- `doMaintenance` Internal Object Method

[source, scala]
----
doMaintenance(): Unit
----

Internally, `doMaintenance` prints the following DEBUG message to the logs:

```
Doing maintenance
```

`doMaintenance` then requests every link:spark-sql-streaming-StateStoreProvider.adoc[StateStoreProvider] (registered in <<loadedProviders, loadedProviders>>) to link:spark-sql-streaming-StateStoreProvider.adoc#doMaintenance[do its own internal maintenance] (only when a `StateStoreProvider` <<verifyIfStoreInstanceActive, is still active>>).

When a `StateStoreProvider` is <<verifyIfStoreInstanceActive, inactive>>, `doMaintenance` <<unload, removes it from the provider registry>> and prints the following INFO message to the logs:

```
Unloaded [provider]
```

NOTE: `doMaintenance` is used exclusively in <<MaintenanceTask, MaintenanceTask daemon thread>>.

==== [[verifyIfStoreInstanceActive]] `verifyIfStoreInstanceActive` Internal Object Method

[source, scala]
----
verifyIfStoreInstanceActive(storeProviderId: StateStoreProviderId): Boolean
----

`verifyIfStoreInstanceActive`...FIXME

NOTE: `verifyIfStoreInstanceActive` is used exclusively when `StateStore` helper object is requested to <<doMaintenance, doMaintenance>> (from a running <<MaintenanceTask, MaintenanceTask daemon thread>>).

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| loadedProviders
| [[loadedProviders]] *Loaded providers* internal cache, i.e. <<spark-sql-streaming-StateStoreProvider.adoc#, StateStoreProviders>> per <<spark-sql-streaming-StateStoreProviderId.adoc#, StateStoreProviderId>>

Used in...FIXME

| _coordRef
| [[_coordRef]] <<spark-sql-streaming-StateStoreCoordinatorRef.adoc#, StateStoreCoordinator RPC endpoint>> (a `RpcEndpointRef` to <<spark-sql-streaming-StateStoreCoordinator.adoc#, StateStoreCoordinator>>)

Used in...FIXME
|===
