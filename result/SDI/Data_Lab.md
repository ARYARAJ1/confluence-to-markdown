# Data Lab

Data lab is an exploratory space/environment created for Insights team’s
analysts / developers to -

-   Explore new data sets to create / test new insights use cases/ideas

-   experiment / analyse existing / new data sets

-   other insights related work

# Assumptions -

All files/API within one ingestion pattern have the same schema
structure

# Scope -

There are 2 ingestion patterns introduced -

1.  Geojson Files from network drive &

2.  APIs - 2 types of API loads

    1.  Token Authorization - This is for the zoom meeting data that
        needs authentication

    2.  No Authorization - Geospatial API data that does not need
        Authorization as it is publicly available

# Initial Setup -

A new user is configured by a request to the data engineer. This setup
will create a user specific DATALAB schema (DATALAB\_\<user initials\>)
on the Snowflake SDI\_\<ENV\>\_CURATED_DB. Here the user will have read
access on the data from all other schemas
(\<SOURCESYSTEM\>\_\<CLASSIFICATION\> ex: PEGA_PROTECTED) and write
access to this new Datalab schema.

New storage integration created between azure and snowflake for for file
to DB copy

Linked services are configured for specific data sources (network
drives/API URLs)

RBAC is setup, New security group in azure and role on Snowflake is
created for specific user access

The user can then access Azure Data factory to execute the required
hyper pipelines to ingest new data as per the need/use case.

Based on the ingestion patterns mentioned above, 2 new hyper pipelines
are created -

1.  datalabHyperPipeline

2.  datalabApiHyperPipeline

## Technical Design - Hyper Pipeline

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio7400380567878607422"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio7400380567878607422"
class="ap-content">

</div>

</div>

## Step details -

parameter parser parses through the input provide for the hyper
pipeline. This is further used throughout the pipeline as parameters.

1.  In acquisition layer, check for token value in parameters and if
    provided, triggers the copy activity of zoom API, else geospatial
    API. The linked service/dataset connects to the network drive (API
    URL, in case of API hyper pipeline) to fetch the file (data and
    create a json file, in case of API hyper pipeline) and places it in
    the landing container of the data lake

2.  In Structured layer, the following steps are performed -

    1.  Azure function copy_snowflake_datalab is triggered. This
        function reads file from LANDING container and strips the outer
        elements of the json file to only capture the array of actual
        values. This is done to catered for the json files which are
        bigger than the max threshold of column size that snowflake can
        handle (16 MB). (All the files are stripped outer element
        through snowflake configuration and also within the Azure
        function to help with the load of bigger json files)

    2.  This new data set is written with the same file name into the
        RAW container.

    3.  The RAW file is then copied over through the Azure Storage
        Integration into SDI\_\<ENV\>\_CURATED_DB.DATALAB\_\<initials\>
        in a RAWJSON\_\<table name\>.

    4.  ~~Schema structure is provided to this data in RAWJSON\_\<table
        name\>, through predefined schema based on each load type within
        the function (to have governance) and the data is loaded into
        the final table for analysis purposes.~~

### Operational Information (Notes)

-   To be able to give access to user to data in the CURATED layer, we
    skip the structured database and copy the Datalab data directly into
    the CURATED database, else we might need to establish a zero copy
    cloning for data between structured and curated DBs.

-   Data lab schema is transient, every execution creates/replaces the
    table (Drops and creates again)  
    The structure of all files for each file type is consistent and
    hence the schema is maintained within the python function that loads
    from Azure Data Lake to snowflake DATALAB schema. If the structures
    tend to change, or a new file type is introduced, It would need
    change in the function or might be configured (based on the use
    case). Reason of this is to maintain governance.

-   Constraint - Every linked service and dataset needs a base and
    relative URL and so every time a new API is to be extracted which is
    from a different URL space, new linked service and dataset needs to
    be created, hyper pipeline might also need to be customised.
    Example - 2 new linked service and data sets are configured in data
    factory to extract 2 APIs currently requested. (zoom URL and
    Geospatial ArcGIS data)

-   DDL/DML scripts executed and kept in the DATALAB\_\<initials\>
    schema needs to be version controlled in Bitbucket as scheduled
    clean-up might result in removal of these scripts.

-   This solution is not meant/equipped for anything except exploratory
    data analysis / experimental use case by the Insights analysts.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/982220850.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/981893241.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/982220859.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/981532732.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/982220868.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/981532741.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/981532750.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/981237819.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/982220877.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/982155350.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design](attachments/981106699/982220925)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design.png](attachments/981106699/982220935.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f97265e44658b00710f438d\~High Level
Design.tmp](attachments/981106699/981139476.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design](attachments/981106699/991002697)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design.png](attachments/981106699/990576728.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design](attachments/981106699/1153597443)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design.png](attachments/981106699/1153302560.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design](attachments/981106699/1154154601)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design.png](attachments/981106699/1154056299.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1153761354.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1153466463.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1153761364.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1153597519.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1153597529.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1153237091.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1153466473.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1154121769.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/1155465337.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design](attachments/981106699/1155432693)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design.png](attachments/981106699/1155530864.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~High
Level Design.tmp](attachments/981106699/981237845.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design](attachments/981106699/982155345)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [High
Level Design.png](attachments/981106699/981237831.png) (image/png)  

</div>
