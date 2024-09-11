# databricks_js_102 - Delta Live Tables

This is a follow-up databricks learning repo based on my [databricks_js_101](https://github.com/jstrassmayr/databricks_js_101). Its main focus are [Delta Live Tables aka. DLT](https://www.databricks.com/de/product/delta-live-tables).

# What is DLT (for)
DLT is a concept and tool-set that helps you building ETL pipelines. 
Instead of building your transformations AND orchestrations AND clusters etc you only have to define the transformations on your data.
DLT handles orchestration-pipelines (=transformation dependencies), cluster management, monitoring, data quality and error handling for you.

## How
You create your DLT objects (Tables, Streaming Tables, Views) by defining where they read from (data source) and where they write to (data sink). At runtime, Databricks will automatically resolve the dependencies for you correctly. 

*Example:* Let's assume you have this data objects:
1) DLT-table X gets its data from datasource A and
2) DLT-table Y gets its data from DLT-table X.

*Result:* ```Table A --> DLT Table X --> DLT Table Y```
When asking Databricks to update the DLT-table Y it will run the load of DLT-table X as it recognizes the dependencies. This saves manual work for setting up orchestration-pipelines.


# Hands-on: Baby names (again) ;-)

## Good to know
1. If running your DLT-updates does not work, please check [this Article](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/tutorial-pipelines#requirements) by MS to check for requirements.
2. A notebook that contains DLT statements cannot be run directly but only from the pipeline itself.


See https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/tutorial-pipelines



