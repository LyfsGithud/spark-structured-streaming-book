== [[StreamingSymmetricHashJoinHelper]] StreamingSymmetricHashJoinHelper Utility

`StreamingSymmetricHashJoinHelper` is a Scala object with the following utility methods:

* <<getStateWatermarkPredicates, getStateWatermarkPredicates>>

=== [[getStateWatermarkPredicates]] Creating JoinStateWatermarkPredicates -- `getStateWatermarkPredicates` Object Method

[source, scala]
----
getStateWatermarkPredicates(
  leftAttributes: Seq[Attribute],
  rightAttributes: Seq[Attribute],
  leftKeys: Seq[Expression],
  rightKeys: Seq[Expression],
  condition: Option[Expression],
  eventTimeWatermark: Option[Long]): JoinStateWatermarkPredicates
----

[[getStateWatermarkPredicates-joinKeyOrdinalForWatermark]]
`getStateWatermarkPredicates` tries to find the index of the <<spark-sql-streaming-EventTimeWatermark.adoc#delayKey, watermark attribute>> among the left keys first, and if not found, the right keys.

NOTE: The <<spark-sql-streaming-EventTimeWatermark.adoc#delayKey, watermark attribute>> is defined using <<spark-sql-streaming-Dataset-operators.adoc#withWatermark, Dataset.withWatermark>> operator.

`getStateWatermarkPredicates` <<getOneSideStateWatermarkPredicate, determines the state watermark predicate>> for the left side of a join (for the given `leftAttributes`, the `leftKeys` and the `rightAttributes`).

`getStateWatermarkPredicates` <<getOneSideStateWatermarkPredicate, determines the state watermark predicate>> for the right side of a join (for the given `rightAttributes`, the `rightKeys` and the `leftAttributes`).

In the end, `getStateWatermarkPredicates` creates a <<spark-sql-streaming-JoinStateWatermarkPredicates.adoc#, JoinStateWatermarkPredicates>> with the left- and right-side state watermark predicates.

NOTE: `getStateWatermarkPredicates` is used exclusively when `IncrementalExecution` is requested to <<spark-sql-streaming-IncrementalExecution.adoc#state, apply the state preparation rule for batch-specific configuration>> (while optimizing query plans with <<spark-sql-streaming-StreamingSymmetricHashJoinExec.adoc#, StreamingSymmetricHashJoinExec>> physical operators).

==== [[getOneSideStateWatermarkPredicate]] Join State Watermark Predicate (for One Side of Join) -- `getOneSideStateWatermarkPredicate` Internal Method

[source, scala]
----
getOneSideStateWatermarkPredicate(
  oneSideInputAttributes: Seq[Attribute],
  oneSideJoinKeys: Seq[Expression],
  otherSideInputAttributes: Seq[Attribute]): Option[JoinStateWatermarkPredicate]
----

`getOneSideStateWatermarkPredicate` finds what attributes were used to define the <<spark-sql-streaming-EventTimeWatermark.adoc#delayKey, watermark attribute>> (the `oneSideInputAttributes` attributes, the <<getStateWatermarkPredicates-joinKeyOrdinalForWatermark, left or right join keys>>) and creates a <<spark-sql-streaming-JoinStateWatermarkPredicate.adoc#, JoinStateWatermarkPredicate>> as follows:

* <<spark-sql-streaming-JoinStateWatermarkPredicate.adoc#JoinStateKeyWatermarkPredicate, JoinStateKeyWatermarkPredicate>> if the watermark was defined on a join key (with the watermark expression for the index of the join key expression)

* <<spark-sql-streaming-JoinStateWatermarkPredicate.adoc#JoinStateValueWatermarkPredicate, JoinStateValueWatermarkPredicate>> if the watermark was defined among the `oneSideInputAttributes` (with the <<spark-sql-streaming-StreamingJoinHelper.adoc#getStateValueWatermark, state value watermark>> based on the given `oneSideInputAttributes` and `otherSideInputAttributes`)

NOTE: `getOneSideStateWatermarkPredicate` creates no <<spark-sql-streaming-JoinStateWatermarkPredicate.adoc#, JoinStateWatermarkPredicate>> (`None`) for no watermark found.

NOTE: `getStateWatermarkPredicates` is used exclusively to <<getStateWatermarkPredicates, create a JoinStateWatermarkPredicates>>.
