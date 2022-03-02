# ADF Hyper Pipeline - Event Cancellation Insurance

## Background

Due to quick nature of creating event insurance and the lengthy process
of creating policies in client 360 it was decided by the claims team to
record these policies in a excel spreadsheet. Later on managing the
excel spreadsheet and generating reports from it was becoming increasing
difficult and insights was approached to help.

## Key Activities

The key activities of this pipeline are:

1.  To clean up the current excel sheet to provide a quality set of data
    to ingest including:

2.  To create a pipeline to import the required sheets from excel into
    SDI/snowflake

3.  To create reports in tableau from the data in snowflake

## Clean up the excel sheet

The excel sheet was cleaned up in the following ways:

1.  Formula fields have been protected/locked from edits. It is still
    possible to highlight and format (colour) the fields, but it is no
    longer possible to edit the formula itself to ensure the data is not
    overridden. 

2.  The column names have been changed to match a more database-friendly
    format for data import. 

3.  Left-aligned many fields for consistency.

4.  Ensured that date fields appear the same way - we might need a
    follow up to ensure date fields don't error out. 

5.  Fixed up naming in the ENTERED_BY, CHECKED_BY, DELEGATION_AUTHORITY
    fields (removed trailing spaces, capitalised first and last names,
    corrected spelling mistakes).

6.  Removed NA/Error's from appearing in formula fields (e.g. blank rows
    showed ERROR or #N/A, that has been removed and blanked out for
    consistency). 

## Pipeline

### High Level Design

Original the plan was to import the excel into Azure Data Factory
directly using a sharepoint connector. However due to time constraints
it was decided that using the exisitng risk console pipeline and a
python script running on the on-prem VM to copy the excel sheet from
sharepoint to the K drive.  

Multiple sheets of the excel file need to be ingested and their is a
sheet parameter introduced in the pipeline for this purpose.  

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio8241367737666999635"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio8241367737666999635"
class="ap-content">

</div>

</div>

### Detailed Design

#### Steps

1.  Parameter parser - Azure function to parse and process the
    parameters passed into the ADF pipeline trigger

2.  Acquisition layer

    1.  Check to see if the file is already in landing and if so skip
        part b

    2.  The linked service/dataset connects to the network (K:/) drive
        to check the file exists and if so places it in the
        landingcontainer of the data lake

    3.  The file is copied to the raw in parquet format.

3.  event_last_pipeline_status_query - use the existing function
    last_pipeline_status_query to check the previous day’s execution for
    the geospatial hyper pipeline was successful or not. Fail the pipe
    if not

4.  Structured layer - Same as Risk console

5.  Curated Layer - Same as Risk console

### Load Cadence

All sheets are loaded daily.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323139073.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323171841.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323171850.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323204609.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323237377.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323106311.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323270145.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323270154.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323302913.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323335681.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323368449.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323040776.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323270163.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323401217.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323139082.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323433985.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323466753.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323302922.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323237386.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323139091.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323499521.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323335690.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323335699.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323532289.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323565057.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323106320.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323597825.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323335708.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323008008.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~6109e891b704b40068042fae\~Untitled
Diagram.drawio.tmp](attachments/1322418177/1323106305.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.drawio](attachments/1322418177/1323335717.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.drawio.png](attachments/1322418177/1323368458.png)
(image/png)  

</div>
