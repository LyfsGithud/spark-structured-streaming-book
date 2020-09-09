== [[StateStoreCoordinator]] StateStoreCoordinator RPC Endpoint -- Tracking Locations of StateStores for StateStoreRDD

`StateStoreCoordinator` keeps track of <<spark-sql-streaming-StateStore.adoc#, state stores>> on Spark executors (per host and executor ID).

`StateStoreCoordinator` is used by `StateStoreRDD` when requested to link:spark-sql-streaming-StateStoreRDD.adoc#getPreferredLocations[get the location preferences of partitions] (based on the location of the stores).

`StateStoreCoordinator` is a `ThreadSafeRpcEndpoint` RPC endpoint that manipulates <<instances, instances>> registry through <<messages, RPC messages>>.

[[messages]]
.StateStoreCoordinator RPC Endpoint's Messages and Message Handlers
[cols="30m,70",options="header",width="100%"]
|===
| Message
| Message Handler

| DeactivateInstances
a| [[DeactivateInstances]] Removes <<spark-sql-streaming-StateStoreProviderId.adoc#, StateStoreProviderIds>> of a streaming query (given `runId`)

---

Internally, `StateStoreCoordinator` finds the `StateStoreProviderIds` of the streaming query per `queryRunId` and the given `runId` and removes them from the <<instances, instances>> internal registry.

`StateStoreCoordinator` prints out the following DEBUG message to the logs:

```
Deactivating instances related to checkpoint location [runId]: [storeIdsToRemove]
```

| GetLocation
a| [[GetLocation]] Gives the location of <<spark-sql-streaming-StateStoreProviderId.adoc#, StateStoreProviderId>> (from <<instances, instances>>) with the host and an executor id on that host.

You should see the following DEBUG message in the logs:

```
Got location of the state store [id]: [executorId]
```

| ReportActiveInstance
a| [[ReportActiveInstance]] One-way asynchronous (fire-and-forget) message to register a new <<spark-sql-streaming-StateStoreProviderId.adoc#, StateStoreProviderId>> on an executor (given `host` and `executorId`).

Sent out exclusively when <<spark-sql-streaming-StateStoreCoordinatorRef.adoc#, StateStoreCoordinatorRef>> RPC endpoint reference is requested to <<spark-sql-streaming-StateStoreCoordinatorRef.adoc#reportActiveInstance, reportActiveInstance>> (when `StateStore` utility is requested to <<spark-sql-streaming-StateStore.adoc#get-StateStore, look up the StateStore by provider ID>> when the `StateStore` and a corresponding `StateStoreProvider` were just created and initialized).

---

Internally, `StateStoreCoordinator` prints out the following DEBUG message to the logs:

```
Reported state store [id] is active at [executorId]
```

In the end, `StateStoreCoordinator` adds the `StateStoreProviderId` to the <<instances, instances>> internal registry.

| StopCoordinator
a| [[StopCoordinator]] Stops `StateStoreCoordinator` RPC Endpoint

You should see the following DEBUG message in the logs:

```
StateStoreCoordinator stopped
```

| VerifyIfInstanceActive
a| [[VerifyIfInstanceActive]] Verifies if a given <<spark-sql-streaming-StateStoreProviderId.adoc#, StateStoreProviderId>> is registered (in <<instances, instances>>) on `executorId`

You should see the following DEBUG message in the logs:

```
Verified that state store [id] is active: [response]
```
|===

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.execution.streaming.state.StateStoreCoordinator` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.streaming.state.StateStoreCoordinator=ALL
```

Refer to <<spark-sql-streaming-logging.adoc#, Logging>>.
====

=== [[instances]] `instances` Internal Registry

[source,scala]
----
instances: HashMap[StateStoreProviderId, ExecutorCacheTaskLocation]
----

`instances` is an internal registry of <<spark-sql-streaming-StateStoreProvider.adoc#, StateStoreProviders>> by their <<spark-sql-streaming-StateStoreProviderId.adoc#, StateStoreProviderIds>> and `ExecutorCacheTaskLocations` (with a `host` and a `executorId`).

* A new `StateStoreProviderId` added when `StateStoreCoordinator` is requested to <<ReportActiveInstance, handle a ReportActiveInstance message>>

* All `StateStoreProviderIds` of a streaming query are removed  when `StateStoreCoordinator` is requested to <<DeactivateInstances, handle a DeactivateInstances message>>