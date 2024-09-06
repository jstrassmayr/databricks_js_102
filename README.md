# databricks_js_102 - Delta Live Tables

This is a follow-up databricks learning repo based on my [databricks_js_101](https://github.com/jstrassmayr/databricks_js_101). Its main focus are [Delta Live Tables aka. DLT](https://www.databricks.com/de/product/delta-live-tables).

# What is it (for)
DLT is a concept and tool-set that helps you building ETL pipelines. 
Instead of building your transformations, orchestrations, clusters, tests etc you only have to define the transformations on your data.
DLT handles orchestration-pipelines, cluster management, monitoring, data quality and error handling for you.

## How
You create your DLT objects (Tables, Streaming Tables, Views) by defining where they read from (data source) and where they write to (data sink). At runtime, Databricks will automatically resolve the dependencies for you correctly. 

*Example:* Let's assume you have this data objects:
1) DLT-table X gets its data from datasource A and
2) DLT-table Y gets its data from DLT-table X.

Then: Databricks DLT will run the load of DLT-table X before DLT-table Y as it recognizes the dependencies given upfront. This saves manual work for setting up orchestration-pipelines.

*Result:* ```Table A --> DLT Table X --> DLT Table Y```



# Example: Baby names (again) ;-)

See https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/tutorial-pipelines



