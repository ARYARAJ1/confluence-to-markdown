# Metadata Management

<div class="contentLayout2">

<div class="columnLayout fixed-width" layout="fixed-width">

<div class="cell normal" data-type="normal">

<div class="innerCell">

## Introduction

This design is to cater for ad-hoc metadata load and management. This
metadata could be business related or reference data/information used to
help standardised/ conform data in the SDI data warehouse and Data
marts.

This load can be designed on a low frequency load schedule
(Monthly/quarterly) combined with on need basis triggers.

</div>

</div>

</div>

<div class="columnLayout two-equal" layout="two-equal">

<div class="cell normal" data-type="normal">

<div class="innerCell">

### Option 1 - RC XLSX pipe reusability

-   existing pattern XLSX/csv to parquet -\> Snowflake

-   Not many changes required, same RC hyper pipe

</div>

</div>

<div class="cell normal" data-type="normal">

<div class="innerCell">

### Option 2 - New Hyper Pipe

-   Existing RC pipes loads into SF curated, not needed Metadata can go
    directly into modelled

-   parquet is complex, not required, might have troubleshooting
    bottlenecks.

-   RC, IDR and CC (parquet pattern), all obsolete after 4-5 months

</div>

</div>

</div>

<div class="columnLayout fixed-width" layout="fixed-width">

<div class="cell normal" data-type="normal">

<div class="innerCell">

### Proposed Solution -

-   Mirror Risk Console XLSX ingestion hyper pipeline

-   Remove unnecessary steps

-   Modify TDQ to work on csv files

## High Level Design

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio-sketch4081792586775646763"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio-sketch4081792586775646763"
class="ap-content">

</div>

</div>

The above is high level design of the Hyper Pipeline for the metadata
management. It would entail of the below steps -

-   XLSX files to be fetched from K drive. Filename to be consistent
    with the entity being loaded.

-   The table/structure to be created as part of the load would be
    gathered and maintained as configuration information in the COSMOS
    DB. Table/Entity name same as Filename

-   The file will be fetched in the Azure data lake landing container in
    a blob storage folder (same as filename).

-   File gets fetched from landing container, converted to CSV and is
    placed in raw container with timestamped folder structure (refer
    Risk console XLSX load).

-   Perform basic Tech Data Quality checks on the file.

    -   TDQ will be obsolete with the legacy source systems.

    -   Tech Data Quality will need to be modified to handle CSV
        validations and load the data into Snowflake DB and schema as
        per configuration (currently, it is hardcoded to load into
        structured and curated DB. Need to change that to include DWH
        DB).

    -   We will copy and make a new version of TDQ to use in Metadata
        pipelines, existing TDQ will keep being used in non Pega
        pipelines and will be decommissioned when the legacy non Pega
        systems seize to exist. This allows us to skip regression
        testing the impact of the TDQ changes on existing PROD pipes.

    -   It should load from Azure Data Lake → Snowflake Stage →
        Transient table (Structured DB or staging schema in DWH DB)

-   Type 2 SCD performed on the data and loaded in the final Modelled
    schema. Generic SCD stored procedure can be used for this.

## Operational Info

-   Files to be placed in K drive - SDI_prd.

-   Scheduled triggers would be made for more frequently changing files,
    else the ingestion would be manually triggered, on need/request
    basis.

-   File related issues would be conveyed to the source of the file for
    fixture. More relevant error handling would evolve into the TDQ
    function in future as per the need.

</div>

</div>

</div>

</div>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design](attachments/1178763363/1178632310)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design.png](attachments/1178763363/1179025506.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/1178763363/1178632355.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/1178763363/1178632350.tmp)
(application/vnd.jgraph.mxfile)  

</div>
