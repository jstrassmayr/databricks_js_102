
# 3 Data set types

## DLT Tables = Materialized Views
- Records are processed as required to return accurate results for the current data state. Conclusion: DLT keeps a "data state" internally.
- Materialized views should be used for data processing tasks such as transformations (updates, deletions...), aggregations, Change-Data-Capture or pre-computing slow queries and frequently used computations.
- The Delta Live Tables runtime automatically creates tables in the Delta format and ensures those tables are updated with the latest result of the query that creates the table.
- Mat views are powerful because they can handle any changes in the input. Each time the pipeline updates, query results are recalculated to reflect changes in upstream datasets.
- Note: If I modify data (using INSERT, UPDATE, …) of a normal DLT table, the modification is undone by the next pipeline-run and the table is fully rewritten.

_Disadvantages/Limitations_
- Identity columns
- now() and "system"-datetime columns
- When will work incrementally, when will it do a full recompute?
- What if I want to change the logic of my code? -> Recompute?
- Full recomputes will increase costs significantly when working on big tables


## DLT Streaming Tables
- Each input record is processed exactly once. DLT keeps track of what it already processed.
- Streaming tables are designed for data sources that are append-only.
- Good for most ingestion workloads aka. Bronze layer
- Note: If I modify data (using INSERT, UPDATE, …) of a streaming DLT, the modification is kept by the next pipeline-run as only new data is added and the "current" data is untouched.

_Disadvantages/Limitations_
- Guess by Johannes: Aggregations?

## DLT Views
- Records are processed every time the view is queried. Use views for intermediate transformations and data quality checks that should not be published to public datasets. But: They leverage caching algorithms to not always have to query the source.
- Are similar to a temporary view in SQL 
- Allow you to name and reuse a given computation/transformation of a source
- Are available from within a pipeline only and cannot be queried interactively after the pipeline





# Pros and Cons of DLT
_Advantages_
- Automatic dependency resolution of tasks/notebooks in pipeline orchestration. This reduces the complexity of pipeline creation and maintenance.
- Automatic cluster management (which cluster, how many workers)
- Incremental Data Processing: DLT supports incremental processing, which is resource-efficient as only new or changed data is processed (see Checkpoints). 
- Data Quality and Validation: With "Expectations," users can define validation rules to ensure data integrity. Invalid data is quarantined, enabling a more reliable data flow and reducing the likelihood of bad data impacting downstream processes.

_Disadvantages_
- Execution of DLT code is not fully possible while developing
- The DLT-engine: It is hard to know what is going on under the DLT-hood e.g. to know when a full recompute is done or not.
- Vendor Lock-In: DLT is optimized for the Databricks ecosystem. Migrating to another platform could be complex.


