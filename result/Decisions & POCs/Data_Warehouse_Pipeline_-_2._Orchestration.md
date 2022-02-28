# Data Warehouse Pipeline - 2. Orchestration

# Introduction

This documentation records possible options that can be used to
orchestrate data warehouse pipeline. The main processes in the data
warehouse pipeline are long-running SQL scripts that load data to
various tables with joins, inserts, updates, etc. Therefore, the
orchestration tools in consideration need to have no constraint on
timeout. There will be inherent dependencies between the SQL scripts,
such as ‘load to a policy table when both risk console policy extracts
and the latest C360 policy extracts are available’. The chosen
orchestration option needs to balance

-   simplicity to build

-   simplicity to operate and maintain

-   flexibility to chain together reusable transformation components and
    bespoke transformation components

-   alignment to the existing SDI technology stack, where possible.

# Options of Orchestration

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Use ADF to orchestrate and run
activities</strong> DECIDED</p></th>
<th class="confluenceTh"><p><strong>Use Durable function to orchestrate
and run activities</strong></p></th>
<th class="confluenceTh"><p><strong>Azure Databricks Workspace to
orchestrate and perform ETL jobs</strong></p></th>
<th class="confluenceTh"><p><strong>Use Logic Apps to orchestrate and
run actions</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>Description</p></td>
<td class="confluenceTd"><p>ADF pipeline is used to as the orchestration
tool to string ADF activities and other Azure services together with
built-in logic components to direct processes for different
situation.</p></td>
<td class="confluenceTd"><p>Use a durable function as a long running
execution process to codify the data warehouse pipeline with components
such as an orchestrator, activities and a trigger. Activities can
include a dependency checker and processes that run and monitor the
outcome of SQL executions in Snowflake.</p></td>
<td class="confluenceTd"><p>Azure Databricks is a fully managed service
that consists of two services:</p>
<ul>
<li><p>Azure Databrcks SQL Analytics <a
href="https://docs.microsoft.com/en-us/azure/databricks/scenarios/what-is-azure-databricks-sqla"
rel="nofollow">runs and visualises SQL queries on azure data lake.</a>
It only integrates with Azure databases and stores, and Power BI. It’s
in preview.</p></li>
<li><p>Azure Databricks Workspace is an analytics platform that enables
multi-user collaboration based on Apache Spark. It can connect to
Snowflake via <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/398852110"
rel="nofollow">Spark connector in Python, Scala, and R</a>. Databricks
is a long running process that runs on a cluster in the VNET.</p></li>
</ul></td>
<td class="confluenceTd"><p>Logic Apps acts as an overarching layer that
strings actions and ADF pipelines together. Unlike ADF, Logic Apps is an
older service that can integrate with more Azure services than ADF. To
integrate with services outside of Azure (e.g. snowflake), Logic Apps
needs to trigger an Azure service that can integrate with the external
service. See <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/401605809/Azure+Logic+Apps+Use+Cases+and+Capability+Exploration"
data-linked-resource-id="401605809" data-linked-resource-version="12"
data-linked-resource-type="page">Azure Logic Apps Use Cases and
Capability Exploration</a> for detail assessment of the <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/401605809/Azure+Logic+Apps+Use+Cases+and+Capability+Exploration#Use-Case-2%3A-Use-Logic-Apps-as-More-Generic-Orchestration-Tool-for-Data-Pipelines"
rel="nofollow">suitability of Logic Apps for orchestration</a>.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Advantages</p></td>
<td class="confluenceTd"><ul>
<li><p>Consistent user experience with Data Lake process, e.g. rerun
from the failed activity, one place to monitor pipeline status.</p></li>
<li><p>Reuse existing knowledge on Azure services which minimises the
risk of encountering unknown issues and service limitations.</p></li>
<li><p>Data factory is already setup with connectivity to
snowflake</p></li>
<li><p>Easy setup via configuration with minimum code (if an activity is
an Azure function).</p></li>
<li><p>Can easily control sequential or parallel runs in a ForEach
loop.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Can view durable function as a codified pipeline that contains
orchestrator, activity, entity and trigger.</p></li>
<li><p>Can leverage <a
href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/quickstart-python-vscode"
rel="nofollow">Visual Studio Code for a quick start with
templates</a></p></li>
<li><p>Failure occurring in activities will surface to orchestration. A
durable function has an easiy integration with and consumes
orchestration status. There are three options:</p>
<ul>
<li><p>Using the <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/401605809/Azure+Logic+Apps+Use+Cases+and+Capability+Exploration"
rel="nofollow">custom status</a> feature to periodically update and
query the status of a durable function to reflect its progress,</p></li>
<li><p>Using the <a
href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-http-api#get-instance-status"
rel="nofollow">Client Function (trigger) inside of a durable function to
read the durable entity</a>,</p></li>
<li><p>Write progress into any databases and read back from the database
in the poll HTTP endpoint.</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>A powerful and flexible option that can only orchestrate and run
ETL processes but also heavy-lifting analytical processing.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Logic Apps is considered as a <a
href="https://platform.deloitte.com.au/articles/azure-durable-functions-vs-logic-apps#:~:text=Durable%20Functions%20is%20an%20extension,workflows%20through%20a%20visual%20designer."
rel="nofollow">comparable codeless alternative of durable function for
server-less and stateful workflow solution.</a> It surpasses durable
function and ADF in its integration with 200+ Azure services, Visual
Designer and troubleshoots.</p></li>
<li><p>The ForEach loop triggers actions by default in parallel and can
use the concurrency setting limit to 1 to achieve sequential
execution.</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Disadvantages</p></td>
<td class="confluenceTd"><ul>
<li><p>In ADF Monitor, there is no default way to expose what data feed
ran in each pipeline. Trigger name, Annotation and User Properties need
to be leveraged to provide more metadata to make debugging experience
easier.</p></li>
<li><p>There are some <a
href="https://github.com/MicrosoftDocs/azure-docs/blob/master/includes/azure-data-factory-limits.md"
rel="nofollow">hard limits on using ADF</a>:</p>
<ul>
<li><p>concurrent pipeline activity runs per subscription per IR region:
1000</p></li>
<li><p>Maximum activities per pipeline: 40</p></li>
<li><p>Maximum parameters per pipeline: 50</p></li>
<li><p>ForEach parallelism: 50</p></li>
<li><p>for the managed integration runtime - Data Integration Units per
copy activity run: 256</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>It is a new service to introduce into the existing solution and
therefore, may bring in uncertainty and risk of unknown issues.</p></li>
<li><p>A durable function is a complex service (contains several types
of functions: orchestrator, activity, entity to manage state and client.
) to introduce into the solution and therefore may introduce unknown
issues and limitations that require time and effort to resolve.</p></li>
<li><p>An orchestrator function needs to follow <a
href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-code-constraints"
rel="nofollow">strict requirements (deterministic, i.e. an orchestrator
function will be replayed multiple times, and it must produce the same
result each time) on how to write the code</a>.</p></li>
<li><p>Activity function is <a
href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview"
rel="nofollow">only guaranteed at least once execute</a> (so it will
incur duplicated execution from time to time) and therefore we need to
make it idempotent, which will add complexity. It can only be triggered
by an orchestrator function.</p></li>
<li><p>Orchestrator and entity functions cannot be triggered directly
using the buttons in the Azure Portal. <a
href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview"
rel="nofollow">Must run a client function</a> that starts an
orchestrator or entity function as part of its implementation. This has
implications for easy operation of the platform.</p></li>
<li><p>A minimum complete durable function requires <a
href="https://docs.microsoft.com/en-us/azure/azure-functions/durable/quickstart-python-vscode"
rel="nofollow">an orchestrator, an activity and a client function as a
trigger</a>. Python version is in preview, and only those three are
available.</p></li>
<li><p>The entire pipeline, restart logic, logging and associated tests
will need to be coded from scratch.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>It is a new service to introduce into the existing solution and
therefore, may bring in uncertainty and risk of unknown issues.</p></li>
<li><p>Azure Databricks may experience some start-time and auto-scaling
delay as it will add new a VM into a cluster. Azure Databricks pools can
reduce the delay by having a set of VMs idle and ready to use. Azure
doesn’t charge for Databricks Units (DBUs), but will charge for idling
VMs.</p></li>
<li><p>A workspace is <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/263421967/Azure+Data+Factory+-+Build"
rel="nofollow">limited to 1000</a> concurrent job runs or needs to
handle 429 Too Many Requests response. It’s limited to 5000 job
creations in a workspace incl. “run now” and “runs submit”.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>It is a new service to introduce into the existing solution and
therefore, may bring in uncertainty and risk of unknown issues.</p></li>
<li><p>Using Logic Apps requires a significant amount of changes to the
existing implemented solution (if we want to keep solution consistent
across Pega and Non-Pega pipeline), e.g. the hyper pipeline will be done
in Logic Apps, using activity output to pass over values can no longer
work if across sub-pipelines, an app can only have one trigger and
therefore needs to either use a loop to process all data feeds or a
different way to inform what data feeds are processed.</p></li>
<li><p>Logic Apps doesn’t care about if any of the execution in a loop
is executed successfully. It only checks if it successfully triggers an
execution. Therefore, a Logic App can show successfully finishing
execution while underneath actions (e.g. ADF pipelines) fail. It can
bring in challenges in handling dependency actions (e.g. we would need
to query pipeline status after a pipeline finishes) and users need to go
to two places (i.e. Logic Apps history and ADF Monitor) to have a
complete picture on execution status.</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

D

## Exploration on Event-driven Architecture

When loading data into a data warehouse, we will need to apply different
methods for different tables. Some of the SQP scripts could run over a
few minutes, which can cause the Azure function to timeout. To handle a
long-running, event-driven architecture is considered as Azure function
can execute a SQL script without waiting for it to finish running. To
know whether the script ends with success, it requires Snowflake to
notify us in a certain way. Therefore, this is the purpose of
exploration.

We found documentation on <a
href="https://docs.snowflake.com/en/sql-reference/sql/create-notification-integration.html#usage-notes"
rel="nofollow">Snowflake notification integration</a>, which allows the
integration with Azure Storage Queue. Notification integration is a
feature in preview and looks promising at first glance. However, after
investigating a use case in detail, it turns out the **notification
integration** is for Snowflake to **receive events** from the Azure
environment, rather than for Snowflake to emit events to Azure. The <a
href="https://docs.snowflake.com/en/user-guide/tables-external-azure.html"
rel="nofollow">use case</a> is about Snowflake automatically refreshes
the information in an external table when a file gets added, changed or
removed in Azure Storage Blob. The illustration shows the role of
Snowflake in this event-driven solution.

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio8374305917175712235"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio8374305917175712235"
class="ap-content">

</div>

</div>

As shown in the diagram, because the Azure Storage Queue directly links
to the notification integration, Snowflake is more a consumer of the
events in the queue. If Snowflake is an event producer, it should
directly link to something like Azure Grid to send out events.
Therefore, **the conclusion is that Snowflake currently cannot support
the event-driven architecture as an event emitter for our use case**.

**Note**: have considered and explored other possibility of event-driven
architecture with Snowflake. However,
<a href="https://www.snowflake.com/guides/event-driven-architecture"
rel="nofollow">Snowflake can be the sunk storage for events coming from
Kafka</a>, but not an event emitter.

# Decision

To decide which option is most suitable, it depends on the what tool can
support required activities. Based on the outcome of
<a href="Data_Warehouse_Pipeline_-_1._Execute_Snowflake_Query.md"
data-linked-resource-id="440271104" data-linked-resource-version="32"
data-linked-resource-type="page">Data Warehouse Pipeline - 1. Execute
Snowflake Query</a>, **ADF lookup activity triggers the execution of a
stored procedure in Snowflake** is the desired option for executing
long-running SQL scripts in Data Warehouse Pipeline. Therefore, the
comparison below is based on this option.

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Principles</strong></p></th>
<th class="confluenceTh"><p><strong>Use ADF to orchestrate and run
activities</strong>DECIDED</p></th>
<th class="confluenceTh"><p><strong>Use Durable function to orchestrate
and run activities</strong></p></th>
<th class="confluenceTh"><p><strong>Azure Databricks Workspace to
orchestrate and perform ETL jobs</strong></p></th>
<th class="confluenceTh"><p><strong>Use Logic Apps to orchestrate and
run actions</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>Simple solution is better solution.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /></p>
<p>Use configuration rather than code to implement the orchestration and
ADF native activities. ForEach loop can handle both sequential and in
parallel execution.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>All the orchestration, activities, triggers will need to be codified
with additional configuration for deployment.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>All the orchestration and activities will need to be codified with
additional configuration for deployment.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /></p>
<p>Use configuration rather than code to implement the orchestration and
Logic Apps' native activities. Can integrate with ADF pipelines. ForEach
loop by default runs in parallel but can use the setting to limit
concurrency to be 1 to achieve sequential execution.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Minimise the introduction of new services
unless necessary.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /></p>
<p>No new service needed.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>Durable Function is a new service to the existing solution.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>Azure Databricks is a new service to the existing solution.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>Logic Apps is a new service to the existing solution.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>It has to be easy to use for on-going
maintenance and operation.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /></p>
<p>ADF Monitor contains all the logs including errors from the Snowflake
side. Logs are also available in Log Analytics workspace.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>A data engineer needs to check both ADF Monitor and Durable Function
logs (i.e. App Insights). All the logs can be accessible from Log
Analytics workspace.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>Azure Databricks logs can be sent to a Log Analytics workspace but <a
href="https://docs.microsoft.com/en-us/azure/architecture/databricks-monitoring/application-logs"
rel="nofollow">requires to leverage some libraries and implement in
code</a>.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>There is no user friendly GUI to see Logic Apps status except in the
Designer view using manually trigger. Logs are available in Log
Analytics workspace.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Let a service to do it best at and leverage
the built-in optimisation.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /></p>
<p>ADF is built to orchestrate and run ETL jobs.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /></p>
<p>Durable function is built to orchestrate and execute activities with
flexibility.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/error.png"
class="emoticon emoticon-cross" data-emoji-id="atlassian-cross_mark"
data-emoji-shortname=":cross_mark:" data-emoji-fallback=":cross_mark:"
data-emoticon-name="cross" width="16" height="16" alt="(error)" /></p>
<p>Azure Databricks is more built to execute SQL on Azure data storages
or run analytical computation.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16" alt="(tick)" /></p>
<p>Logic Apps is built for integration and orchestration a large number
of Azure services.</p></td>
</tr>
</tbody>
</table>

</div>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460619841.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460619850.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460685413.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460685422.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460292206.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460128365.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460161134.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460554312.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460456059.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~snowflake_event_driven.svg.tmp](attachments/460554258/460128333.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[snowflake_event_driven.svg](attachments/460554258/460652610.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[snowflake_event_driven.svg.png](attachments/460554258/460554325.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~snowflake_event_driven.svg.tmp](attachments/460554258/460292215.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~snowflake_event_driven.svg.tmp](attachments/460554258/460456076.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[snowflake_event_driven.svg](attachments/460554258/460685431.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[snowflake_event_driven.svg.png](attachments/460554258/460456068.png)
(image/png)  

</div>
