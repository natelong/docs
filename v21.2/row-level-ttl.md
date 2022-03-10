---
title: Batch Delete Expired Data with Row-Level TTL
summary: Automatically delete rows of expired data older than a specified interval.
toc: true
keywords: ttl,delete,deletion,bulk deletion,bulk delete,expired data,expire data,time to live,row-level ttl,row level ttl
docs_area: develop
---

{% include {{page.version.version}}/sql/row-level-ttl.md %}

By using row-level TTL, you can avoid the complexity of writing and managing scheduled jobs from your application to perform the deletes. This can become complicated due to the need to balance the timeliness of the deletions vs. the effect on database performance when processing foreground traffic from your application.

## Use cases

Use cases for row-level TTL include:

- Delete inactive data events to manage data size and performance: For example, you may want to delete order records from an online store after 90 days.

- Delete data no longer needed for compliance: For example, a banking application may need to keep some subset of data for a period of time due to financial regulations. A TTL can be used to remove data older than that period on a continuous basis.

- Outbox pattern: When events are written to an outbox table and published to a system like Kafka, those events must be deleted to prevent unbounded growth in the size of the outbox table. The TTL can be useful for this.

## How it works

At a high level, Row-Level TTL works by doing approximately the same things you would do yourself if you had to implement this feature in your application.  For example, if you were implementing batch deletes of expired data yourself, you might do something like the following:

1. Issue a [selection query](selection-queries.html) at a [historical timestamp](as-of-system-time.html), yielding a set of rows that are eligible for deletion.
2. Rather than issue a naive [`DELETE`](delete.html) on all the rows that are eligible for deletion, which would likely have [bad performance implications](delete.html#preserving-delete-performance-over-time), write a batch-delete loop similar to that described by [Batch delete on an indexed column](bulk-delete-data.html#batch-delete-on-an-indexed-column). This batch-delete loop runs in the background and operates on a subset of the eligible rows per iteration to protect the performance of foreground application queries.
3. Depending on the performance impact of the previous step, you'd need to tweak the size of the batches per iteration, and limit the rate of those iterations, to find the best tradeoff you can between the timeliness of the deletions vs. the effect on overall database performance.

In other words, Row-Level TTL handles:

- Sharding the delete into multiple queries, and running those queries in parallel as background jobs
- Deciding how many rows to SELECT and DELETE at once
- Rate-limiting to control the rate of SELECTs and DELETEs

### Low-level details: table metadata and storage parameters

TTLs are defined on a per-table basis. The syntax for creating a table with an automatically managed TTL extends the `storage_parameter` syntax used by PostgreSQL. For example, the surface syntax

{% include_cached copy-clipboard.html %}
~~~ sql
CREATE TABLE events (
  id UUID PRIMARY KEY,
  description TEXT
) WITH (ttl_expire_after = '15 minutes');
~~~

has the following effects:

1. Creates a repeating scheduled job for the `events` table
2. Adds a hidden column `crdb_internal_expiration` of type [`TIMESTAMPTZ`](timestamp.html) to represent the TTL
3. Implicitly adds the `ttl` and `ttl_automatic_column` storage parameters.

This results in the above statement "compiling down" to SQL that looks like the following:

{% include_cached copy-clipboard.html %}
~~~ sql
CREATE TABLE tbl (
   id INT PRIMARY KEY,
   text TEXT,
   crdb_internal_expiration TIMESTAMPTZ
      NOT VISIBLE
      NOT NULL
      DEFAULT current_timestamp() + '5 minutes'
      ON UPDATE current_timestamp() + '5 minutes'
) WITH (ttl = 'on', ttl_automatic_column = 'on', ttl_expire_after = '5 minutes')
~~~

XXX

### The deletion job

XXX

## Examples

### Basic examples

#### Create a table with row-level TTL

To specify a TTL when creating a table, use the SQL syntax shown below.  For example, to create a new table with rows that expire after 15 minutes, execute a statement like the following:

{% include_cached copy-clipboard.html %}
~~~ sql
CREATE TABLE events (
  id UUID PRIMARY KEY,
  description TEXT
) WITH (ttl_expire_after = '15 minutes');
~~~

#### Add row-level TTL to an existing table

To add a TTL to an existing table, use the SQL syntax shown below.

{% include_cached copy-clipboard.html %}
~~~ sql
ALTER TABLE events SET (ttl_expire_after = '15 minutes');
~~~

#### Drop row-level TTL from a table

To drop the TTL on an existing table, 

### Advanced examples

XXX

## Limitations

XXX

## See also

- [Bulk-delete Data](bulk-delete-data.html)
- [SQL Performance Best Practices](performance-best-practices-overview.html)
- [Developer Guide Overview](developer-guide-overview.html)
