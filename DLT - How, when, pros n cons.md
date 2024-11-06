There are 3 data set types in DLT
__DLT Tables = Materialized Views__
- Records are processed as required to return accurate results for the current data state. Materialized views should be used for data sources with updates, deletions, or aggregations, and for change data capture processing (CDC).
- The Delta Live Tables runtime automatically creates tables in the Delta format and ensures those tables are updated with the latest result of the query that creates the table.
- Tables can be temporary: If used, it instructs Delta Live Tables to create a table that is available during execution of the pipeline only. 
- Note: If I modify data (using INSERT, UPDATE, …) of a normal DLT table, the modification is undone by the next pipeline-run and the table is fully rewritten.

__DLT Streaming Tables__
By default, streaming tables require append-only sources.
Each record is processed exactly once.
- Note: If I modify data (using INSERT, UPDATE, …) of a streaming DLT, the modification is kept by the next pipeline-run as only new data is added and the "current" data is untouched.

__DLT Views__
- Records are processed when the view is queried. Use views for intermediate transformations and data quality checks that should not be published to public datasets.
- are similar to a temporary view in SQL 
- are an alias for some computation (or transformation)
- allow you to reuse a given computation/transformation as a source (for more than one table)
- are available from within a pipeline only and cannot be queried interactively after the pipeline



Suggestion from Databricks
- Use 
