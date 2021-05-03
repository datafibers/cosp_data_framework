<p align="left"><img src="https://cryptologos.cc/logos/binance-coin-bnb-logo.png" width="150">   <img src="https://miro.medium.com/max/9074/1*RoFb2sFULMV-gnOy727FoQ.png" width="400"></p>

COSP Data Framework (CDF) is designed to be a generic data processing framework to read various of data source, such as hive, flat files, database, elastic search, Kafka, etc), perform across data source transformation, and write to the different destinations. CDP is implemented using Scala and Apache Spark.

# Data Source Specific Files
Follow files are required to be cusomized for each data source. Each pipeline, which reads, transform, and write data, is called a application. Here, we use ```APP_CODE``` to identify each application.

| File Name | Functions|
|-----------|----------|
|`src/main/resources/conf/app_{APP_CODE}.yml`| data source specific configuration file|
|`src/main/resources/sql/app_{APP_CODE}.sql`| data source specific configuration file|
|`src/test/resources/setup/tsql_${APP_CODE}.sql`| data source specific test data preparision file|

# Appliction Configurations
All application settings are configured through YML files in the ```src/main/resources/conf/application.yml.{env}```. They are different sessions for Spark, Hive, Elastic settings applied to the whole project.

# Data Source/Target Configurations
In the following sessions, we'll introduce how to configurate the source/target config files in ```src/main/resources/conf/app_{APP_CODE}.yml```.
## Generic Source Configuration Keys
Following yml keys are supported for reading different data sources. These keys are defined under `source` key in the yml files.

|Key            |Request  |Default| Functions|
|-----------|----------|-----------|----------|
|`database`|optional|default|hive database name. Can also be specified with table name, such as, database_name.table_name|
|`regx`|optional| |database name with regular expression, such as test_prod_([0-9]{8}).|
|`table`|mandatory||hive table name.|
|`alias`|optional|table|alias for the data source|
|`type`|optional|reference|table type, driver or referenec|
|`row_filter`|optional||row number filter with partition by and order by fields|
|`read_strategy`|optional|all|how to read data, all -  all data. latest - the latest partition.|
|`look_back`|optional|12|number of database to read according to `regx`|
|`disabled`|optional|false|diable this source's config|

## Flat File Specific Configuration Keys
|Key            |Request  |Default| Functions|
|-----------|----------|-----------|----------|
|`path`|mandatory| |the path where to read the file. `*` is supported to match all.|
|`delimitor`|optional| `\|`|file fields delimitor |
|`quote`|optional|`""`|file quote|
|`optional_fields`|optional||list of comma seperated columns when they are not avaliable in source giving default value as placeholder |

## Generic Target Configuration Keys
Following yml keys are supported for writing different data targets.

|Key            |Request  |Default| Functions|
|-----------|----------|-----------|----------|
|`disabled`|optional|false|disable this target's config|
|`outputs`|optional| |when it is specified, you can define multiple output under this key|
|`output`|implicit| |output file root path in HDFS/ELASTIC. By default it gets value from `${DE_OUTPUT_ROOT_PATH}`|
|`output_log`|implicit| |output log root path in HDFS. By default it gets value from `${DE_LOG_ROOT_PATH}`|
|`output_type`|optional|csv|output type supported, such as csv, paruqet, avro, json, hive, elastic.|
|`hive_db_tbl_name`|optional| |when `output_type` is hive, use `database_name.table_name`.|
|`idx_name`|optional| app_code|when `output_type` is elastic, this is index name.|
|`@timestamp`|optional| |used in index name as metadata to append current timestamp to the index name to make it unique.|
|`idx_create`|optional|false |whether to create the index before writing data.|
|`idx_purge_on_time`|optional|false |when index is created with `@timestamp`, old indices are removed when it is `true`.|
|`idx_alias_create`|optional|false |whether to create alias|
|`idx_alias`|optional|idx_name|alias of the index, by default it is index name without timestamp|
|`idx_setting`|optional|  |whether to add setting section during index creation|
|`idx_map_rule`|optional| |type - generate mapping from dataframe data type. name - generate mapping from column postfix, such as address--idxm-keyword|
|`idx_type_else`|optional|text:dummy|this is requested when `idx_create` is `true`. For example, text:col1,col2, this is to set column with string type for col1 and col2 as text. For other string type columns, set it as keyword.|
|`idx_prop_ov`|optional|  |this is where to pass whole json string as mapping properties to overwrite auto generated index mappings|

## Other Application Configuration Keys
|Key            |Request  |Default| Functions|
|-----------|----------|-----------|----------|
|`job_disabled`|optional|false|disable this application code job by skipping it.|
|`dry_run`|optional|false|verify the job without running the job.|
|`cob_discover`|optional|false|auto discover the cob in yyyy-MM-dd from source read.|
|`lookback_defalt`|optional| |default lookback value for all source data.|
|`sql_resource_root`|optional|../sql|default location where to read the sql files|
|`sql_file_part`|optional||specify how many sql partial files to read in bulk|
|`sql_init`|optional|false|true, load `init.sql` ahead running all other sql files.|
|`sql_init_global`|optional|false|true, load `AppDefaultConfig.GLOBAL_INIT_SQL` ahead running all other sql files.|
|`sql_casesensitive`|optional|false|same to Spark sql case sensitive setting|
|`app_name`|implicit||application name|
|`app_code`|implicit||application code|

**Note:**
1. when `output_type` is elastic, the `etl_es_id` in the dataframe maps to `es.mapping.id`, which is the key of index.
1. when `output_type` is elastic, the `etl_batch_id` defines how many batch to divide and load data to elastic in sequence.