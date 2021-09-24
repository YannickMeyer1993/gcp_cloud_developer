# Big Query x Transformers

```
BigQuery
Anzeigen des Meta-Daten eines Datasets:
Ermittlung der nicht mehr benötigten Datasets in brain-jobber-tst:
BigQuery in Cloud Shell
Löschen eines Datasets in der Cloud Shell (Projekt setzen auf brain-jobber-test)
Erstellen eines Datasets in der Cloud Shell (Projekt setzen auf brain-jobber-test)
JSON Function in BigQuery
Example
STRUCT's and ARRAY's in BigQuery
Example: Querying Nested JSON with UNNEST
ARRAY Functions
ARRAY_AGG()
ARRAY_LENGTH()
```
## BigQuery

## Anzeigen des Meta-Daten eines Datasets:

```
-- Returns metadata for tables in a single dataset.
SELECT * FROM dataset_name.INFORMATION_SCHEMA.TABLES;
```
```
-- Only returns table names
SELECT table_name FROM dataset_name.INFORMATION_SCHEMA.TABLES;
```
Zusätzliche Infos: https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/courses/data-engineering/demos/information_schema.md

## Ermittlung der nicht mehr benötigten Datasets in brain-jobber-tst:

#### SELECT

```
* EXCEPT(schema_owner)
FROM
`brain-jobber-tst`.INFORMATION_SCHEMA.SCHEMATA
where last_modified_time <= TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL -2 HOUR)
and schema_name like 'transformers_%'
order by creation_time;
```
ACHTUNG: Ggf. ist es erforderlich in den Abfrageeinstellungen den Standort auf EU zu ändern!

## BigQuery in Cloud Shell


### Löschen eines Datasets in der Cloud Shell (Projekt setzen auf brain-jobber-test)

```
gcloud config set project brain-jobber-tst
bq rm -r -f -d <datasetname aus liste>
```
### Erstellen eines Datasets in der Cloud Shell (Projekt setzen auf brain-jobber-test)

```
gcloud config set project brain-jobber-tst
bq --location=EU mk -d --description "This is my dataset" mydataset
```
## JSON Function in BigQuery

```
Function Purpose
```
```
JSON_EXTRACT_SCALAR Extracts JSON Fields from a JSON
```
```
JSON_EXTRACT You can query the whole field of a JSON, even if it is not a scalar
```
```
JSON_QUERY Same
```
```
JSON_QUERY_ARRAY Enables you to query an array within a JSON
```
```
JSON_VALUE_ARRAY Same
```
The primary difference between the two functions is that JSON_QUERY returns an object or an array, while JSON_VALUE returns a scalar. Same with
JSON_QUERY_ARRAY/JSON_VALUE_ARRAY

### Example

```
-- Spalte ids = Array
SELECT ids.key
FROM `brain-lethe-tst.LETHE_SHARED_TRANSFORMERS.records_to_delete`, UNNEST(ids) as ids
```
## STRUCT's and ARRAY's in BigQuery

Main Information Source: https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/courses/data-engineering/demos/nested.md

These types of data types are used to unite the advantages of Normalization and Denormalization. You will not have duplicate data and you still have fast
joins over all tables. Structs are pre-joined tables within a table. They have the data type RECORD and the mode is REPEATED. The function count(*) will
count the amount of entries in the "main" columns. Arrays have the data type ARRAY and the mode is also REPEATED. Arrays can be part of regular
fields or structs. A single table can have many structs. Structs can have other structs nested inside of it. BigQuery stores repeated, nested fields in a
columnar format (value + definition level + repetition level). Using Structs results in less Data Shuffling and more efficient joins.

Just as ARRAY values give you the flexibility to go deep into the granularity of your fields, another data type allows you to go wide in your schema by
grouping related fields together. That SQL data type is the STRUCT data type. STRUCTs (and ARRAYs) must be unpacked before you can operate over
their elements. Wrap an UNNEST() around the name of the struct itself or the struct field that is an array in order to unpack and flatten it.

Syntax: Inside the world of Structs, the comma is a correlated CROSS JOIN. With Array, you need to unnest the columns via the UNNEST command. The
columns have different granularity and with UNNEST we break down the repeated part in the array.

### Example: Querying Nested JSON with UNNEST


#### WITH

```
base AS (
SELECT
dl_payload,
dl_partition_day_utc,
dl_insert_timestamp_utc,
gravitas_metadata
FROM
brain-mktg-campaign-prd.kuvo.view_partnerincentive )
SELECT
JSON_VALUE(dl_payload, '$.data.kuvoUUID') AS partner_incentive_id,
JSON_VALUE(dl_payload, "$.data.partner.id") AS partner_id,
JSON_VALUE(dl_payload, "$.data.kuvoName") AS partnerincentive_name,
JSON_VALUE(dl_payload, "$.data.kuvoShortName") AS partnerincentive_short_name,
JSON_VALUE(dl_payload, "$.data.conditionText") AS condition_text,
JSON_QUERY(dl_payload, '$.data.accountingConditions') AS accounting_condition_array, --get whole ARRAY
JSON_VALUE(dl_payload, '$.data.accountingConditions[0].marketplace') AS accounting_condition_marketplace,
JSON_VALUE(dl_payload, '$.data.accountingConditions[0].partner') AS accounting_condition_partner,
JSON_VALUE(dl_payload, "$.data.unit") AS partnerincentive_unit,
JSON_VALUE(dl_payload, "$.data.value") AS partnerincentive_value,
JSON_VALUE(dl_payload, "$.data.partner.name") AS partner_name,
JSON_VALUE(dl_payload, "$.data.partner.shopName") AS partner_shop_name,
JSON_VALUE(dl_payload, "$.data.campaignName") AS campaign_name,
JSON_VALUE(dl_payload, "$.data.periodStart") AS period_start,
JSON_VALUE(dl_payload, "$.data.periodEnd") AS period_end,
JSON_VALUE(dl_payload, "$.data.status") AS partnerincentive_status,
JSON_VALUE(dl_payload, "$.data.eventTimestamp") AS event_timestamp,
JSON_VALUE(number, '$.mode') AS assortment_condition_mode,
JSON_VALUE(number, '$.type') AS assortment_condition_type,
assortment_condition_value,
JSON_VALUE(dl_payload, "$.event.eventId") AS event_Id,
JSON_VALUE(dl_payload, '$.event.eventState') AS event_state,
JSON_VALUE(dl_payload, '$.event.eventType') AS event_type,
dl_partition_day_utc,
dl_insert_timestamp_utc
FROM
base
, UNNEST(JSON_QUERY_ARRAY(JSON_QUERY(dl_payload, '$.data.assortmentConditions')) ) AS number --comma
instead of CROSS JOIN
, UNNEST(JSON_VALUE_ARRAY(JSON_QUERY(number, '$.values'))) AS assortment_condition_value --comma
instead of CROSS JOIN
-- JSON_VALUE_ARRAY because we want the value of the string and not the object (which is '\"string\"')
;
```
## ARRAY Functions

### ARRAY_AGG()

You are also able to revert the result set of a query to an array by using the function ARRAY_AGG()


#### SELECT

```
fullVisitorId,
date,
v2ProductName,
pageTitle
FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
ORDER BY date
```
```
SELECT
fullVisitorId,
date,
ARRAY_AGG(v2ProductName) AS products_viewed,
ARRAY_AGG(pageTitle) AS pages_viewed
FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date
```
### ARRAY_LENGTH()

Query the length of an array by using ARRAY_LENGTH().

#### SELECT

```
fullVisitorId,
date,
v2ProductName,
pageTitle
FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
ORDER BY date
```
```
SELECT
fullVisitorId,
date,
ARRAY_AGG(v2ProductName) AS products_viewed,
ARRAY_AGG(pageTitle) AS pages_viewed
FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date
```

