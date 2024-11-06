
__3 Data set types__

_DLT Tables = Materialized Views_
- Records are processed as required to return accurate results for the current data state. Materialized views should be used for data sources with updates, deletions, or aggregations, and for change data capture processing (CDC).
- The Delta Live Tables runtime automatically creates tables in the Delta format and ensures those tables are updated with the latest result of the query that creates the table.
- Tables can be temporary: If used, it instructs Delta Live Tables to create a table that is available during execution of the pipeline only. 
- Note: If I modify data (using INSERT, UPDATE, …) of a normal DLT table, the modification is undone by the next pipeline-run and the table is fully rewritten.

_DLT Streaming Tables_
By default, streaming tables require append-only sources.
Each record is processed exactly once.
- Note: If I modify data (using INSERT, UPDATE, …) of a streaming DLT, the modification is kept by the next pipeline-run as only new data is added and the "current" data is untouched.

_DLT Views_
- Records are processed when the view is queried. Use views for intermediate transformations and data quality checks that should not be published to public datasets.
- are similar to a temporary view in SQL 
- are an alias for some computation (or transformation)
- allow you to reuse a given computation/transformation as a source (for more than one table)
- are available from within a pipeline only and cannot be queried interactively after the pipeline



Suggestion from Databricks
- Use DLT Streaming ables only for Bronze layer
- 


_Pros and Cons of DLT_
Advantages
# Automatic dependency resolution of tasks/notebooks in pipeline orchestration. This reduces the complexity of pipeline creation and maintenance.
# Incremental Data Processing: DLT supports incremental processing, which is resource-efficient as only new or changed data is processed (see Checkpoints). 
# Data Quality and Validation: With "Expectations," users can define validation rules to ensure data integrity. Invalid data is quarantined, enabling a more reliable data flow and reducing the likelihood of bad data impacting downstream processes.

Disadvantages
# Execution of DLT code is not fully possible while developing
# The DLT-engine: It is hard to know what is going on under the DLT-hood e.g. to know when a full recompute is done or not.
# Vendor Lock-In: DLT is optimized for the Databricks ecosystem. Migrating to another platform could be complex.

_Disadvantages/Limitations of DLT tables aka. Materialized views_
# Identity columns
# now() and "system"-datetime columns
# When will work incrementally, when will it do a full recompute?
# What if I want to change the logic of my code? -> Recompute?
# Full recomputes will increase costs significantly when working on big tables
# 
