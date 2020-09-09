== [[InputProcessor]] InputProcessor Helper Class of FlatMapGroupsWithStateExec Physical Operator

`InputProcessor` is a helper class to manage state in the <<store, state store>> for every partition of a <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#, FlatMapGroupsWithStateExec>> physical operator.

`InputProcessor` is <<creating-instance, created>> exclusively when `FlatMapGroupsWithStateExec` physical operator is requested to <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#doExecute, execute and generate a recipe for a distributed computation (as an RDD[InternalRow])>> (and uses `InputProcessor` in the `storeUpdateFunction` while processing rows per partition with a corresponding per-partition state store).

[[creating-instance]][[store]]
`InputProcessor` takes a single <<spark-sql-streaming-StateStore.adoc#, StateStore>> to be created. The `StateStore` manages the per-group state (and is used when processing <<processNewData, new data>> and <<processTimedOutState, timed-out state data>>, and in the <<onIteratorCompletion, "all rows processed" callback>>).

=== [[processNewData]] Processing New Data (Creating Iterator of New Data Processed) -- `processNewData` Method

[source, scala]
----
processNewData(dataIter: Iterator[InternalRow]): Iterator[InternalRow]
----

`processNewData` creates a grouped iterator of (of pairs of) per-group state keys and the row values from the given data iterator (`dataIter`) with the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#groupingAttributes, grouping attributes>> and the output schema of the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#child, child operator>> (of the parent `FlatMapGroupsWithStateExec` physical operator).

For every per-group state key (in the grouped iterator), `processNewData` requests the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#stateManager, StateManager>> (of the parent `FlatMapGroupsWithStateExec` physical operator) to <<spark-sql-streaming-StateManager.adoc#getState, get the state>> (from the <<spark-sql-streaming-StateStore.adoc#, StateStore>>) and <<callFunctionAndUpdateState, callFunctionAndUpdateState>> (with the `hasTimedOut` flag off, i.e. `false`).

NOTE: `processNewData` is used exclusively when `FlatMapGroupsWithStateExec` physical operator is requested to <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#doExecute, execute and generate a recipe for a distributed computation (as an RDD[InternalRow])>>.

=== [[processTimedOutState]] Processing Timed-Out State Data (Creating Iterator of Timed-Out State Data) -- `processTimedOutState` Method

[source, scala]
----
processTimedOutState(): Iterator[InternalRow]
----

`processTimedOutState` does nothing and simply returns an empty iterator for <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#isTimeoutEnabled, GroupStateTimeout.NoTimeout>>.

With <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#isTimeoutEnabled, timeout enabled>>, `processTimedOutState` gets the current timeout threshold per <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#timeoutConf, GroupStateTimeout>>:

* <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#batchTimestampMs, batchTimestampMs>> for <<spark-sql-streaming-GroupStateTimeout.adoc#ProcessingTimeTimeout, ProcessingTimeTimeout>>

* <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#eventTimeWatermark, eventTimeWatermark>> for <<spark-sql-streaming-GroupStateTimeout.adoc#EventTimeTimeout, EventTimeTimeout>>

`processTimedOutState` creates an iterator of timed-out state data by requesting the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#stateManager, StateManager>> for <<spark-sql-streaming-StateManager.adoc#getAllState, all the available state data>> (in the <<store, StateStore>>) and takes only the state data with timeout defined and below the current timeout threshold.

In the end, for every timed-out state data, `processTimedOutState` <<callFunctionAndUpdateState, callFunctionAndUpdateState>> (with the `hasTimedOut` flag enabled).

NOTE: `processTimedOutState` is used exclusively when `FlatMapGroupsWithStateExec` physical operator is requested to <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#doExecute, execute and generate a recipe for a distributed computation (as an RDD[InternalRow])>>.

=== [[callFunctionAndUpdateState]] `callFunctionAndUpdateState` Internal Method

[source, scala]
----
callFunctionAndUpdateState(
  stateData: StateData,
  valueRowIter: Iterator[InternalRow],
  hasTimedOut: Boolean): Iterator[InternalRow]
----

[NOTE]
====
`callFunctionAndUpdateState` is used when `InputProcessor` is requested to process <<processNewData, new data>> and <<processTimedOutState, timed-out state data>>.

When <<processNewData, processing new data>>, `hasTimedOut` flag is off (`false`).

When <<processTimedOutState, processing timed-out state data>>, `hasTimedOut` flag is on (`true`).
====

`callFunctionAndUpdateState` creates a key object by requesting the given `StateData` for the `UnsafeRow` of the key (_keyRow_) and converts it to an object (using the internal <<getKeyObj, state key converter>>).

`callFunctionAndUpdateState` creates value objects by taking every value row (from the given `valueRowIter` iterator) and converts them to objects (using the internal <<getValueObj, state value converter>>).

`callFunctionAndUpdateState` <<spark-sql-streaming-GroupStateImpl.adoc#createForStreaming, creates a new GroupStateImpl>> with the following:

* The current state value (of the given `StateData`) that could possibly be `null`

* The <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#batchTimestampMs, batchTimestampMs>> of the parent `FlatMapGroupsWithStateExec` operator (that could possibly be <<spark-sql-streaming-GroupStateImpl.adoc#NO_TIMESTAMP, -1>>)

* The <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#eventTimeWatermark, event-time watermark>> of the parent `FlatMapGroupsWithStateExec` operator (that could possibly be <<spark-sql-streaming-GroupStateImpl.adoc#NO_TIMESTAMP, -1>>)

* The <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#timeoutConf, GroupStateTimeout>> of the parent `FlatMapGroupsWithStateExec` operator

* The <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#watermarkPresent, watermarkPresent>> flag of the parent `FlatMapGroupsWithStateExec` operator

* The given `hasTimedOut` flag

`callFunctionAndUpdateState` then executes the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#func, user-defined state function>> (of the parent `FlatMapGroupsWithStateExec` operator) on the key object, value objects, and the newly-created `GroupStateImpl`.

For every output value from the user-defined state function, `callFunctionAndUpdateState` updates <<numOutputRows, numOutputRows>> performance metric and wraps the values to an internal row (using the internal <<getOutputRow, output value converter>>).

In the end, `callFunctionAndUpdateState` returns a `Iterator[InternalRow]` which calls the <<onIteratorCompletion, completion function>> right after rows have been processed (so the iterator is considered fully consumed).

==== [[onIteratorCompletion]] "All Rows Processed" Callback -- `onIteratorCompletion` Internal Method

[source, scala]
----
onIteratorCompletion: Unit
----

`onIteratorCompletion` branches off per whether the `GroupStateImpl` has been marked <<spark-sql-streaming-GroupStateImpl.adoc#hasRemoved, removed>> and no <<spark-sql-streaming-GroupStateImpl.adoc#getTimeoutTimestamp, timeout timestamp>> is specified or not.

When the `GroupStateImpl` has been marked <<spark-sql-streaming-GroupStateImpl.adoc#hasRemoved, removed>> and no <<spark-sql-streaming-GroupStateImpl.adoc#getTimeoutTimestamp, timeout timestamp>> is specified, `onIteratorCompletion` does the following:

. Requests the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#stateManager, StateManager>> (of the parent `FlatMapGroupsWithStateExec` operator) to <<spark-sql-streaming-StateManager.adoc#removeState, remove the state>> (from the <<store, StateStore>> for the key row of the given `StateData`)

. Increments the <<numUpdatedStateRows, numUpdatedStateRows>> performance metric

Otherwise, when the `GroupStateImpl` has not been marked <<spark-sql-streaming-GroupStateImpl.adoc#hasRemoved, removed>> or the <<spark-sql-streaming-GroupStateImpl.adoc#getTimeoutTimestamp, timeout timestamp>> is specified, `onIteratorCompletion` checks whether the timeout timestamp has changed by comparing the timeout timestamps of the <<spark-sql-streaming-GroupStateImpl.adoc#getTimeoutTimestamp, GroupStateImpl>> and the given `StateData`.

(only when the `GroupStateImpl` has been <<spark-sql-streaming-GroupStateImpl.adoc#hasUpdated, updated>>, <<spark-sql-streaming-GroupStateImpl.adoc#hasRemoved, removed>> or the timeout timestamp changed) `onIteratorCompletion` does the following:

. Requests the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#stateManager, StateManager>> (of the parent `FlatMapGroupsWithStateExec` operator) to <<spark-sql-streaming-StateManager.adoc#putState, persist the state>> (in the <<store, StateStore>> with the key row, updated state object, and the timeout timestamp of the given `StateData`)

. Increments the <<numUpdatedStateRows, numUpdatedStateRows>> performance metrics

NOTE: `onIteratorCompletion` is used exclusively when `InputProcessor` is requested to <<callFunctionAndUpdateState, callFunctionAndUpdateState>> (right after rows have been processed)

=== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| getKeyObj
a| [[getKeyObj]] A *state key converter* (of type `InternalRow => Any`) to deserialize a given row (for a per-group state key) to the current state value

* The deserialization expression for keys is specified as the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#keyDeserializer, key deserializer expression>> when the parent `FlatMapGroupsWithStateExec` operator is created

* The data type of state keys is specified as the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#groupingAttributes, grouping attributes>> when the parent `FlatMapGroupsWithStateExec` operator is created

Used exclusively when `InputProcessor` is requested to <<callFunctionAndUpdateState, callFunctionAndUpdateState>>.

| getOutputRow
a| [[getOutputRow]] A *output value converter* (of type `Any => InternalRow`) to wrap a given output value (from the user-defined state function) to a row

* The data type of the row is specified as the data type of the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#outputObjAttr, output object attribute>> when the parent `FlatMapGroupsWithStateExec` operator is created

Used exclusively when `InputProcessor` is requested to <<callFunctionAndUpdateState, callFunctionAndUpdateState>>.

| getValueObj
a| [[getValueObj]] A *state value converter* (of type `InternalRow => Any`) to deserialize a given row (for a per-group state value) to a Scala value

* The deserialization expression for values is specified as the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#valueDeserializer, value deserializer expression>> when the parent `FlatMapGroupsWithStateExec` operator is created

* The data type of state values is specified as the <<spark-sql-streaming-FlatMapGroupsWithStateExec.adoc#dataAttributes, data attributes>> when the parent `FlatMapGroupsWithStateExec` operator is created

Used exclusively when `InputProcessor` is requested to <<callFunctionAndUpdateState, callFunctionAndUpdateState>>.

| numOutputRows
a| [[numOutputRows]] `numOutputRows` performance metric

|===
