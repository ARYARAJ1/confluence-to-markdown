# Data Warehouse Platform (Snowflake Vs Synapse Analytics)

<div>

<div>

Add your comments directly to the page. Include links to any relevant
research, data, or feedback.

</div>

</div>

<div class="plugin-tabmeta-details">

<div class="table-wrap">

|              |                                                                                                                                                                                                                                                                                                                                      |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Status       | OUTCOME DECIDED                                                                                                                                                                                                                                                                                                                      |
| Impact       | MEDIUM                                                                                                                                                                                                                                                                                                                               |
| Driver       | Jason Dang (Deactivated)                                                                                                                                                                                                                                                                                                             |
| Approver     | Hendry Susilo                                                                                                                                                                                                                                                                                                                        |
| Contributors | h.lindsaysmith@vmia.vic.gov.au Lindsay-Smith                                                                                                                                                                                                                                                                                         |
| Informed     | Lasath Kahingala                                                                                                                                                                                                                                                                                                                     |
| Due date     | 20 Jul 2020                                                                                                                                                                                                                                                                                                                          |
| Outcome      | Following the demonstration, recommendation specified and internal project leadership discussion happy to endorse to proceed with Snowflake as VMIA’s Data Platform. Security, Risk and Privacy Assessment along with Master Services Agreement will be performed shortly before we can move to the deployment/implementation stage. |

</div>

</div>

## Background

A VMIA Enterprise Data Warehouse will provide human readable and system
consumable data to sources internal and external to the strategic data
platform. Selecting of a platform to serve this purpose is one of the
key design decisions of the data platform and there is a need to
evaluate potential technology candidates (Synapse analytics or
Snowflake) against a set of key considerations and requirements for
VMIA.

## Relevant data

### What is Synapse Analytics?

Azure Synapse Analytics is an evolution of Azure SQL Data Warehouse. At
its core, Azure Synapse contains the MPP, scale-out technology designed
to process and store large volumes of data within the Microsoft Azure
cloud platform.

Azure Synapse has four components:

-   Synapse SQL: Complete T-SQL based analytics – Generally Available

    -   SQL pool (pay per DWU provisioned)

        -   Represents a collection of analytic resources that re being
            provisioned when using Synapse SQL. The size of SQL pool is
            determined by Data Warehousing Units (DWU).

    -   SQL on-demand (pay per TB processed) (preview)

-   Spark: Deeply integrated Apache Spark (preview)

-   Synapse Pipelines: Hybrid data integration (preview)

-   Studio: Unified user experience. (preview)

### What is Snowflake?

Snowflake is an analytic data warehouse provided as
Software-as-a-Service (SaaS). Snowflake provides a data warehouse that
is faster, easier to use, and far more flexible than traditional data
warehouse offerings.

Snowflake’s unique architecture consists of three key layers:

-   Database Storage: Snowflake reorganizes that data into its internal
    optimized, compressed, columnar format. Snowflake stores this
    optimized data in cloud storage.

-   Query Processing: Query execution is performed in the processing
    layer. Snowflake processes queries using “virtual warehouses”. Each
    virtual warehouse is an MPP compute cluster composed of multiple
    compute nodes allocated by Snowflake from a cloud provider.

-   Cloud Services: The cloud services layer is a collection of services
    that coordinate activities across Snowflake. These services tie
    together all of the different components of Snowflake in order to
    process user requests, from login to query dispatch.

### Snowflake Vs Synapse Analytics Feature Comparison

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Feature</strong></p></th>
<th class="confluenceTh"><p><strong>Snowflake</strong></p></th>
<th class="confluenceTh"><p><strong>Synapse Analytics</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>Security</p></td>
<td class="confluenceTd"><ul>
<li><p>Ability to choose geographical location of where data should be
stored</p></li>
<li><p>User authentication through standard user/password as well as
MFA, SSO and OAuth included</p></li>
<li><p>Communication between client and server protected through
TLS</p></li>
<li><p>Can be deployed within an Azure Virtual Network</p></li>
<li><p>Isolation of data (for loading and unloading) using Azure Storage
Access Controls</p></li>
<li><p>Support for PHI data (in compliance with HIPAA regulations) —
requires Business Critical Edition</p></li>
<li><p>Automatic data encryption by Snowflakee</p></li>
<li><p>Object-level access control</p></li>
<li><p>column level data masking available</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>SQL pool currently supports SQL Server Authentication with a
username and password, and with Azure Active Directory</p></li>
<li><p>Authorization privileges are determined by role memberships and
permissions</p></li>
<li><p>Transparent Data Encryption (TDE) helps protect against the
threat of malicious activity by encrypting and decrypting your data at
rest. </p></li>
<li><p>Provides advanced capabilities for discovering, classifying,
labeling, and reporting the sensitive data in your databases.</p></li>
<li><p>Private Link and Virtual Endpoint is available</p></li>
<li><p>Column-level and row level security</p></li>
<li><p>column level data masking available</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Deployment Offering (SaaS, PaaS
etc)</p></td>
<td class="confluenceTd"><p>SaaS-delivered Data Warehouse as a
Service</p></td>
<td class="confluenceTd"><p>Platform as a Service (PaaS)</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>SQL Support</p></td>
<td class="confluenceTd"><p>Snowflake is a data platform and data
warehouse that supports the most common standardized version of SQL:
ANSI. This means that all of the most common operations are usable
within Snowflake. Snowflake also supports all of the operations that
enable data warehousing operations, like create, update, insert,
etc.</p></td>
<td class="confluenceTd"><p>Azure Synapse SQL is a big data analytic
service that enables you to query and analyze your data using the T-SQL
language. You can use standard ANSI-compliant dialect of SQL language
used on SQL Server and Azure SQL Database for data analysis.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>User Interface</p></td>
<td class="confluenceTd"><p>Web-based GUI for account and general
management, monitoring of resources and system usage, and querying
data.</p></td>
<td class="confluenceTd"><p>Synapse Studio (in preview, therefore not
generally available to the required Azure region and not recommended for
production workloads)</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Connectivity</p></td>
<td class="confluenceTd"><ul>
<li><p>Supports BI Tools such as PowerBI, Tableau</p></li>
<li><p>Supports multiple SQL Clients such as DBeaver and SQL
Workbench</p></li>
<li><p>Supports multiple programming interfaces including Python,
NodeJS, ODBC and JDBC</p></li>
<li><p>Can inter-operate with multiple ML and Data Science platforms
such as Databricks, SAS and Apache Spark</p></li>
<li><p>Can inter-operate with multiple data integration platforms such
as Azure Data Factory</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>supports mulitple BI tools</p></li>
<li><p>Can integrate with multiple Azure Services such as Power BI,
Azure Data Factory. Azure Machine Learning and Azure Stream Analytics
however these are still in preview.</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Data Loading</p></td>
<td class="confluenceTd"><ul>
<li><p>Support for bulk loading and unloading data into/out of tables,
including:</p>
<ul>
<li><p>Load any data that uses a supported character encoding.</p></li>
<li><p>Load data from compressed files.</p></li>
<li><p>Load most flat, delimited data files (CSV, TSV, etc.).</p></li>
<li><p>Load data files in JSON, Avro, ORC, Parquet, and XML
format.</p></li>
<li><p>Load from S3 data sources and local files using Snowflake web
interface or command line client.</p></li>
</ul></li>
<li><p>Support for continuous bulk loading data from files:</p>
<ul>
<li><p>Use Snowpipe to load data in micro-batches from internal (i.e.
Snowflake) stages or external (Amazon S3, Google Cloud Storage, or
Microsoft Azure) stages.</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd"><p>While SQL pool supports many loading
methods, including popular SQL Server options such as
<strong>b</strong>ulk <strong>c</strong>opy <strong>p</strong>rogram
utility and the SqlBulkCop API, the fastest and most scalable way to
load data is through PolyBase external tables and the COPY
Statement <strong>(preview)</strong>.</p>
<p>With PolyBase and the COPY statement, you can access external data
stored in Azure Blob storage or Azure Data Lake Store via the T-SQL
language. For the most flexibility when loading, we recommend using the
COPY statement.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Data Sharing</p></td>
<td class="confluenceTd"><ul>
<li><p>Secure Data Sharing enables sharing selected objects (tables,
secure views, and secure UDFs) in a database in your account with other
Snowflake accounts. There are 3 different types of product offering for
data sharing:</p>
<ul>
<li><p>Direct Share is the simplest form of data sharing that enables
account-to-account sharing of data utilizing Snowflake’s Secure Data
Sharing.</p></li>
<li><p>The Snowflake Data Marketplace utilizes Snowflake Secure Data
Sharing to connect providers of data with consumers. </p></li>
<li><p>Data Exchange is your own data hub for securely collaborating
around data between a selected group of members that you invite. It
enables providers to <em>publish</em> data that can then
be <em>discovered</em> by consumers. This is a preview feature</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd"><p>Azure Data Share enables organizations to
simply and securely share data with multiple customers and partners. In
just a few clicks, you can provision a new data share account, add
datasets, and invite your customers and partners to your data
share.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Database Replication and Failover</p></td>
<td class="confluenceTd"><p>Support for replicating and syncing
databases across multiple Snowflake accounts in different regions:</p>
<ul>
<li><p>Replicate databases between Snowflake accounts (within the same
organization) and keep the database objects and stored data
synchronized.</p></li>
<li><p>Configure database failover to one or more Snowflake accounts for
business continuity and disaster recovery.</p></li>
</ul></td>
<td class="confluenceTd"><p>Use SQL pool restore points to recover or
copy your data warehouse to a previous state in the primary region. Use
data warehouse geo-redundant backups to restore to a different
geographical region.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Workload Management</p></td>
<td class="confluenceTd"><p>Snowflake’s unique multi-cluster, shared
data architecture makes it possible to allocate multiple independent,
isolated clusters for processing while sharing the same data. Each
cluster (called a “virtual warehouse”) can both read and write data,
with full transactional consistency ensured by Snowflake’s cloud
services layer.</p>
<p>The size and resources of each virtual warehouse can be chosen
independently by the user based on the characteristics and performance
requirements for the workload(s) that will run on each virtual
warehouse.</p>
<p>With Snowflake’s ability to configure auto-scale mode, a
multi-cluster warehouse eliminates the need for resizing the warehouse
or starting and stopping additional warehouses to handle fluctuating
workloads. Snowflake automatically starts and stops additional clusters
as needed.</p></td>
<td class="confluenceTd"><p>Synapse SQL pool workload management in
Azure Synapse consists of three high-level concepts: Workload
Classification, Workload Importance and Workload Isolation. These
capabilities give you more control over how your workload utilizes
system resources.</p>
<ul>
<li><p>Workload classification is the concept of assigning a request to
a workload group and setting importance levels.</p></li>
<li><p>Workload importance influences the order in which a request gets
access to resources. On a busy system, a request with higher importance
has first access to resources. Importance can also ensure ordered access
to locks.</p></li>
<li><p>Workload isolation reserves resources for a workload
group. </p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Time Travel - Ability to access historical
data at any point in time<br />
</p></td>
<td class="confluenceTd"><p>Snowflake Time Travel enables accessing
historical data (i.e. data that has been changed or deleted) at any
point within a defined period. It serves as a powerful tool for
performing the following tasks.</p>
<p>Using Time Travel, you can perform the following actions within a
defined period of time:</p>
<ul>
<li><p>Query data in the past that has since been updated or
deleted.</p></li>
<li><p>Create clones of entire tables, schemas, and databases at or
before specific points in the past.</p></li>
<li><p>Restore tables, schemas, and databases that have been
dropped.</p></li>
</ul></td>
<td class="confluenceTd"><p>Synapse Analytics does not have this
capability</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Fail-safe: Ensures historical data is
protected in the event of a system failure or other catastrophic event,
e.g. a hardware failure or security breach.</p></td>
<td class="confluenceTd"><p>Fail-safe provides a (non-configurable)
7-day period during which historical data is recoverable by Snowflake.
This period starts immediately after the Time Travel retention period
ends.</p></td>
<td class="confluenceTd"><p>The capability that is most similar to this
is Synapse backup and restore capability. However this method doubles or
even triples overall data storage. In addition, data recovery can be
painful and costly due to numerous factors, including:</p>
<ul>
<li><p>Time required to reload lost data.</p></li>
<li><p>Business downtime during recovery.</p></li>
<li><p>Loss of data since the last backup.</p></li>
</ul>
<p>A bespoke manual process or automated solution would be needed to
take advantage of the bakcup and restore.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Database Administration</p></td>
<td class="confluenceTd"><p>Snowflake requires near zero database
administration management</p></td>
<td class="confluenceTd"><p>Scaling an Azure Synapse Analytics data
warehouse requires administrator attention. Administrators are also
required to partition data structures to improve performance and do
other kinds of performance optimization. in addition once the extra
parts of synapse analytics make it out of preview there is more overhead
to maintain metadata for 4 diefferen query engines and decide which one
to use.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Data Cloning: Cloning creates a copy of a
database, schema or table. A snapshot of data present in the source
object is taken when the clone is created, and is made available to the
cloned object. The cloned object is writable, and is independent of the
clone source. That is, changes made to either the source object or the
clone object are not part of the other</p></td>
<td class="confluenceTd"><p>With Snowflake’s Zero Copy Cloning Feature,
customers can create multiple copies of the data tables, schemas, and
databases, without replicating the data itself. This gives customers the
ability to almost <em>instantly</em> make the data available to use for
multiple user groups, without the additional cost (or time) of actually
replicating the data.</p></td>
<td class="confluenceTd"><p>Synapse Analytics does not have this
capability</p></td>
</tr>
</tbody>
</table>

</div>

### Snowflake Pricing Data

Snowflake pricing is based on the usage of the three distinct layers in
their architecture (storage, virtual warehouses (compute) and cloud
services) and of serverless features. Snowflake credits are used to pay
for the consumption of resources on Snowflake. Snowflake offers 4
different editions, with each providing progressively more features.

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Standard Edition</strong></p></th>
<th class="confluenceTh"><p><strong>Enterprise Edition</strong></p></th>
<th class="confluenceTh"
data-highlight-colour="#4c9aff"><p><strong>Business
Critical</strong></p></th>
<th class="confluenceTh"><p><strong>Virtual Private
Snowflake</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"
data-highlight-colour="#f4f5f7"><p><strong>Features</strong></p></td>
<td class="confluenceTd"><ul>
<li><p>Complete SQL Data Warehouse</p></li>
<li><p>Secure Data Sharing across regions / clouds</p></li>
<li><p>Premier Support 24 x 365</p></li>
<li><p>1 day of time travel</p></li>
<li><p>Always-on enterprise grade encryption in transit and at
rest</p></li>
<li><p>Customer dedicated virtual warehouses</p></li>
<li><p>Federated authentication</p></li>
<li><p>Database Replication</p></li>
<li><p>External Functions</p></li>
<li><p>Snowsight analytics UI</p></li>
<li><p>Create your own Data Exchange</p></li>
<li><p>Data Marketplace access</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Standard +</p></li>
<li><p>Multi-cluster warehouse</p></li>
<li><p>Up to 90 days of Time Travel</p></li>
<li><p>Annual rekey of all encrypted data</p></li>
<li><p>Materialized Views</p></li>
<li><p>Search Optimization Service</p></li>
<li><p>Dynamic Data Masking</p></li>
<li><p>External Data Tokenization</p></li>
</ul></td>
<td class="confluenceTd" data-highlight-colour="#4c9aff"><ul>
<li><p>Enterprise +</p></li>
<li><p>HIPAA Support</p></li>
<li><p>PCI Compliance</p></li>
<li><p>Data encryption everywhere</p></li>
<li><p>Tri-Secret Secure using customer managed keys</p></li>
<li><p>Azure PrivateLink support</p></li>
<li><p>Database failover and failback for business continuity</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Business Critical +</p></li>
<li><p>Customer dedicated virtual servers wherever the encryption key is
in memory</p></li>
<li><p>Customer dedicated metadata store</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd" data-highlight-colour="#f4f5f7"><p>Price Per
Snowflake Credit<br />
(On Demand)</p></td>
<td class="confluenceTd"><p>$2.75</p></td>
<td class="confluenceTd"><p>$4.05</p></td>
<td class="confluenceTd"
data-highlight-colour="#4c9aff"><p>$5.50</p></td>
<td class="confluenceTd"><p>Contact Snowflake</p></td>
</tr>
</tbody>
</table>

</div>

## Options considered

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p>Option 1: Snowflake</p></th>
<th class="confluenceTh"><p>Option 2: Synapse Analytics</p></th>
</tr>

<tr class="odd">
<th class="confluenceTh"><p>Description</p></th>
<td class="confluenceTd"><p>Snowflake is an analytic data warehouse
provided as Software-as-a-Service (SaaS). Snowflake provides a data
warehouse that is faster, easier to use, and far more flexible than
traditional data warehouse offerings.</p></td>
<td class="confluenceTd"><p>Azure Synapse Analytics is an evolution of
Azure SQL Data Warehouse. At its core, Azure Synapse contains the MPP,
scale-out technology designed to process and store large volumes of data
within the Microsoft Azure cloud platform.</p></td>
</tr>
<tr class="even">
<th class="confluenceTh"><p>Pros and cons</p></th>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Ability
to access historical data at any point in time through time travel</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" />
Additional Data Protection through Fail Safe</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Ability
to quickly clone/replicate data for development/testing purposes without
incurring additional time or storage costs</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Close
to zero database administration</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" />
Baseline security managed by snowflake</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Deployment infrastructure owned by snowflake</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> care
and attention needed to make sure snowflake costs are optimised</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> some
additional network configuration required</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Native
Azure Resource, no separate procurement</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Some
Azure services such as Azure Data Factory will be closely integrated
with Synapse Analytics</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Product suite is less mature then Snowflake as a many new features are
still in preview</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Requires a database administrator</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> Does
not have any time travel and cloning capabilities requiring extra
solution engineering to workaround.</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> less
flexibility for workload management</p></td>
</tr>
<tr class="odd">
<th class="confluenceTh"><p>Estimated cost</p></th>
<td class="confluenceTd"><p>compute and storageMEDIUM</p>
<p>implementation and operational LOW</p></td>
<td class="confluenceTd"><p>compute and storageMEDIUM</p>
<p>implementation and operational HIGH</p></td>
</tr>
</tbody>
</table>

</div>

<a href="attachments/8978487/88047818.xlsx"
data-nice-type="Microsoft Excel Spreadsheet"
data-file-src="/wiki/download/attachments/8978487/DataWarehouse%20Pricing%20Calculator%20for%20VMIA.xlsx?version=1&amp;modificationDate=1594705903477&amp;cacheVersion=1&amp;api=v2"
data-linked-resource-id="88047818"
data-linked-resource-type="attachment"
data-linked-resource-container-id="8978487"
data-linked-resource-default-alias="DataWarehouse Pricing Calculator for VMIA.xlsx"
data-mime-type="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
data-has-thumbnail="true" data-linked-resource-version="1"
data-media-id="2b35cac4-b038-42bc-a44e-43a00354983a"
data-media-type="file"><img
src="attachments/thumbnails/8978487/88047818" height="250" /></a>

Estimated snowflake costs are $40-60k AUD. These estimates can be
refined during build

## Action items

-   

## Recommendation

In comparison to Synapse Analytics, Snowflake has much more advanced
capabilities as part of their data warehouse offering such as time
travel, zero copy cloning, complete workload isolation with instaneous
elastic scaling and SnowPipe data ingestion. It also inter-operates with
a vast majority of industry leading tools and technologies, some of
which are relevant and/or currently in use within VMIA such as Azure
Data Factory, Tableau, PowerBI, SAS and Databricks. Snowflake also
offers an almost completely automated solution with close to zero effort
required to maintain and administer the platform such as auto-scaling,
thus much reducing the need for a database administrator within VMIA
past setting up schemas, uers and groups

While Synapse Analytics is considered an evolution of Azure’s previous
offering, Azure SQL Data Warehouse, a number of it’s features are still
in preview as of the date of this document, which suggests in comparison
to Snowflake, is lacking behind in platform capabilities. In addition to
this, with Synapse Analytics, VMIA would be required to invest in a
database administrator role to manage and optimise the resources within
and usage of the platform. Even when the preview features arrive they
appear to be a Web UI skin over a number of difference product, leading
to a confusing experience, compared to snowflakes simpler solution.

Therefore the recommendation would be for VMIA to use Snowflake as the
Data Warehouse Platform for the Strategic Data Infrastructure Platform.
Additionally, the recommended edition VMIA should consider using would
be Business Critical as this offers even higher levels of data
protection to support the needs of organisations with extremely
sensitive data such as Health Data.

## Outcome

Following the demonstration, recommendation specified and internal
project leadership discussion happy to endorse to proceed with Snowflake
as VMIA’s Data Platform. Security, Risk and Privacy Assessment along
with Master Services Agreement will be performed shortly before we can
move to the deployment/implementation stage.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DataWarehouse Pricing Calculator for
VMIA.xlsx](attachments/8978487/88047818.xlsx)
(application/vnd.openxmlformats-officedocument.spreadsheetml.sheet)  

</div>
