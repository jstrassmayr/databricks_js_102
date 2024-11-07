See [What is Delta Live Tables](https://docs.databricks.com/en/delta-live-tables/index.html).


# Pros and Cons of DLT
_Advantages_
- Automatic dependency resolution of tables/notebooks in pipeline orchestration. This reduces the complexity of pipeline creation and maintenance.
- Automatic cluster management (which cluster, how many workers)
- Incremental Data Processing: DLT supports incremental processing, which is resource-efficient as only new or changed data is processed (see Checkpoints). 
- Data Quality and Validation: With "Expectations," users can define validation rules to ensure data integrity. Invalid data is quarantined, enabling a more reliable data flow and reducing the likelihood of bad data impacting downstream processes.
- DLT performs maintenance tasks on tables being updated. Maintenance can improve query performance and reduce cost by removing old versions of tables. 

_Disadvantages_
- Execution of DLT code is not fully possible while developing. 
- The DLT-engine: It is hard to know what is going on under the DLT-hood e.g. to know when a full recompute is done or not.
- Vendor Lock-In: DLT is optimized for the Databricks ecosystem. Migrating to another platform could be complex.
- You cannot use Delta Sharing with a Delta Live Tables materialized view or streaming table published to Unity Catalog.
- The underlying files supporting materialized views might include data from upstream tables (including possible personally identifiable information) that do not appear in the materialized view definition. This data is automatically added to the underlying storage to support incremental refreshing of materialized views. Because the underlying files of a materialized view might risk exposing data from upstream tables not part of the materialized view schema, Databricks recommends not sharing the underlying storage with untrusted downstream consumers. For example, suppose a materialized view definition includes a COUNT(DISTINCT field_a) clause. Even though the materialized view definition only includes the aggregate COUNT DISTINCT clause, the underlying files will contain a list of the actual values of field_a.




# 3 Dataset types
- DLT Tables = Materialized Views
- DLT Streaming Tables
- DLT Views

# DLT Tables = Materialized Views
- The Delta Live Tables runtime automatically creates MVs in the Delta format and ensures they contain the latest result of the query.
- MVs results are refreshed incrementally avoiding the need to completely rebuild the view when new data arrives. An internal state is kept for this.
- MVs are powerful because they can handle any changes in the input. Each time the pipeline updates, query results are recalculated to reflect changes in upstream datasets.
- Note: If I modify data (using INSERT, UPDATE, …) of a normal DLT table, the modification is undone by the next pipeline-run and the table is rewritten.

## Consider using a materialized view when:
- Materialized views should be used for data processing tasks such as transformations (updates, deletions...), aggregations, Change-Data-Capture or pre-computing slow queries and frequently used computations e.g. in Silver- and Gold-Layer.
- Multiple downstream queries consume the table. Because views are computed on demand, the view is re-computed every time the view is queried.
- Other pipelines, jobs, or queries consume the table. Because views are not materialized, you can only use them in the same pipeline.
- You want to view the results of a query during development. Because tables are materialized and can be viewed and queried outside of the pipeline, using tables during development can help validate the correctness of computations. After validating, convert queries that do not require materialization into views.

## Disadvantages/Limitations
- Identity columns are not supported with tables/mat. views that are the target of APPLY CHANGES INTO and might be recomputed during updates. For this reason, Databricks recommends using identity columns in Delta Live Tables only with streaming tables. See [Use identity columns in Delta Lake](https://docs.databricks.com/en/delta/generated-columns.html#identity&language-python).
- now() and "system"-datetime columns
- Full recomputes will increase costs and runtime when working on big tables
- TBD: Late arriving dimensions/Early arriving facts are not a problem for MVs as the runtime notices the update(s) in the dimension and updates the result in the target table.
- TBD: When will work incrementally, when will it do a full recompute?
- TBD: What if I want to change the logic of my code? -> Recompute?


# DLT Streaming Tables
- Each input record is processed exactly once. DLT keeps track of what it already processed.
- Streaming tables are designed for data sources that are append-only.
- Note: If I modify data (using INSERT, UPDATE, …) of a streaming DLT, the modification is kept even after the next pipeline-run as only new data is added and the "current" data is untouched.

## Consider using a streaming table when:
- Ingesting data e.g. into the Bronze-Layer
- A query is defined against a data source that is continuously or incrementally growing.
- Query results should be computed incrementally.
- The pipeline needs high throughput and low latency.

## Disadvantages/Limitations
- Certain transformations that depend on the entire history of the dataset (e.g. aggregations) may not be efficiently handled in a streaming context. But: DLT tries to solve this by keeping intermediate data in the background. See [this link](https://docs.databricks.com/en/delta-live-tables/transform.html#calculate-aggregates-efficiently).
- Late arriving dimensions/Early arriving facts: With each pipeline update, new records from the stream are joined with the most current snapshot of the static dimension-table. If records are added or updated in the static dim-table after data from the streaming table has been processed, the resultant records are not recalculated unless a full refresh is performed. See [this example](https://docs.databricks.com/en/delta-live-tables/transform.html#stream-static-joins).


# DLT Views
- Records are processed every time the view is queried. Use views for intermediate transformations and data quality checks that should not be published to public datasets. But: They leverage caching algorithms to not always have to query the source.
- Are similar to a temporary view in SQL 
- Allow you to name and reuse a given computation/transformation of a source

## Consider using a view to do the following:
- Break a large or complex query that you want into easier-to-manage queries.
- Validate intermediate results using expectations.
- Reduce storage and compute costs for results you don’t need to persist. Because tables are materialized, they require additional computation and storage resources.

## Disadvantages/Limitations
- Are available from within a pipeline only and cannot be queried interactively after the pipeline


# Pipelines
A pipeline contains materialized views and streaming tables declared in Python or SQL source files. Delta Live Tables infers the dependencies between these tables, ensuring updates occur in the correct order. For each dataset, Delta Live Tables compares the current state with the desired state and proceeds to create or update datasets using efficient processing methods.
  





# Expectations
- Define requirements on row level to attributes of that row.
- Can define a behaviour for cases of errors e.g. expect_or_fail, expect_or_drop_row.


# Coding Practices
* https://www.databricks.com/blog/applying-software-development-devops-best-practices-delta-live-table-pipelines
* https://www.sicara.fr/blog-technique/databricks-delta-live-tables-software-engineering-best-practices
* Use Poetry for Python dependency management (this is mentioned in both links above): https://python-poetry.org/
* Use Nutter for Testing Python notebooks: https://github.com/microsoft/nutter


# Sources
- https://docs.databricks.com/en/delta-live-tables/index.html
- https://docs.databricks.com/en/delta-live-tables/unity-catalog.html
- https://docs.databricks.com/en/delta-live-tables/transform.html
- https://www.databricks.com/blog/introducing-materialized-views-and-streaming-tables-databricks-sql
- https://docs.databricks.com/en/tables/streaming.html
