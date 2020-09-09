== [[JoinStateWatermarkPredicate]] JoinStateWatermarkPredicate Contract (Sealed Trait)

`JoinStateWatermarkPredicate` is the <<contract, abstraction>> of <<implementations, join state watermark predicates>> that are described by a <<expr, Catalyst expression>> and <<desc, desc>>.

`JoinStateWatermarkPredicate` is created using <<spark-sql-streaming-StreamingSymmetricHashJoinHelper.adoc#getOneSideStateWatermarkPredicate, StreamingSymmetricHashJoinHelper>> utility (for <<spark-sql-streaming-IncrementalExecution.adoc#state, planning a StreamingSymmetricHashJoinExec physical operator for execution with execution-specific configuration>>)

`JoinStateWatermarkPredicate` is used to create a <<spark-sql-streaming-OneSideHashJoiner.adoc#, OneSideHashJoiner>> (and <<spark-sql-streaming-JoinStateWatermarkPredicates.adoc#, JoinStateWatermarkPredicates>>).

[[contract]]
.JoinStateWatermarkPredicate Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| desc
a| [[desc]]

[source, scala]
----
desc: String
----

Used exclusively for the <<toString, textual representation>>

| expr
a| [[expr]]

[source, scala]
----
expr: Expression
----

A Catalyst `Expression`

Used for the <<toString, textual representation>> and a <<spark-sql-streaming-StreamingSymmetricHashJoinHelper.adoc#getStateWatermarkPredicates, JoinStateWatermarkPredicates>> (for <<spark-sql-streaming-StreamingSymmetricHashJoinExec.adoc#, StreamingSymmetricHashJoinExec>> physical operator)

|===

[[implementations]]
.JoinStateWatermarkPredicates
[cols="30,70",options="header",width="100%"]
|===
| JoinStateWatermarkPredicate
| Description

| JoinStateKeyWatermarkPredicate
a| [[JoinStateKeyWatermarkPredicate]] Watermark predicate on state keys (i.e. when the <<spark-sql-streaming-watermark.adoc#, streaming watermark>> is defined either on the <<spark-sql-streaming-StreamingSymmetricHashJoinExec.adoc#leftKeys, left>> or <<spark-sql-streaming-StreamingSymmetricHashJoinExec.adoc#rightKeys, right>> join keys)

Created when `StreamingSymmetricHashJoinHelper` utility is requested for a <<spark-sql-streaming-StreamingSymmetricHashJoinHelper.adoc#getStateWatermarkPredicates, JoinStateWatermarkPredicates>> for the left and right side of a stream-stream join (when `IncrementalExecution` is requested to optimize a query plan with a <<spark-sql-streaming-StreamingSymmetricHashJoinExec.adoc#, StreamingSymmetricHashJoinExec>> physical operator)

Used when `OneSideHashJoiner` is requested for the <<spark-sql-streaming-OneSideHashJoiner.adoc#stateKeyWatermarkPredicateFunc, stateKeyWatermarkPredicateFunc>> and then to <<spark-sql-streaming-OneSideHashJoiner.adoc#removeOldState, remove an old state>>

| JoinStateValueWatermarkPredicate
| [[JoinStateValueWatermarkPredicate]] Watermark predicate on state values

|===

NOTE: `JoinStateWatermarkPredicate` is a Scala *sealed trait* which means that all the <<implementations, implementations>> are in the same compilation unit (a single file).

=== [[toString]] Textual Representation -- `toString` Method

[source, scala]
----
toString: String
----

NOTE: `toString` is part of the link:++https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#toString--++[java.lang.Object] contract for the string representation of the object.

`toString` uses the <<desc, desc>> and <<expr, expr>> for the string representation:

```
[desc]: [expr]
```
