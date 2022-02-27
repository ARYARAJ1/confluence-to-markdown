# ADF Hyper Pipeline - DB Pull (Client Connect/IDR2)

This section contains the build summary of ADF Hyper Pipeline for Client
Connect.

At a high level, both ADF Hyper Pipeline and sub-pipeline (Acquisition,
Structured and Curated) will be provisioned via `pulumi `and ARM
template. The pipeline will constrains the pre-defined connections,
activities, datasets and **placeholders** for **runtime parameters.**

<div class="toc-macro rbtoc1645673422619">

-   1 [Hyper Pipeline
    Design](#ADFHyperPipeline-DBPull(ClientConnect/IDR2)-HyperPipelineDesign)
    -   1.1 [Trigger workflow
        description:](#ADFHyperPipeline-DBPull(ClientConnect/IDR2)-Triggerworkflowdescription:)
    -   1.2 [Pipeline Workflow
        description:](#ADFHyperPipeline-DBPull(ClientConnect/IDR2)-PipelineWorkflowdescription:)
-   2 [Checkpoint Parameter and Decision
    Rules](#ADFHyperPipeline-DBPull(ClientConnect/IDR2)-CheckpointParameterandDecisionRules)

</div>

# Hyper Pipeline Design

Below is a diagram to illustrate the activities and functions that has
been scheduled in the pipeline.

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio318278795770804198"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio318278795770804198"
class="ap-content">

</div>

</div>

### **Trigger workflow description:**

**1.**A trigger will be waken up at the scheduled time, query Run-time
config from the cosmosDB, and pass in the corresponding global
parameters to the pipeline. All triggers have been created based on the
build-time config during the deployment phase.

**2.** The trigger triggers the hyper pipeline with prepared parameter
values. 

### **Pipeline Workflow description:**

**1a.** A parameter parsing function will be triggered, consume the
trigger parameters and create the required parameters for consumption of
the daily file.

**1b.** A linked service from ADF enables ADF to access Client Connect
SQL server and perform a full load.

**1c.** Via a COPY activity, grab all records and columns using SELECT
\* from a table and convert into a parquet file. ADF will automatically
identify and apply the SQL server table schema (i.e. data types and
columns) to the parquet file. Since Client Connect is hosted in a
private network, the self-hosted ADF integration runtime VM is used. If
the VM fails to perform the COPY activity (e.g. missing Java JER as
dependency), we will lose the snapshot of a given period. If the data
from a period is critical, we can restore a SQL server snapshot to a new
database and manually process it. This would recommend the scheduled
trigger time to be close to the snapshot taken time so that the snapshot
would provide what data were supposed to be acquired mostly.

**2.** If the acquisition pipeline finishes successfully, it will
trigger the checkpoint function. Failures of the structured or curated
pipelines will not prevent the execution of the acquisition pipeline for
the next pipeline execution. this design will reduce the likelyhood that
an acquisition snapshot is not captured. **In essence everytime the
pipeline executes it copies data to raw before inferring whether to
proceed**

**3.** The checkpoint function (Azure Function) will query pipeline and
activity status from Log Analytics Workspace using <a
href="https://docs.microsoft.com/en-us/azure/data-explorer/kusto/concepts/"
rel="nofollow">Kusto</a> (see <a
href="https://medium.com/@IrekRomaniuk/query-azure-log-analytics-from-python-f282151b7d46"
rel="nofollow">how to use Python query Log Analytics</a>.). Since Log
Analytics doesn’t support AD token authentication, we will need to
create a service principal and authenticate Azure Function with Log
Analytics using a secret. Log Analytics can access both ADF and Azure
Function’s logs (for the latter, need to access app insights via Log
Analytics Workspace). Some example code (<a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/285999125/ADF+Logging"
data-linked-resource-id="285999125" data-linked-resource-version="2"
data-linked-resource-type="page">ADF Logging</a> contains example of how
to query app insights log using Kusto from Log Analytics Workspace):

**4.** The checkpoint function reads in the pipeline dependency
conditions from a config stored in Cosmos, and apply to the pipeline and
activity status.

**5a.** If all conditions are met, continue to the structured pipeline.

**5b.** If fail to met condition(s), cease the execution of the hyper
pipeline and make pipeline failed.

**6a, 6b, 6c** are the <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/370638849/Technical+Data+Quality+%28TDQ%29+Check+-+Non-PEGA+data+source"
data-linked-resource-id="370638849" data-linked-resource-version="46"
data-linked-resource-type="page">Technical Data Quality (TDQ) Check -
Non-PEGA data source</a>. The parquet files will be loaded into the
structured layer using the COPY command of snowflake if all the data
quality checks pass successfully.

**7.** If TDQ failed, trigger the move activity in the structure
pipeline.

**8.** If the structured pipeline (i.e. TDQ checker) finishes
successfully, it will trigger the curated structure.

**9a** and **9b** refer to <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/432144521/Time+Varying+Data+Load"
data-linked-resource-id="432144521" data-linked-resource-version="5"
data-linked-resource-type="page">Time Varying Data Load</a> .

# Checkpoint Parameter and Decision Rules

Same as stated in
<a href="ADF_Hyper_Pipeline_-_File_Server_Risk_Console_"
data-linked-resource-id="424116229" data-linked-resource-version="23"
data-linked-resource-type="page">ADF Hyper Pipeline - File Server (Risk
Console)</a>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio.png](attachments/437190686/437485682.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio](attachments/437190686/437354598.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-023209.png](attachments/437190686/437190717.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-022846.png](attachments/437190686/437190720.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-020659.png](attachments/437190686/437190723.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-015745.png](attachments/437190686/437190726.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-015740.png](attachments/437190686/437190729.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-002120.png](attachments/437190686/437190732.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-002005.png](attachments/437190686/437190735.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-001500.png](attachments/437190686/437190738.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-001130.png](attachments/437190686/437190741.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-000348.png](attachments/437190686/437190744.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201124-000234.png](attachments/437190686/437190747.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201123-233202.png](attachments/437190686/437190750.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201123-232302.png](attachments/437190686/437190753.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201123-230456.png](attachments/437190686/437190756.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f3210f19aa96500466e388b\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/437190759.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio](attachments/437190686/440238267.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio.png](attachments/437190686/440533105.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio](attachments/437190686/440500430.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio.png](attachments/437190686/437354806.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio](attachments/437190686/462815739.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio.png](attachments/437190686/462914040.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462651785.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462881412.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462913988.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462651803.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462815715.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462651817.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462815725.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462914026.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/462881454.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~hyperpipeline_Diagram.drawio.tmp](attachments/437190686/437190714.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio](attachments/437190686/683114621.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio.png](attachments/437190686/679968894.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio](attachments/437190686/437190711.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[hyperpipeline_Diagram.drawio.png](attachments/437190686/437190708.png)
(image/png)  

</div>
