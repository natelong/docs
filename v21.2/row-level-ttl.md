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

SELECT * AS OF SYSTEM TIME xxx

DELETE FROM [^^^] where ttl ...

To specify a TTL when creating a table, use the SQL syntax shown below.  A background deletion job is then scheduled to delete any rows that have expired. For example, to create a new table with rows that expire after 15 minutes, execute a statement like the following:

## Examples

### Basic examples

#### Create a table with row-level TTL

To specify a TTL when creating a table, use the SQL syntax shown below.  A background deletion job is then scheduled to delete any rows that have expired. For example, to create a new table with rows that expire after 15 minutes, execute a statement like the following:

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

### Advanced examples

XXX

## Limitations

XXX

## See also

- [Bulk-delete Data](bulk-delete-data.html)
- [SQL Performance Best Practices](performance-best-practices-overview.html)
- [Developer Guide Overview](developer-guide-overview.html)
