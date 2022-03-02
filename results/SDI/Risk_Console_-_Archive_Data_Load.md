# Risk Console - Archive Data Load

## Introduction

This is the final once off feed of all the data to be received form Risk
Console and to be loaded into Snowflake Curated database for fetching
into data warehouse layer for further reporting purposes.

## Data Pipeline

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio-sketch1005598145458131433"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio-sketch1005598145458131433"
class="ap-content">

</div>

</div>

The below manual activates are done to facilitate the load since this is
an once off activity -

<div class="panel"
style="background-color: #EAE6FF;border-color: #998DD9;border-width: 1px;">

<div class="panelContent" style="background-color: #EAE6FF;">

Since this is once off load. The load in snowflake will happen directly
into curated layer.

</div>

</div>

-   Created a schema “RISKCONSOLE_ARCHIVE” in SDI_PRD_CURATED_DB

-   created a file format and stage as below relevant to the files being
    loaded; where \<ENV\> is the environment where the objects are to be
    created - DEVRP/PRD. Replacing value should be case sensitive

    <div class="code panel pdl" style="border-width: 1px;">

    <div class="codeContent panelContent pdl">

    ``` sql
    CREATE FILE FORMAT "SDI_<ENV>_CURATED_DB"."RISKCONSOLE_ARCHIVE".FORMAT_RISKCONSOLE_ARCHIVE_CSV TYPE = 'CSV' COMPRESSION = 'AUTO' FIELD_DELIMITER = '|' RECORD_DELIMITER = '\n' SKIP_HEADER = 0 FIELD_OPTIONALLY_ENCLOSED_BY = 'NONE' TRIM_SPACE = FALSE ERROR_ON_COLUMN_COUNT_MISMATCH = TRUE ESCAPE = 'NONE' ESCAPE_UNENCLOSED_FIELD = '\134' DATE_FORMAT = 'AUTO' TIMESTAMP_FORMAT = 'AUTO' NULL_IF = ('\\N');

    create or replace stage AZURE_RISKCONSOLE_ARCHIVE
    storage_integration = AZURE_INTEGRATION_<ENV>_RAW_DATALAKE
    url = 'azure://stsdi<env>datalake.blob.core.windows.net/raw-sdi-<env>-datalake/RISKCONSOLE_ARCHIVE/'
    file_format = FORMAT_RISKCONSOLE_ARCHIVE_CSV;
    ```

    </div>

    </div>

-   Created a click ops pipe to loop through a folder which has all the
    relevant files and copy individual files into raw containers of
    storage account “raw-sdi-\<env\>-datalake”. and then use the above
    stage to read the files and copy into its relevant tables in
    snowflake.

-   Some variables created for the pipelines is as below (to be
    manipulated if needed) -  
    schema - RISKCONSOLE_ARCHIVE  
    folderPath - ‘Apps\\SAS\\From_AON\\VentivDecom’  
    timestamp - ‘\_20210922_0000.txt’ (part of the file to be removed to
    have table names)

## Table Structures

The process to have the tables created for these files is as below -

1.  create configs for the tables to be ingested in <a
    href="https://bitbucket.org/VMIASourceCode/sdi_configuration/src/master/"
    data-card-appearance="inline"
    rel="nofollow">https://bitbucket.org/VMIASourceCode/sdi_configuration/src/master/</a>
    ; build and deploy into cosmos DB

2.  Use the
    /workspaces/sdi_database_definitions/schema_evolution/bin/generate-ddl.sh
    to generate the create statements for these data feeds/tables using
    the configs created in step 1

3.  Run
    /workspaces/sdi_database_definitions/schema_evolution/bin/validation_framework.sh
    to check if the create statements deploy correctly and push these
    changes into further branches until master/PROD

<div class="panel"
style="background-color: #EAE6FF;border-color: #998DD9;border-width: 1px;">

<div class="panelContent" style="background-color: #EAE6FF;">

-   The tables for this load will not have audit columns as there wont
    be any history maintenance ever done.

-   Fetch from this dataset into DWH layer will be a simple select \*
    from \<table_name\> without any conditions/where clause

</div>

</div>

<div>

<div>

Since this is simple fetch and load. The file/tables structure
validations is prerequisite to this ingestion.

</div>

</div>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [RC
Archive Load](attachments/1287847959/1287684224)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [RC
Archive Load.png](attachments/1287847959/1287585883.png) (image/png)  

</div>
