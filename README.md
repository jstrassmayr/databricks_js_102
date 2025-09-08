# databricks_js_102 - Delta Live Tables (DLT)

This is a follow-up databricks learning repo based on my [databricks_js_101](https://github.com/jstrassmayr/databricks_js_101). Its main focus is working with [Delta Live Tables aka. DLT](https://www.databricks.com/de/product/delta-live-tables). To see how and when (not) to use DLT and potential pros and cons, see [this page](https://github.com/jstrassmayr/databricks_js_102/blob/main/DLT%20-%20How%2C%20when%2C%20pros%20n%20cons.md).


# Topics covered
- What is DLT
- Hands-on: Baby names (again) ;-)
- Download data - to Raw layer
- Our first DLT-table - in Bronze layer
- Let's create our first DLT-pipeline
- Let's get fancy - Silver layer

# What is DLT
DLT is a concept and tool-set that helps you building ETL pipelines. 
Instead of building your transformations AND orchestrations AND clusters etc you only have to define the transformations on your data.
DLT handles orchestration-pipelines (=transformation dependencies), cluster management, monitoring, data quality and error handling for you.

## How
You create your DLT datasets like [Tables, Streaming Tables or Views](https://docs.databricks.com/en/delta-live-tables/index.html#what-are-delta-live-tables-datasets) by defining where they read from (data source) and where they write to (data sink). At runtime, Databricks will automatically resolve the dependencies for you correctly. 

*Example:* Let's assume you have these data objects:
1) DLT-table X gets its data from datasource A and
2) DLT-table Y gets its data from DLT-table X.

*Result:* ```Table A --> DLT Table X --> DLT Table Y```
When asking Databricks to update the DLT-table Y it will run the load of DLT-table X as it recognizes the dependencies. This saves manual work for setting up orchestration-pipelines.


# Hands-on: Baby names (again) ;-)

## Good to know
1. If running your DLT-code does not work, please check [this Article](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/tutorial-pipelines#requirements) by MS to check for upfront requirements.
2. A notebook that contains DLT statements (e.g. ```import dlt```) cannot be run directly but only from the pipeline itself.

## Create your schema
I (Johannes) created a catalog named 'dbx_dlt_102' specifically for this learning session. 
- Create your own schema name within this catalog using the pattern "<firstname3><lastname3>_schema" e.g. "johstr_schema"
- Create a (managed) volume within your schema named e.g. "my_files"

## Import the DLT module
Every python notebook that contains DLT code, needs the python DLT-module.
- Create a folder "dbx_dlt_102" in your workspace
- Create a new notebook named "01 Bronze and Silver" within your previously created folder
- Copy and paste the following cell's code into the first cell of your newly created notebook

```python
import dlt
from pyspark.sql.functions import *
```

# Download data - to Raw layer
First we need to download the babyname data from ny.gov into our Raw layer.
- Copy and paste the following cell's code into the first cell of your newly created notebook
- Modify the value of the UNITY_CATALOG_VOLUME_PATH env. variable to meet our requirements. Hint: You can copy the full path from the overview-page of your volume.
- Click "Run cell"
```python
# Raw layer
import os

os.environ["UNITY_CATALOG_VOLUME_PATH"] = "/Volumes/<catalog-name>/<schema-name>/<volume-name>/" # don't forget the trailing slash (/)
os.environ["DATASET_DOWNLOAD_URL"] = "https://health.data.ny.gov/api/views/jxy9-yhdk/rows.csv"
os.environ["DATASET_DOWNLOAD_FILENAME"] = "rows.csv"

dbutils.fs.cp(f"{os.environ.get('DATASET_DOWNLOAD_URL')}", f"{os.environ.get('UNITY_CATALOG_VOLUME_PATH')}{os.environ.get('DATASET_DOWNLOAD_FILENAME')}")
```

# Our first DLT-table - in Bronze layer
Let's load data from our CSV file into a table in our Bronze layer with only one modification: The column 'First Name' must be renamed as space-characters (" ") are not allowed in column names.
- Copy and paste the following cell's code into the next cell.
  - Note: The ```@dlt.table``` decorator tells the DLT-system to create a table that contains the result of a DataFrame returned by a function. Add the ```@dlt.table``` decorator before any Python function definition that returns a *Spark DataFrame* to register a new table in Delta Live Tables.
- Click "Run all"
```python
# Bronze layer
@dlt.table(
  comment="Popular baby first names in New York"
)
def baby_names_raw():
  df = spark.read.csv(f"{os.environ.get('UNITY_CATALOG_VOLUME_PATH')}{os.environ.get('DATASET_DOWNLOAD_FILENAME')}", header=True, inferSchema=True)
  df_renamed = df.withColumnRenamed("First Name", "First_Name") # columns with spaces need to be renamed
  return df_renamed
```
This will lead to an error as mentioned in the 'Good to know'-section. In order to execute the code, we need to create a DLT pipeline first.



# Let's create our first DLT-pipeline
- Click 'Delta Live Tables' in the sidebar and click Create Pipeline.
- Enter a name e.g. 'babynames dlt johstr'
- Select 'Serverless'. 
- Select 'Triggered' for Pipeline Mode.
- In section 'Source code': Click the File Picker Icon and find your notebook 
- In section 'Destination': Choose our catalog 'dbx_dlt_102' and your Target schema
- Click Create.

![image](https://github.com/user-attachments/assets/f8e8281a-0c03-40f4-b065-24792ee7936a)
An empty page will show as the pipeline has never been run alongside the pipeline details.

## Starting the pipeline
- Click the 'Start' button on top right

This will do the following
 1. A cluster is started and the pipeline's notebook(s) are executed
 2. Any tables that don’t exist are created (after ensuring that the schema is correct for any existing tables).
 3. Updates tables with the latest data available.
 4. Shuts down the cluster when the update is complete (in 'Production' mode only)

When using "Development" mode, the cluster is not shut down immediately but kept for a few minutes to be re-used if you start the pipeline again (which is a regular thing to do while developing ;-).
![image](https://github.com/user-attachments/assets/d3e08611-1efe-41bb-a45a-5a084213b8c8)

## Let's explore the new data
- Go to Catalog -> dbx_dlt_102 -> your schema -> your table
- Choose 'Sample data' (and select a compute if asked)

![image](https://github.com/user-attachments/assets/3a500b4f-0a33-4aa4-9fc0-99d4178c8b05)
Ok, we see data. But the result is rather boring as nothing fancy or unexpected happened. But...

# Let's get fancy - Silver layer
In order to get the advantages mentioned at the beginning of this page (e.g. table dependency resolution, pipeline self creation, invalid data handling...), we need an "actual pipeline" of multiple tables and dependencies between them.
We will load data from Bronze to Silver layer doing the following enhancements
- Let's add tests aka. data validations to only ingest valid data into our Silver layer. We want...
  - Warnings from rows containing the first name 'OLIVIA'
  - To drop rows where Count is 0
  - To fail the pipeline immediately if no first name was given
- Let's rename the column 'Year' to 'Year_Of_Birth' to make it more understandable in business terms

Proceed as follows
- Open your source code notebook
- Copy and paste the following cell's code into the next cell
```python
# Silver layer
@dlt.table(
  comment="New York popular baby first name data cleaned and prepared for analysis."
)
@dlt.expect("no_olivias", "First_Name <> 'OLIVIA'")
@dlt.expect_or_drop("valid_count", "Count > 0")
@dlt.expect_or_fail("valid_first_name", "First_Name IS NOT NULL")
def baby_names_silver():
  df = dlt.read("baby_names_raw")
  df_renamed = df.withColumnRenamed("Year", "Year_Of_Birth")
  return df_renamed
```
- Run again: Click on the compute selector (top right) and attach the notebook to your previously created pipeline (e.g. babyname_dlt_johstr) and from now on: Simply hit the "Start" button to start your notebook (or actually the pipeline containing this notebook).

You will now see 2 tables in your pipeline graph: baby_names_raw and baby_names_silver.

## Verify data quality
- Click on 'Delta Live Tables' in the left sidebar
- Open your specific pipeline
- Tap on the table for which we defined '[expectations](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/expectations)' (silver table)
- Select the tab 'Data quality' in the right panel.

![image](https://github.com/user-attachments/assets/68482be5-8cce-42d1-b9db-872507337548)
You will notice that data validation shows 532 failed rows from the expectation named 'no_olivias'. Since we just wanted a warning but no row-droppings for Olivias, the 'Write'-rate is still 100%.

### Expectation handling
| Action         | Type (Python)  | Result                                                                                                              |
|----------------|----------------|---------------------------------------------------------------------------------------------------------------------|
| warn (default) | expect         | Invalid records are written to the target; failure is reported as a metric for the dataset.                         |
| drop           | expect_or_drop | Invalid records are dropped before data is written to the target; failure is reported as a metrics for the dataset. |
| fail           | expect_or_fail | Invalid records prevent the update from succeeding. Manual intervention is required before re-processing.           |


### Dependency resolution
To show that the order of the DLT table definitions in the notebook is irrelevant for the execution, you could simply drag and drop the ```Silver layer``` cell above the ```Raw layer``` cell and execute the pipeline again.

# Gold layer
Let's extend our pipeline by creating a table in a Gold layer which contains the top baby names of 2021.
- Open your folder "dbx_dlt_102" in your Workspace
- Create a new notebook named "02 Gold"
- Copy and paste the following cell's code into the first cell
```python
# Gold layer
import dlt
from pyspark.sql.functions import *
```
- Copy and paste the following cell's code into the next cell
```python
@dlt.table(
  comment="A table summarizing counts of the top baby names for New York for 2021."
)
def top_baby_names_2021():
  return (
    dlt.read("baby_names_silver")
      .filter(expr("Year_Of_Birth == 2021"))
      .groupBy("First_Name")
      .agg(sum("Count").alias("Total_Count"))
      .sort(desc("Total_Count"))
      .limit(10)
  )
```
- Click 'Delta Live Tables' in the sidebar and open your pipeline
- Click on 'Settings' (top right) and scroll down to 'Source code'
- Add your '02 Gold' notebook as additional source code
- Save and Start your pipeline

This is what you should see:

![image](https://github.com/user-attachments/assets/93d762ff-fcfa-40fa-8654-8d0613d3fb27)


