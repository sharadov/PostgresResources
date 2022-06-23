# Postgres Resources


How to read an Explain Plan?

https://thoughtbot.com/blog/reading-an-explain-analyze-query-plan

https://www.cybertec-postgresql.com/en/how-to-interpret-postgresql-explain-analyze-output/

Different joins in an Explain Plan

https://stackoverflow.com/questions/49023821/nested-join-vs-merge-join-vs-hash-join-in-postgresql

Checkpoints occuring too frequently

https://dba.stackexchange.com/questions/117479/checkpoints-are-occurring-too-frequently-during-pg-restore

Troubleshoot Postgres Performance Deep Dive

https://www.justwatch.com/blog/post/debugging-postgresql-performance-the-hard-way/

What is MVCC?

MVCC, which stands for multiversion concurrency control, is one of the main techniques Postgres uses to implement transactions. MVCC lets Postgres run many queries that touch the same rows simultaneously, while keeping those queries isolated from each other.

Postgres handles transaction isolation by using MVCC to create a concept called “snapshots”. Whenever a query starts, it takes a snapshot of the current state of the database. Only the effects of transactions that committed before the snapshot was created are visible to the query. Rows inserted by transactions that didn’t commit before the query started are invisible to the query, and rows deleted by a transaction that didn’t commit before the query started are still visible.

MVCC works by assigning every transaction a serially incremented transaction id (commonly abbreviated txid), with earlier transactions having smaller txids. Whenever a query starts, it records the next txid to be issued, and the txid of every transaction currently running. Collectively this information allows Postgres to determine if a transaction had completed before the query started. A transaction occurred before the query started if the txid of the transaction is both smaller than the next txid to be issued when the query started and is not in the list of txids of transactions that were running when the query started.

As part of MVCC, every row records both the txid of the transaction that inserted it, and if the row was deleted, the txid of the transaction that deleted it. With this information and the ability to determine whether a transaction completed before a query started, Postgres has enough information to implement snapshots. A row should be included in the snapshot of the query if that row was inserted by a transaction that completed before the query started and not deleted by a transaction that completed before the query.

Under MVCC, an insert is handled by inserting a new row with the txid of the current transaction as the insert id, deletes are handled by setting the delete txid of the row being deleted to the txid of the current transaction, and an update is just a delete followed by an insert of the new row1. Since a row isn’t physically deleted when a query deletes it, the physical disk space will need to freed sometime later. This is typically handled by a background job called the vacuum.

MVCC is ultimately great for two reasons. First, it offers a simple model for reasoning about the behavior of queries. All queries create a snapshot of the database when they start and only see the data in that snapshot. Second it offers decent performance. With MVCC, there is almost no need for locks. MVCC makes it impossible for read queries to block on write queries or for write queries to block on read queries.

RDS Proxy

https://aws.amazon.com/rds/proxy/

pg-pool2 vs pgbouncer

https://scalegrid.io/blog/postgresql-connection-pooling-part-2-pgbouncer/

Elastic search 

https://www.alibabacloud.com/blog/combining-elasticsearch-with-dbs-application-system-scenarios_597112


Postgres Parameters

https://postgresqlco.nf/doc/en/param/shared_buffers/

