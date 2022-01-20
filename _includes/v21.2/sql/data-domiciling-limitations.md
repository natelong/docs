Using CockroachDB as part of your approach to data domiciling has several limitations:

- Data domiciling does not work on data that is [indexed](indexes.html), since the replicas underlying the index are distributed across the nodes in the cluster. Therefore, we recommend keeping data that must be domiciled out of indexes.
- Some metadata about the objects in a schema may be stored in [system ranges](architecture/distribution-layer.html#meta-ranges), system tables, etc. CockroachDB synchronizes these system ranges and system tables across nodes. This synchronization does not respect any multi-region settings applied via either the [multi-region SQL statements](multiregion-overview.html), or the low-level [zone configs](configure-replication-zones.html) mechanism. This might result in potential "leakage" outside of the desired domicile if your schema includes primary keys, table names, etc., that may reveal information about their contents.
- [Zone configs](configure-replication-zones.html) can be used for data placement but these features were historically built for performance, not for domiciling. Further, the way CockroachDB performs replica placement is currently heuristic-based, so the zone config settings may not be obeyed at all times.
- Make sure your [log files](logging-overview.html) are stored according to the legal requirements of the jurisdictions you're operating in. For example, it may be easiest to store your log files for a multi-region cluster in the locality with the most restrictive subset of regulations.
- If you start a node with a [`--locality`](cockroach-start.html#locality) flag that says the node is in region _A_, but the node is actually running in some region _B_, this approach will not work. A CockroachDB node only knows its locality based on the text supplied to the `--locality` flag; it can not ensure that it is actually running in that physical location.