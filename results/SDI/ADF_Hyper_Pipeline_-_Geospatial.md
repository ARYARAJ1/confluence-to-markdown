# ADF Hyper Pipeline - Geospatial

## Introduction

Geospatial Data Load is loaded for the business use cases as mentioned
in <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/1088487472/1.1+Catastrophic+Event+-+Potential"
data-linked-resource-id="1088487472" data-linked-resource-version="26"
data-linked-resource-type="page">1.1 Catastrophic Event - Potential</a>
& <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/1088978976/1.2+Catastrophic+Event+-+Immediate+Danger"
data-linked-resource-id="1088978976" data-linked-resource-version="12"
data-linked-resource-type="page">1.2 Catastrophic Event - Immediate
Danger</a>

## Pre-requisite

To be able to extract data from ArcGIS platform, a python script will be
written, that will -

1.  Extract data for all listed shape files and convert to JSON format
    file

2.  Create delta files for files bigger than 500 MB (**may not be
    required)**,

3.  Strip extra outer elements of the JSON and only capture the array of
    actual values having the key list of values as separate records in
    the file. This is done to catered for the JSON files which are
    bigger than the max threshold of column size that snowflake can
    handle (16 MB). Example -

    <div class="code panel pdl" style="border-width: 1px;">

    <div class="codeContent panelContent pdl">

    ``` java
    {
        "type": "FeatureCollection",
        "crs": {
            "type": "name",
            "properties": {
                "name": "EPSG:4326"
            }
        },
        "features": [
            {
                "type": "Feature",
                "id": 0,
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        145.69557906700004,
                        -37.839049165999938
                    ]
                },
                "properties": {
                    "FID": 0,
                    "PLOT": 1638,
                    "EAST": 385223,
                    "NORTH": 5811241,
                    "TEAM": "FS4",
                    "DLDATA": "17-Apr-2009 13:15",
                    "EXAMIN": "CC,JB",
                    "FIRE": "Bunyip"
                }
            }
        ]
    }
    ```

    </div>

    </div>

    changed to

    <div class="code panel pdl" style="border-width: 1px;">

    <div class="codeContent panelContent pdl">

    ``` java
    {
        "type": "Feature",
        "id": 0,
        "geometry": {
            "type": "Point",
            "coordinates": [
                145.69557906700004,
                -37.839049165999938
            ]
        },
        "properties": {
            "FID": 0,
            "PLOT": 1638,
            "EAST": 385223,
            "NORTH": 5811241,
            "TEAM": "FS4",
            "DLDATA": "17-Apr-2009 13:15",
            "EXAMIN": "CC,JB",
            "FIRE": "Bunyip"
        }
    }
    ```

    </div>

    </div>

    This script is as below and is going to be used by Geospatial
    analyst to create the files listed here
    <a href="https://vmia.atlassian.net/wiki/spaces/VIA/pages/648937473"
    data-linked-resource-id="648937473" data-linked-resource-version="10"
    data-linked-resource-type="page">https://vmia.atlassian.net/wiki/spaces/VIA/pages/648937473</a>  

    <div class="code panel pdl" style="border-width: 1px;">

    <div class="codeContent panelContent pdl">

    ``` py
    import os
    import sys
    import json
    import logging

    logger = logging.getLogger()

    def main():
        in_path = os.path.join(os.path.dirname(os.path.abspath(__file__)),"./in_Files/")
        out_path = os.path.join(os.path.dirname(os.path.abspath(__file__)),"./out_Files/")

        filelist = os.listdir(in_path)
        print(filelist)

        for f in filelist:
            print(f'file processed: {f}')
            in_file = os.path.join(in_path,f)
            out_file = os.path.join(out_path,f)
            r = open(in_file,"r")

            data = json.load(r)

            for key, value in data.items():
                if (type(value)) == list:
                    j_data = data[key]
                else:
                    j_data = data

            j_data_s = json.dumps(j_data).encode('utf-8')
            outfile_path = os.path.join(out_path,f)
            logger.info("Creating JSON File.")
            with open(out_file,"w") as w:
                json.dump(j_data, w)


    if __name__ == "__main__":
        main()
    ```

    </div>

    </div>

## High Level Design -

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio-sketch5703425701399040460"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio-sketch5703425701399040460"
class="ap-content">

</div>

</div>

### Curated Hyper Pipeline

-   The core infrastructure and run-time setup for the ADF
    hyper-pipeline will follow
    <a href="Data_Lab.md" data-linked-resource-id="981106699"
    data-linked-resource-version="11" data-linked-resource-type="page">Data
    lab</a> approach with most of the logical processing and functions
    re-used from
    <a href="ADF_Hyper_Pipeline_-_File_Server_Risk_Console_.md"
    data-linked-resource-id="424116229" data-linked-resource-version="23"
    data-linked-resource-type="page">ADF Hyper Pipeline - File Server (Risk
    Console)</a>

-   Including a config layer for defining/validating structure of the
    data feed

#### Detail Technical Design -

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio-sketch3725352479903668655"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio-sketch3725352479903668655"
class="ap-content">

</div>

</div>

#### Steps

1.  Parameter parser - Azure function to parse and process the
    parameters passed into the ADF pipeline trigger

2.  Acquisition layer

    1.  The linked service/dataset connects to the network (K:/) drive
        (API URL, in case of API hyper pipeline) to fetch the file (data
        and create a json file, in case of API hyper pipeline) and
        places it in the rawcontainer of the data lake

3.  geo_last_pipeline_status_query - use the existing function
    last_pipeline_status_query to check the previous day’s execution for
    the geospatial hyper pipeline was successful or not. Fail the pipe
    if not

4.  Structured layer

    1.  Azure function copy_snowflake is triggered. This function reads
        file from Raw container and copies over through the Azure
        Storage Integration into
        SDI\_\<ENV\>\_STRUCTURED_DB.GEOSPATIAL_OFFICIAL in a
        RAW\_\<table name\>. **Note that this will be unstructured
        data**

    2.  A separate function is added to copy_snowflake to run a create
        statement for structured data based on the table config for
        these feeds. This is to validate the source data based on our
        current understanding of structure of the data feed (Schema
        Validation).

5.  Curated Layer - Use the time varying data load and all the needed
    configurations to be duplicated same as the Risk Console pipeline
    for this step (*Although in this case the merge statement will
    behave very simple, Ignore if present, else insert, no updates
    required. Reason is that the whole record with attributes would be
    considered as one single value to be compared, and so even if
    logically its an update because 1 attribute for the record changed,
    technically its a new record if considered as one single value*).
    The change data capture would be “Full”, so records would also
    soft/inferred delete if it stops coming in from source. **Note that
    this will be unstructured data**

### Geo Data DWH load

The geospatial data loaded into curated layer will be considered as
dimension tables for DWH layer and DWH hyper pipeline to be used for
ingestion - <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/1054113793/Data+Warehouse+Pipeline"
data-linked-resource-id="1054113793" data-linked-resource-version="22"
data-linked-resource-type="page">Detailed technical design - Data
Warehouse hyper pipeline</a>

-   Changes to the DWH config items to include the geo data in the
    ingestion

-   Separate trigger for Geo feeds to decouple from other DWH loads, in
    case of Geo load anomalies, if the geo trigger needs to be stopped,
    it does not impact the rest of the DWH load.

### Benefits -

-   Most of the Data lab pipe re-used with features/logic used from
    Risk-Console pipelines

-   Simplified history retention with un-structured data.

-   Load into DWH can be incorporated with little tweaks in config

### Load Cadence

To be set up individually based on each feed.

At the moment, all entities in the warehouse are loaded daily. The
warehouse takes whatever is valid at the window end time trigger date.
This means that hourly feeds in curated will be refreshed with daily
cadence in the warehouse.

**Note - Test if the hourly load works?**

<div class="table-wrap">

|     |                               |                          |                                                                                                                                                                                                                                        |
|-----|-------------------------------|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     | **Data feed**                 | **Frequency**            | **Solution**                                                                                                                                                                                                                           |
| 1   | Immediate danger bushfire API | Daily and Hourly refresh | Setup hourly refresh on the table level and then create a view to extract distinct data on a daily level (create view \<view name\> as select distinct \* from \<table name\> where valid_from between 9 AM yesterday and 9 AM today). |
| 2   |                               |                          |                                                                                                                                                                                                                                        |

</div>

### Dependent Tables/Views for Geo Analytics -

All view that are required for the Geo analytics use cases are to be
created using the <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/595099738/Stored+Procedures+and+Views+Management+Framework"
data-linked-resource-id="595099738" data-linked-resource-version="10"
data-linked-resource-type="page">Stored Procedures and Views Management
Framework</a> available in the DB definition repo - <a
href="https://bitbucket.org/VMIASourceCode/sdi_database_definitions/src/develop/view_sp_definitions_framework/"
data-card-appearance="inline"
rel="nofollow">https://bitbucket.org/VMIASourceCode/sdi_database_definitions/src/develop/view_sp_definitions_framework/</a>

**Note - Business rules to define protected assets**

Some examples of view creation would be -

-   Creating view on PEGA Asset table to convert longitude and latitude
    information into geography and some other relevant attribute. this
    will be a v_Asset_geo.sql create view V_ASSET_GEO.

-   Creating view for daily snapshot of data from the hourly history
    maintained Bushfire API data as mentioned in point 1, here - <a
    href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/1271070723/ADF+Hyper+Pipeline+-+Geospatial#Load-Cadence"
    data-card-appearance="inline"
    rel="nofollow">https://vmia.atlassian.net/wiki/spaces/SDI/pages/1271070723/Geospatial+Data+Load#Load-Cadence</a>

#### Solution -

The view and stored procedure deployment framework to be modified to be
able to create views in DWH layer alongside the curated layer views.
This change will help us slowly transition the views created in curated
layer for reporting into the DWH layer, if needed, and eventually
phasing out the views created in the curated layer for report
consumption and only have views if required, in DWH/consumption layer.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Geo
Data Curated](attachments/1271070723/1276772475)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Geo
Data Curated.png](attachments/1271070723/1276215371.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276182601.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276182611.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276411967.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276870715.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276739707.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276346497.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276215353.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276772465.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1276313671.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Geo
Data Curated.tmp](attachments/1271070723/1271595022.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Geo
Data Curated](attachments/1271070723/1269727267)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Geo
Data Curated.png](attachments/1271070723/1270087703.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline](attachments/1271070723/1276608804)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline.png](attachments/1271070723/1276674334.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline](attachments/1271070723/1276543334)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline.png](attachments/1271070723/1276150103.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline](attachments/1271070723/1276805454)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline.png](attachments/1271070723/1276477832.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276871072.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276576167.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276510646.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276543374.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276871082.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276740050.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276182909.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276805444.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276576177.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297645573.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline](attachments/1271070723/1298432037)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline.png](attachments/1271070723/1298268213.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297940481.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298071553.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298137089.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297743885.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297743895.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298006021.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297743909.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298235393.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298169861.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297678355.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298202633.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298333697.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298104325.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298006039.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297973257.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297743931.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297907733.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298300937.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297809425.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297645588.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298038797.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298235403.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298333707.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297940511.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297809439.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297612832.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297743945.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297743955.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298432013.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298399245.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297678369.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298006053.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297842197.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298300947.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298300957.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298300967.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297645606.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298006067.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298268185.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1297678379.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298268195.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298071575.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298137123.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298432027.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1298366485.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Curated Hyper Pipeline.tmp](attachments/1271070723/1276575893.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline](attachments/1271070723/1276346511)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Curated
Hyper Pipeline.png](attachments/1271070723/1276477555.png) (image/png)  

</div>
