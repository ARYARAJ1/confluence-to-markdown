# Data Warehouse Pipeline - 1. Execute Snowflake Query

<div class="contentLayout2">

<div class="columnLayout fixed-width" layout="fixed-width">

<div class="cell normal" data-type="normal">

<div class="innerCell">

# Introduction

As part of the data warehouse pipeline design, we need to decide
**where** and **how** SQL scripts used to load data into the data
warehouse are going to be executed. The following options explore using
various Azure services to run the scripts either as a stored procedure
or plain SQL string inside Snowflake, or process data in a cluster
outside of Snowflake.

<div class="toc-macro rbtoc1639539209516">

-   1
    [Introduction](#DataWarehousePipeline-1.ExecuteSnowflakeQuery-Introduction)
-   2 [Options of Executing SQL Scripts in
    Snowflake](#DataWarehousePipeline-1.ExecuteSnowflakeQuery-OptionsofExecutingSQLScriptsinSnowflake)
    -   2.1 [ADF Pipeline Lookup Activity Triggers a Stored Procedure in
        Snowflake](#DataWarehousePipeline-1.ExecuteSnowflakeQuery-ADFPipelineLookupActivityTriggersaStoredProcedureinSnowflake)
    -   2.2 [ADF Data Flow Integration with
        Snowflake](#DataWarehousePipeline-1.ExecuteSnowflakeQuery-ADFDataFlowIntegrationwithSnowflake)
    -   2.3 [ADF custom activity + Azure batch + Python application to
        execute SQL scripts stored in
        config](#DataWarehousePipeline-1.ExecuteSnowflakeQuery-ADFcustomactivity+Azurebatch+PythonapplicationtoexecuteSQLscriptsstoredinconfig)
-   3
    [Decision](#DataWarehousePipeline-1.ExecuteSnowflakeQuery-Decision)
-   4 [Meeting to review decision on Mapping dataflows vs stored
    procedures](#DataWarehousePipeline-1.ExecuteSnowflakeQuery-MeetingtoreviewdecisiononMappingdataflowsvsstoredprocedures)

</div>

# Options of Executing SQL Scripts in Snowflake

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>ADF lookup activity triggers the
execution of a stored procedure in Snowflake</strong> DECIDED</p></th>
<th class="confluenceTh"><p><strong>Azure Function / Durable Function
execute SQL scripts stored in config</strong></p></th>
<th class="confluenceTh"><p><strong>ADF data flow with Snowflake linked
service</strong></p></th>
<th class="confluenceTh"><p><strong>ADF custom activity + Azure batch +
Python application to execute SQL scripts stored in
config</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>Description</p></td>
<td class="confluenceTd"><p>The <a
href="https://docs.microsoft.com/en-us/azure/data-factory/control-flow-lookup-activity"
rel="nofollow">Lookup activity in ADF supports ADF to trigger a store
procedure in Snowflake</a>. In the documentation, how to achieve this is
vague. A POC has been created to confirm this function (see <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/440271104/Data+Warehouse+Pipeline+-+1.+Execute+Snowflake+Query#ADF-Pipeline-Lookup-Activity-Triggers-a-Stored-Procedure-in-Snowflake"
rel="nofollow"><strong>ADF Pipeline Lookup Activity Triggers a Stored
Procedure in Snowflake</strong></a> for detail).</p>
<p><strong>Note</strong>: <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/263421967/Azure+Data+Factory+-+Build"
rel="nofollow">COPY activity in ADF supports stored procedure for SQL
Server, Oracle, etc. using sqlReaderStoredProcedureName or
sqlReaderQuery but doesn’t support Snowflake</a>.</p></td>
<td class="confluenceTd"><p>Azure function or durable function can
be</p>
<ol>
<li><p>used to retrieve SQL scripts from a config file stored in Azure
Storage Account or from a JSON object stored in Azure CosmosDB,</p></li>
<li><p>pass in dynamic parameters values, and</p></li>
<li><p>use Python connector to execute scripts in Snowflake.</p></li>
</ol>
<p>For an event-driven architecture, Azure Function can be used to
trigger the execution of SQL scripts without waiting for the outcome of
the execution, considering the 4 minutes hard timeout limit
(<strong>Note</strong>: Snowflake currently cannot emit a notification
or event to Azure. The notification integration is for Snowflake to
consume an event from Azure queue storage. See more detail in <a
href="Data_Warehouse_Pipeline_-_2._Orchestration.md"
data-linked-resource-id="460554258" data-linked-resource-version="16"
data-linked-resource-type="page">Data Warehouse Pipeline - 2.
Orchestration</a> <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/460554258/Data+Warehouse+Pipeline+-+2.+Orchestration#Exploration-on-Event-driven-Architecture"
rel="nofollow"><strong>event-driven architecture section</strong></a>).
Otherwise, Durable Function can be utilised to trigger and wait for the
outcome (i.e. SQL script successfully finishes or not).</p>
<p>For ADF to work with a Durable Function, it requires an <strong>Azure
Function to kick off the Durable Function in an asynchronous
way</strong> (don’t wait for the execution outcome) and receive
<code>statusQueryGetUri</code> which contains the Durable Function’s
status changes. Then use a <strong>Web Activity</strong> for ADF to poll
the status of the Durable Function until status code 200 (complete
successfully) appears. An example pipeline looks like below in ADP: See
<a
href="https://endjin.com/blog/2019/09/azure-data-factory-long-running-functions#:~:text=Durable%20Functions%20enable%20us%20to,that%20return%20202%20status%20codes."
rel="nofollow">more details</a> and <a
href="https://docs.microsoft.com/en-us/azure/data-factory/control-flow-azure-function-activity"
rel="nofollow">documentation</a> here.</p>
<img src="attachments/440271104/458096780.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/458096780.png"
data-height="250" data-width="802" data-unresolved-comment-count="0"
data-linked-resource-id="458096780" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201202-015649.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="08990aca-db11-4d43-9ff2-6ed785705673"
data-media-type="file" /></td>
<td class="confluenceTd"><p>ADF data flow can integrate with Snowflake
with a linked service to retrieve data from and write data to Snowflake.
ADF data flow is built and operated on a spark cluster starting on when
a data flow runs. The intended is to consume data from the source into
the spark cluster, process and transform data in memory, and write out
to a destination source. Broadcasting small data frames to all nodes in
a cluster is recommended as one way to optimise performance. Some
pre-configuration can be set up in the integration runtime (i.e. cluster
type). As a managed service, it charges based on cluster size and
execution time.</p>
<p>See the subsection “<a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/440271104/Data+Warehouse+Pipeline+-+1.+Execute+Snowflake+Query#ADF-Data-Flow-Integration-with-Snowflake"
rel="nofollow"><strong>ADF Data Flow Integration with
Snowflake</strong></a>“ for more detail.</p></td>
<td class="confluenceTd"><p>ADF custom activity a bespoke application.
It cannot run on the ADF integration runtime and therefore will need to
use Azure Batch. Different from Azure Function, ADF custom activity is
<a
href="https://mrpaulandrew.com/2018/11/12/creating-an-azure-data-factory-v2-custom-activity/"
rel="nofollow">long-running and can scale up compute nodes if
needed</a>. See the detail in <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/440271104/Data+Warehouse+Pipeline+-+1.+Execute+Snowflake+Query#ADF-custom-activity-%2B-Azure-batch-%2B-Python-application-to-execute-SQL-scripts-stored-in-config"
rel="nofollow"><strong>ADF custom activity + Azure batch + Python
application to execute SQL scripts stored in config</strong></a> section
on how to set up.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Advantages</p></td>
<td class="confluenceTd"><ul>
<li><p>No need to use another long-running service to trigger a
Snowflake stored procedure, and therefore simplify the solution. No new
service is introduced.</p></li>
<li><p>Monitor pipeline status in one place as the error message can be
propagated to the log in ADF. The stored procedure ID and execution
outcome (e.g. number of row inserted) can be <a
href="https://docs.snowflake.com/en/sql-reference/stored-procedures-api.html#object-statement"
rel="nofollow">extracted from resultSet object</a>.</p></li>
<li><p>Heavy-lifting (compute intensive operations) is done on the
Snowflake side.</p></li>
<li><p>A stored procedure can be parameterised (see <a
href="https://docs.snowflake.com/en/sql-reference/sql/create-procedure.html"
rel="nofollow">examples</a>). Returned value can be retrieved by the
subsequent activities using <strong>@json(activity('&lt;lookup activity
name&gt;').output.firstRow.&lt;stored procedure name in upper
case&gt;).&lt;key name in the returned object if
any&gt;</strong></p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Monitor pipeline status in one place.</p></li>
<li><p>Flexible in manipulating SQL scripts.</p></li>
<li><p>Can retrieve SQL script execution output from Snowflake during
runtime.</p></li>
<li><p>SQL script testing can be easily incorporated as part of Python
script testing.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Native integration between ADF and Data Flow.</p></li>
<li><p>Leverage configuration to minimise coding and is more user
friendly.</p></li>
<li><p>No timeout concern (max seven days hard limit for pipeline
activity runs).</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Monitor pipeline status in one place.</p></li>
<li><p>Flexible in manipulating SQL scripts.</p></li>
<li><p>Can retrieve SQL script execution output from Snowflake during
runtime.</p></li>
<li><p>Support long-running process compared with Azure Function and is
easier to setup compared to Durable Function.</p></li>
<li><p>SQL script testing can be easily incorporated as part of Python
script testing.</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Disadvantages</p></td>
<td class="confluenceTd"><ul>
<li><p>Snowflake <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/40927337"
rel="nofollow">stored procedure is written in JavaScript</a> which can
execute SQL statements by calling a JavaScript API. This design
introduces a new requirement in VMIA’s capability for on-going
maintenance and development.</p></li>
<li><p>Snowflake stored procedure is similar to how Python connector
execute a script (see <a
href="https://docs.snowflake.com/en/sql-reference/sql/create-procedure.html"
rel="nofollow">store procedure example</a>). Each query execution can
only run one statement.</p></li>
<li><p>Additional work to deploy/pre-create store procedures in
Snowflake.</p></li>
<li><p>Additional work to design and test Snowflake stored
procedure.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Durable Function is a new service to add to the solution, and it
can be quite complicated. The minimum components for a durable function
include an orchestrator function, an activity function, and a client
function as a trigger to a durable function.</p></li>
<li><p>Config stored in the Storage Account or CosmosDB (particularly
the latter) can be easily altered by people and can cause inconsistency
between the expected and the actual scripts.</p></li>
<li><p>The setup for ADF to interact with Durable Function is quite
clunky.</p></li>
<li><p>The solution using Azure Function is pending on the exploration
of the event-driven capability of Snowflake.</p></li>
<li><p>SQL scripts stored in a JSON object can be hard to read as
scripts have to be in one line.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>ADF data flow essentially is a spark application, running on a
spark cluster. Therefore, t<a
href="https://docs.microsoft.com/en-us/azure/data-factory/concepts-data-flow-performance"
rel="nofollow">his incurs a 3-5min start-up time. For sequential jobs,
this can be reduced by enabling time to live value (soft limit max 4
hours).</a> Each data flow runs on a separate cluster, which is
potentially quite expensive</p></li>
<li><p>Data will be sucked out into Spark for processing rather than let
Snowflake do the heavy lifting job.</p></li>
<li><p>Have to finish with a sink source or validation fails.</p></li>
<li><p>The Query field cannot contain “into” or “update” or multiple
“select” statements. So far, only one single “select” statement has
successfully run.</p></li>
<li><p><a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/401605809/Azure+Logic+Apps+Use+Cases+and+Capability+Exploration"
rel="nofollow">The concurrent number of data flow per factory has a soft
limit 50</a>.</p></li>
<li><p>In general it takes about 4+min to start a spark cluster which a
data flow is running upon. In the experiment where two tables with 4
columns and 5 records each, it takes 7+ min to run the SCD, while doing
in a stored procedure is about less than a minute.</p></li>
<li><p>If the data flow uses runtime dynamic values, Preview function
won’t work (as there is no static values to feed in), and debug and
development experience can be lengthy and inconvenient (need to
temporarily sink to a table).</p></li>
<li><p>When having multiple data flows running in parallel, multiple
spark clusters will need to start, which can be costly.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>More complicated than running ADF lookup activity with a
snowflake stored procedure.</p></li>
<li><p>Additional deployment to put Python scripts in Azure Storage
Account.</p></li>
<li><p>Values that want to be passed out of the function need to be
written into <code>outputs.json</code>. This will show up in ADF as
activity output and can be accessed via
<code>@activity('&lt;MyCustomActivity&gt;').output.customOutput</code>.
Errors will go to <strong>stderr.txt</strong> accessible from Batch
GUI.</p></li>
<li><p>Any print-outs will go to <a
href="https://docs.microsoft.com/en-us/azure/data-factory/transform-data-using-dotnet-custom-activity"
rel="nofollow"><strong>stdout.txt</strong></a>. Python logging and
return are not supported.</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

d

## ADF Pipeline Lookup Activity Triggers a Stored Procedure in Snowflake

ADF Lookup activity supports triggering a pre-created stored procedure
and executes it within Snowflake. The configuration is as below. The
**Source dataset** can just a linked service without specifying any
table because any target tables can be included in the stored procedure.
The **Call** statement is entered in the **Query** field. The example
shows that the **Query** is a string and can be parameterised

<img src="attachments/440271104/472810583.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/472810583.png"
data-height="692" data-width="1178" data-unresolved-comment-count="0"
data-linked-resource-id="472810583" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201209-045802.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="7ce2a5bd-e2c0-4a2c-9269-a1074623adcb"
data-media-type="file" />

Source dataset requires for the Snowflake linked service (see **ADF Data
Flow Integration with Snowflake** step 1 for configuration detail). No
need to specify a table.

<img src="attachments/440271104/455245907.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455245907.png"
data-height="642" data-width="1019" data-unresolved-comment-count="0"
data-linked-resource-id="455245907" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201201-224434.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="f19504b3-26e6-432f-b2e8-608d2724be06"
data-media-type="file" />

A subsequent activity consumes the output of the Lookup activity.

<img src="attachments/440271104/472613862.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/472613862.png"
data-height="704" data-width="934" data-unresolved-comment-count="0"
data-linked-resource-id="472613862" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201209-021016.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="6cdb36ca-7852-4d85-9117-ac2532b35c00"
data-media-type="file" />

A simple example of stored procedure in Snowflake:

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeContent panelContent pdl">

``` sql
use database "SF_TEST";
create or replace procedure sp_sf_test(exp_row_count varchar)
    returns string not null
    language javascript
    as
    $$
    
    var sql_string = "insert into \"RISK_CONSOLE\".\"TEST_2\" select * from \"RISK_CONSOLE\".\"LANDING_ALLASSETS_EN\" limit 5;";
    var sql_statement = snowflake.createStatement( {sqlText: sql_string} );
    var result_set = sql_statement.execute();
    
    var sf_query_id = result_set.getQueryId();
    var sf_query_output_key = result_set.getColumnName(1);
    
    result_set.next();
    var sf_query_output_value = result_set.getColumnValue(1);
    
    if (sf_query_output_value == parseInt(EXP_ROW_COUNT)) {
        return JSON.stringify({"query_id": sf_query_id, "status": "succeeded", "file_name": "test_1.txt"})

    } else {
        return JSON.stringify({"query_id": sf_query_id, "status": "failed"})
    }    
    

    $$
    ;
```

</div>

</div>

Regardless what data type of return from a stored procedure,

-   return string, see the above example stored procedure

-   return JSON object,

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeContent panelContent pdl">

``` java
# in line 3 as in the example above
  returns variant not null

# in line 19
  return {"query_id": sf_query_id, "status": "succeeded", "file_name": "test_1.txt"}
  
# in like 22
  return {"query_id": sf_query_id, "status": "failed"}
```

</div>

</div>

The output displayed by the Lookup Acivity on the ADF side is always in
string:

</div>

</div>

</div>

<div class="columnLayout two-equal" layout="two-equal">

<div class="cell normal" data-type="normal">

<div class="innerCell">

When a Snowflake stored procedure return string:

<img src="attachments/440271104/577635272.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/577635272.png"
data-height="274" data-width="510" data-unresolved-comment-count="0"
data-linked-resource-id="577635272" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20210119-045503.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="7d513e59-7926-4935-a4d9-097c5ee76607"
data-media-type="file" />

</div>

</div>

<div class="cell normal" data-type="normal">

<div class="innerCell">

When a Snowflake stored procedure return object:

<img src="attachments/440271104/574948486.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/574948486.png"
data-height="271" data-width="548" data-unresolved-comment-count="0"
data-linked-resource-id="574948486" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20210119-045645.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="443cf980-700b-43af-ad01-a3df948fbb43"
data-media-type="file" />

</div>

</div>

</div>

<div class="columnLayout fixed-width" layout="fixed-width">

<div class="cell normal" data-type="normal">

<div class="innerCell">

Therefore, when want to refer to any key within the object string, we
need to convert the string into JSON first:

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeContent panelContent pdl">

``` java
# refer to the value of key = status
@json(activity('run_snowflake_sp').output.firstRow.SP_SF_TEST).status
```

</div>

</div>

## ADF Data Flow Integration with Snowflake

**Step 1**: create a Snowflake linked service

<img src="attachments/440271104/455278603.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455278603.png"
data-height="710" data-width="571" data-unresolved-comment-count="0"
data-linked-resource-id="455278603" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201201-010130.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="29eb0dc2-3e16-43eb-85ac-140908886fbe"
data-media-type="file" />

**Step 2:** create a new data flow. Add Snowflake as a Source.

<img src="attachments/440271104/455376914.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455376914.png"
data-height="720" data-width="1048" data-unresolved-comment-count="0"
data-linked-resource-id="455376914" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201201-010014.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="511f3b2c-40f2-4b41-8a5c-13c7cffa5522"
data-media-type="file" />

When defining the source in Source Options, can choose either use table
or query. There are limitations in what query can run. So far, I only
find a single “SELECT” statement is allowed. Have tried “Insert into”
and “UPDATE”, and both of them throw an error. Based on the Input
options, the query intends to provide some flexibility (i.e. filtering,
subletting) when retrieving data into ADF.

<img src="attachments/440271104/455245829.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455245829.png"
data-height="721" data-width="826" data-unresolved-comment-count="0"
data-linked-resource-id="455245829" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201201-010314.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="7965b1f5-d304-476c-9c75-a1429264117f"
data-media-type="file" />

**Step 3**: a data flow can only be operationalised in a pipeline.
Create a new data pipeline and drag the data flow into it. Trigger the
data pipeline to run the data flow.

<img src="attachments/440271104/455376922.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455376922.png"
data-height="699" data-width="1047" data-unresolved-comment-count="0"
data-linked-resource-id="455376922" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201201-010701.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="0fccf8c0-7f3d-48fe-8aa1-d78aff9841e1"
data-media-type="file" />

**Step 4**: go to ADF Monitor for log. Because data flow runs on a Spark
cluster, ADF provides additional information to show the running
durations of different components.

<img src="attachments/440271104/455147525.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455147525.png"
data-height="510" data-width="1250" data-unresolved-comment-count="0"
data-linked-resource-id="455147525" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201130-231116.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="36e0bf99-c4ff-47ad-9a64-a4a60d752f8a"
data-media-type="file" />

The additional information is available when drill down to activity
level. This view provides information on cluster startup time,
processing time, and a breakdown of each process.

<img src="attachments/440271104/455278597.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455278597.png"
data-height="566" data-width="1497" data-unresolved-comment-count="0"
data-linked-resource-id="455278597" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201130-231006.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="ab340ccf-245a-40ba-a49b-b18df67fa3d7"
data-media-type="file" />

This diagram provides an example on what information is available for
activity level monitoring. See <a
href="https://docs.microsoft.com/en-us/azure/data-factory/concepts-data-flow-performance"
rel="nofollow">how to optimise data flow performance here</a>.

<img src="attachments/440271104/455245837.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/455245837.png"
data-height="1129" data-width="1648" data-unresolved-comment-count="0"
data-linked-resource-id="455245837" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201201-011150.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="b7d86de0-9a73-4cfb-a0de-4e6b6528b97c"
data-media-type="file" />

It is possible to create a generic config-driven slow change dimension
(SCD) using data flow and parameters. The parameters are data flow
specific and are created after dragging the data flow into a pipeline as
an activity (see the screenshot below). Because transformation steps
within a data flow cannot refer to activity output in a pipeline, config
relies on the data flow parameters to carry over any activity output.

<img src="attachments/440271104/490307659.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/490307659.png"
data-height="659" data-width="1167" data-unresolved-comment-count="0"
data-linked-resource-id="490307659" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201215-055414.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="bd64fa15-fcfc-426b-ac5b-940b456db133"
data-media-type="file" />

A generic data flow for SCD can be constructed as below. Please refer to
<a href="https://www.youtube.com/watch?v=tc283k8CWh8"
rel="nofollow">this video</a> for details within each step.

<img src="attachments/440271104/487784571.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/487784571.png"
data-height="625" data-width="1433" data-unresolved-comment-count="0"
data-linked-resource-id="487784571" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201215-055618.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="e4a1795c-3d3c-4b83-9c50-bb15bb0754cf"
data-media-type="file" />

## **ADF custom activity + Azure batch + Python application to execute SQL scripts stored in config**

For detail setup, refer to this <a
href="https://medium.com/@ashish.kats/custom-batch-activity-in-azure-data-factory-110d8c1c957b"
rel="nofollow">link</a>. The custom code written in Python needs to be
firstly uploaded to the Azure Storage Account where Azure Batch linked
service knows where to retrieve the .py file. In the Command field, put
in the execution string. A downstream Copy activity is created to link
to the custom activity to test accessing and parsing output from the
custom activity.

<img src="attachments/440271104/475299872.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/475299872.png"
data-height="690" data-width="1177" data-unresolved-comment-count="0"
data-linked-resource-id="475299872" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201209-051341.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="ad5593fd-46fe-4c65-b0c3-dbcbdad6b5b5"
data-media-type="file" />

In the .py file, to produce activity output, the code needs to write out
the content into **outputs.json**.

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeContent panelContent pdl">

``` java
import logging

logger = logging.getLogger(__name__)

def main():
    out_dict = {}
    out_dict["my_key"] = "test_1.txt"
    logger.info('---------- use logger')
    print('----------- use print')
    with open('outputs.json','w') as file:
        file.write(str(out_dict))

    return '------- values from return'

if __name__ == "__main__":
    main()
```

</div>

</div>

The content of custom activity output is visible in ADF log.

<img src="attachments/440271104/472810595.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/472810595.png"
data-height="630" data-width="1483" data-unresolved-comment-count="0"
data-linked-resource-id="472810595" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201209-052100.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="1659b289-371b-4b97-96c5-1b8c8e0aa872"
data-media-type="file" />

In the above screenshot, the highlighted id string can be used to
identify the batch job in Azure Batch log where you can access addition
information, e.g. as logs in stdout.txt, and error message in
stderr.txt. Those information is **NOT** propagated to ADF. If an error
happens, in ADF log, only a **generic error message** will return:

Operation on target run_script failed: Hit unexpected exception, please
retry. If the problem persists, please contact Azure Support".

In the stdout.txt after running the above python code, only **print**
message is captured, logger and returned message are not.

<img src="attachments/440271104/475168806.png" class="image-center"
loading="lazy" data-image-src="attachments/440271104/475168806.png"
data-height="582" data-width="1536" data-unresolved-comment-count="0"
data-linked-resource-id="475168806" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20201209-052424.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="440271104"
data-linked-resource-container-version="32"
data-media-id="cb7459bc-4739-4c60-80e2-32e7bb01fbb4"
data-media-type="file" />

# Decision

The decision is made based on four principles:

<div class="table-wrap">

|                                                                        |                                                                                                                                                                                      |                                                                                                                                                                                                            |                                                                                                                                                                                                                                                                                                                                                                                           |                                                                                                                                                                                                             |
|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Principles**                                                         | **ADF lookup activity triggers the execution of a stored procedure in Snowflake** DECIDED                                                                                            | **Azure Function / Durable Function execute SQL scripts stored in config**                                                                                                                                 | **ADF data flow with Snowflake linked service**                                                                                                                                                                                                                                                                                                                                           | **ADF custom activity + Azure batch + Python application to execute SQL scripts stored in config**                                                                                                          |
| Simple solution is better solution.                                    | <img src="images/icons/emoticons/check.png"                                                                                                                                          
                                                                          class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"                                                                                                                   
                                                                          data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"                                                                                                                
                                                                          data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> SQL scripts are written as Snowflake stored procedure. A simple activity in ADF pipeline to trigger and get status.  | <img src="images/icons/emoticons/error.png"                                                                                                                                                                
                                                                                                                                                                                                                                                                 class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                        
                                                                                                                                                                                                                                                                 data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                      
                                                                                                                                                                                                                                                                 data-emoticon-name="cross" width="16" height="16" alt="(error)" /> Complexity of Durable Function. Use Azure function to trigger and a web service activity to poll execution status.                       | <img src="images/icons/emoticons/error.png"                                                                                                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoticon-name="cross" width="16" height="16" alt="(error)" /> Use functions in the data flow to build out data warehouse loading process.                                                                                                                                                                                                                                             | <img src="images/icons/emoticons/check.png"                                                                                                                                                                 
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"                                                                                                                                          
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> Use Custom Activity to integrate with Azure Batch. The script running on Batch provides flexibility and control on the SQL scripts to run.  |
| Minimise the introduction of new services unless necessary.            | <img src="images/icons/emoticons/check.png"                                                                                                                                          
                                                                          class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"                                                                                                                   
                                                                          data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"                                                                                                                
                                                                          data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> No new service needed.                                                                                               | <img src="images/icons/emoticons/error.png"                                                                                                                                                                
                                                                                                                                                                                                                                                                 class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                        
                                                                                                                                                                                                                                                                 data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                      
                                                                                                                                                                                                                                                                 data-emoticon-name="cross" width="16" height="16" alt="(error)" /> Durable Function is a new service to the existing solution.                                                                              | <img src="images/icons/emoticons/error.png"                                                                                                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoticon-name="cross" width="16" height="16" alt="(error)" /> ADF data flow is a new service to the existing solution.                                                                                                                                                                                                                                                                | <img src="images/icons/emoticons/error.png"                                                                                                                                                                 
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                         
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoticon-name="cross" width="16" height="16" alt="(error)" /> Azure Batch is a new service to the existing solution.                                                                                    |
| It has to be easy to use for on-going maintenance and operation.       | <img src="images/icons/emoticons/check.png"                                                                                                                                          
                                                                          class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"                                                                                                                   
                                                                          data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"                                                                                                                
                                                                          data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> ADF Monitor contains all the logs, including errors from the Snowflake side.                                         | <img src="images/icons/emoticons/error.png"                                                                                                                                                                
                                                                                                                                                                                                                                                                 class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                        
                                                                                                                                                                                                                                                                 data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                      
                                                                                                                                                                                                                                                                 data-emoticon-name="cross" width="16" height="16" alt="(error)" /> A data engineer needs to check both ADF Monitor and Durable Function logs. All the logs can be accessible from Log Analytics workspace.  | <img src="images/icons/emoticons/error.png"                                                                                                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoticon-name="cross" width="16" height="16" alt="(error)" /> ADF Monitor contains all the logs, including errors from the Snowflake side. However, the debug time can be long due to the overhead time for a spark cluster to start and run. If a data flow is driven by config, dynamic values prohibit using Preview function, making debugging and checking output inconvenient.  | <img src="images/icons/emoticons/error.png"                                                                                                                                                                 
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                         
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoticon-name="cross" width="16" height="16" alt="(error)" /> The logging and error messages of scripts running on Batch can only be accessible from Azure Batch’s job section.                         |
| Let a service to do it best at and leverage the built-in optimisation. | <img src="images/icons/emoticons/check.png"                                                                                                                                          
                                                                          class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"                                                                                                                   
                                                                          data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"                                                                                                                
                                                                          data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> Fully leverage Snowflake’s built-in optimisation and data processing power.                                          | <img src="images/icons/emoticons/check.png"                                                                                                                                                                
                                                                                                                                                                                                                                                                 class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"                                                                                                                                         
                                                                                                                                                                                                                                                                 data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"                                                                                                                                      
                                                                                                                                                                                                                                                                 data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> Fully leverage Snowflake’s built-in optimisation and data processing power.                                                                | <img src="images/icons/emoticons/error.png"                                                                                                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"                                                                                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"                                                                                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                                                                                                                                                                                                              data-emoticon-name="cross" width="16" height="16" alt="(error)" /> Execution happens in memory of the spark cluster.                                                                                                                                                                                                                                                                       | <img src="images/icons/emoticons/check.png"                                                                                                                                                                 
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"                                                                                                                                          
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"                                                                                                                                       
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> Fully leverage Snowflake’s built-in optimisation and data processing power.                                                                 |

</div>

# Meeting to review decision on Mapping dataflows vs stored procedures

Attendees:

Lasath Kahingala

Bo Kwon (Deactivated)

Rahul Pandey (Deactivated)

Sarah Fan (Deactivated)

h.lindsaysmith@vmia.vic.gov.au Lindsay-Smith

Date: 14 Jan 2021

Summary:

The two different approaches that seemed the best options from the four
here for running transformations were discussed in more detail
**Snowflake stored procedures** and **ADF dataflows**. In addition the
following assumption was introduced

-   **Most of the data warehouse build will not be generic parameterised
    transformations.** *e.g. fact table loads will be bespoke, joining
    to create dimensions will be bespoke, generic parts would include
    managing audit keys and update strategies for dimensions.*

Due to some of the reasons for initially rejecting ADF dataflows being
around the clunkyness of creating generic transformations the dismissal
of this option merited further discussion.

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Considerations</strong></p></th>
<th class="confluenceTh"><p><strong>ADF lookup activity triggers the
execution of a stored procedure in Snowflake</strong> DECIDED</p></th>
<th class="confluenceTh"><p><strong>ADF data flow with Snowflake linked
service</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>simplicity of transformations</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> SQL
scripts (embedded in Stored procs) are responsible for all
transformation so no split storage of business logic.</p>
<p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /> There
is a small amount of Javascript boiler plate code required in the stored
procedures</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /> Often
in SSIS (or other ETL tools) transformation logic gets split between the
tool and the sql on the data base (e.g. pushing down joins or other
statements for performance reasons).</p>
<p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" />
Transformation logic needs users to learn the ADF dataflow syntax and
nuances of the adf data flow expression browser</p>
<p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> Visual
transformations can be simpler for people with MSBI stack experience to
pick up</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Minimise the introduction of new services
unless necessary.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> No new
service needed.</p>
<p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /> There
is a small amount of Javascript boiler plate code required in the stored
procedures, this is a new language</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /> ADF
data flow is a new service to the existing solution.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>It has to be easy to use for on-going
maintenance and operation.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> ADF
Monitor contains all the logs, including errors from the Snowflake
side.</p>
<p><strong>Question</strong>: how easy are stored procedures to debug
and test</p>
<p><strong>Question:</strong> how quick and efficient is the development
process using stored procs</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> ADF
Monitor contains all the logs, including errors from the Snowflake
side.</p>
<p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /> From
the investigations However, the debug time can be long due to the
overhead time for a spark cluster to start and run. or if a debugging
cluster is left running it could be expensive. If a data flow is driven
by config, dynamic values prohibit using Preview function, making
debugging and checking output inconvenient.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Optimal usage of services.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> Fully
leverage Snowflake’s built-in optimisation and data processing
power.</p>
<p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /> The dev
warehouses can be set to spin down after a few minutes of being idle and
have no spin up time.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" />
Execution happens in memory of the spark cluster and data is required to
be moved in and out of snowflake. however VMIA’s data volumes are
low.</p>
<p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" />
debugging requires a spark cluster to be left on for a period of time
which could be more expensive than snowflakes on demand
warehouses</p></td>
</tr>
</tbody>
</table>

</div>

**Discussion summary.**

Bo Kwon (Deactivated) and Rahul Pandey (Deactivated) expressed a
preference to use stored procedures. With an expectation that most of
them would be bespoke, however if generic one can be created, then it
was agreed this would also be a good idea. no one was in favour of using
mapping data flows due to the movement of data in and out of snowflake,
the fact that VMIA resources are pretty comfortable with writing SQL and

Even though Bo is familiar with SSIS, she expressed that from her
experience, writing a sql statement is much neater and easier than
creating multiple steps in SSIS. Also, she prefers not to have complex
logic and scripts inside SSIS steps as it is hard to debug. The
comparison between a merge statement and the equivalent data flow
example seconded her first point. In her experience SSIS often involved
using SQL AND ssis packages leading to business logic in two places.
Plus it was discussed that leaving the transformation logic in SQL and
the monitoring in ADF made sense as the transformation logic could be
altered without changing the orchestration monitoring and logging. The
only negative to the chosen option is the addition of javascript
hoiwever this was not mentioned as a major worry.

There was a preference from Rahul Pandey (Deactivated) to have a POC to
see the design in action

</div>

</div>

</div>

</div>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441516051.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441450533.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441417738.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441384979.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441417747.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441319443.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441450542.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441450551.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441417756.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[data_warehouse_orchestration.svg](attachments/440271104/443973672.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~data_warehouse_orchestration.svg.tmp](attachments/440271104/440762483.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[data_warehouse_orchestration.svg.png](attachments/440271104/443973682.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441614369.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441417845.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441614378.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441614387.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441614396.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441614405.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441581604.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441417854.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441352280.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Data_warehouse_hyper_pipeline.svg.tmp](attachments/440271104/441319452.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Data_warehouse_hyper_pipeline.svg](attachments/440271104/441352289.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Data_warehouse_hyper_pipeline.svg.png](attachments/440271104/441417863.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~data_warehouse_orchestration.svg.tmp](attachments/440271104/444203016.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~data_warehouse_orchestration.svg.tmp](attachments/440271104/441352333.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~data_warehouse_orchestration.svg.tmp](attachments/440271104/444137485.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~data_warehouse_orchestration.svg.tmp](attachments/440271104/444137480.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[data_warehouse_orchestration.svg](attachments/440271104/441352202.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[data_warehouse_orchestration.svg.png](attachments/440271104/441352210.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201130-231006.png](attachments/440271104/455278597.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201130-231116.png](attachments/440271104/455147525.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201201-010014.png](attachments/440271104/455376914.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201201-010130.png](attachments/440271104/455278603.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201201-010314.png](attachments/440271104/455245829.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201201-010701.png](attachments/440271104/455376922.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201201-011150.png](attachments/440271104/455245837.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201201-223917.png](attachments/440271104/454950978.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201201-224434.png](attachments/440271104/455245907.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201202-015649.png](attachments/440271104/458096780.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201209-020958.png](attachments/440271104/474972195.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201209-021016.png](attachments/440271104/472613862.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201209-021206.png](attachments/440271104/472810563.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201209-045802.png](attachments/440271104/472810583.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201209-051341.png](attachments/440271104/475299872.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201209-052100.png](attachments/440271104/472810595.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201209-052424.png](attachments/440271104/475168806.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201215-055414.png](attachments/440271104/490307659.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201215-055618.png](attachments/440271104/487784571.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20210119-045451.png](attachments/440271104/577536836.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20210119-045503.png](attachments/440271104/577635272.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20210119-045645.png](attachments/440271104/574948486.png)
(image/png)  

</div>
