# Data Lake and Data Warehouse Rollback and Reload

# Introduction

We may ingest incorrect data into the data lake and data warehouse from
time to time due to new changes implemented in a source system, wrong
manual data entry, etc. To ensure data integrity in the data lake and
data warehouse, where downstream analysts and business users would
directly consume the data for reporting and analysis, we need to remove
incorrect data from the data lake and data warehouse and reload with
correct data. Therefore, a rollback and reload in data lake and data
warehouse will happen in:

-   data warehouse consumption (star schema) layer (snowflake)

-   data warehouse storage (data vault) layer (snowflake)

-   data lake curated layer (snowflake)

-   data lake acquisition layer (storage account)

## Caveat

There are a numerous of erroneous situations and it is impossible to
have a document to cover all the bits and corners. The purpose of the
documentation is to provide **building blocks** that can be used to deal
with some common situations. It is not a manual that prescribes
treatments for all situations. The appropriate treatment is the result
of careful **examination of cause, impact and trade-off**.

Likewise it is not prudent to engineer a solution for something which
may never happen or only happen once. The **principle** should always be
to make sure **no errors go undetected** (e.g. reconciliation frameworks
pick them up), and then **error which occur regularly** are candidates
to engineer a solution to **automate** the fixes.

<div class="toc-macro rbtoc1639539663466">

-   1
    [Introduction](#DataLakeandDataWarehouseRollbackandReload-Introduction)
    -   1.1 [Caveat](#DataLakeandDataWarehouseRollbackandReload-Caveat)
-   2 [Assumptions and
    Treatments](#DataLakeandDataWarehouseRollbackandReload-AssumptionsandTreatments)
-   3 [Rollback](#DataLakeandDataWarehouseRollbackandReload-Rollback)
    -   3.1 [Pre-requisite for
        Rollback](#DataLakeandDataWarehouseRollbackandReload-Pre-requisiteforRollback)
    -   3.2 [Rollback Sequence in Data Lake and Data Warehouse
        (High-level)](#DataLakeandDataWarehouseRollbackandReload-RollbackSequenceinDataLakeandDataWarehouse(High-level))
    -   3.3 [Rollback Process Step-by-Step in Snowflake (Insert &
        Update)](#DataLakeandDataWarehouseRollbackandReload-RollbackProcessStep-by-StepinSnowflake(Insert&Update))
    -   3.4 [Rollback Process Step-by-Step in Snowflake (Data Vault,
        Insert
        Only)](#DataLakeandDataWarehouseRollbackandReload-RollbackProcessStep-by-StepinSnowflake(DataVault,InsertOnly))
    -   3.5 [Rollback Process Step-by-Step in Storage
        Account](#DataLakeandDataWarehouseRollbackandReload-RollbackProcessStep-by-StepinStorageAccount)
-   4 [Reload](#DataLakeandDataWarehouseRollbackandReload-Reload)
    -   4.1 [Pre-requisite for
        Reload](#DataLakeandDataWarehouseRollbackandReload-Pre-requisiteforReload)
    -   4.2 [Load the New Batch with Correction for Delta and
        Full](#DataLakeandDataWarehouseRollbackandReload-LoadtheNewBatchwithCorrectionforDeltaandFull)
    -   4.3 [Load the New Batch with Correction for
        Transaction](#DataLakeandDataWarehouseRollbackandReload-LoadtheNewBatchwithCorrectionforTransaction)

</div>

# Assumptions and Treatments

How to handle errors in data caused by a source system depends on the
**type of load** and **how a source system provides the correct reload
batch**.

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Error Impact</strong></p></th>
<th class="confluenceTh"><p><strong>Assumptions</strong></p></th>
<th class="confluenceTh"><p><strong>Treatment</strong></p></th>
<th class="confluenceTh"><p><strong>Decision</strong></p></th>
</tr>

<tr class="odd">
<td rowspan="2" class="confluenceTd"><p>Delta pattern</p>
<p>(PEGA, Risk Console)</p></td>
<td class="confluenceTd"><p>SMALL</p></td>
<td class="confluenceTd"><p>A source system includes the correct records
in the new batch.</p></td>
<td class="confluenceTd"><p><strong>No rollback</strong>. Load the new
batch with correction as usual. This treatment retains the errors
associated with a subset of records within a small period of time as an
batch of records with a valid to and from.</p></td>
<td rowspan="2" class="confluenceTd"><p>Does it matter that the bad
records persist forever in between the first load and corrected
load?</p>
<p>This would mean ‘as-at’ analysis for the time period would always
show the data</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>LARGE</p></td>
<td class="confluenceTd"><p>A source system corrects the records in the
bad batch and resends the batch.</p></td>
<td class="confluenceTd"><p><strong>Roll back</strong> to the snapshot
prior to the ingestion of the bad batch. Reload the resent batch and all
the subsequent batches.</p></td>
</tr>
<tr class="odd">
<td rowspan="2" class="confluenceTd"><p>Full pattern</p>
<p>(Risk Console, Client Connect)</p></td>
<td class="confluenceTd"><p>SMALL</p></td>
<td class="confluenceTd"><p>A source system correct the error in the new
batch.</p></td>
<td class="confluenceTd"><p><strong>No rollback</strong>. Load the new
batch with correction as usual.</p></td>
<td rowspan="2" class="confluenceTd"><p>As above does it matter if the
bad data persists?</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>LARGE</p></td>
<td class="confluenceTd"><p>A source system correct the error in the
earliest bad batch and resends all the affected subsequent
batch.</p></td>
<td class="confluenceTd"><p><strong>Roll back</strong> to the snapshot
prior to the ingestion of the bad batch. Reload the all the resent
batches.</p></td>
</tr>
<tr class="odd">
<td rowspan="2" class="confluenceTd"><p>Transaction pattern</p>
<p>(Risk Console)</p></td>
<td class="confluenceTd"><p>SMALL</p></td>
<td class="confluenceTd"><p>A source system includes the correct records
in the new batch.</p></td>
<td class="confluenceTd"><p><strong>Replace</strong> the bad records
with the new records.</p></td>
<td rowspan="2" class="confluenceTd"><p>As above does it matter if the
bad data persists?</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>LARGE</p></td>
<td class="confluenceTd"><p>A source system correct the records in in
the bad batch and resends the batch.</p></td>
<td class="confluenceTd"><p><strong>Roll back</strong> to the snapshot
prior to the ingestion of the bad batch. Reload the resent batch and all
the subsequent batches.</p></td>
</tr>
</tbody>
</table>

</div>

**Note**: The table provides a simplified version of the situations and
treatments. However, how we actually handle the situation will be done
at case by case level, e.g. if a value is incorrect, the easiest, but
risky method could be manually corrected in the raw layer files and then
load to the curate layer and the data warehouse.

# Rollback

## Pre-requisite for Rollback

We can roll back Database tables by leveraging
the <a href="https://docs.snowflake.com/en/user-guide/data-time-travel.html"
rel="nofollow"><strong>time travel function in Snowflake</strong></a>.
The **retention period **for VMIA is currently configured as **60
days**, which means that we can travel back up to 60 days. Knowing when
to roll back to requires the following **audit fields** to exist in a
snowflake table:

-   **INSERT_RUN_ID**. This is a unique identifier generated by ADF used
    to identify a pipeline processed and orchestrated by ADF for a
    batch. This field will be populated when a record is first inserted.
    The RunId is of the sub-pipeline, not the hyper pipeline.

-   **UPDATE_RUN_ID**. Same as the INSERT_RUN_ID and it will be
    populated when an existing record is updated by a process later,
    e.g. end-dated. The RunId is of the sub-pipeline.

-   **RECORD_INSERT_DATETIME**. This is the timestamp when a record gets
    inserted into a snowflake table. It can be controlled either by an
    external function (e.g. azure function) (recommended for
    consistency) or Snowflake's internal clock.

-   **RECORD_UPDATE_DATETIME.** This is the timestamp when an existing
    record gets updated with new information, e.g. end-dated. It can be
    controlled either by an external function (e.g. azure function)
    (recommended for consistency) or Snowflake's internal clock.

-   **FILE_PATH**. This is the file path of a file within a **storage
    account raw layer**, including the container name and directory.
    This file path can be ready for the **Delete Activity in ADF** to
    remove a file.

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeContent panelContent pdl">

``` java
# file path example
raw-sdi-prd-datalake/PEGA_PROTECTED/VMIA_CSP_Work_Claims_Estimate/YEAR=2020/MONTH=11/BIX_VMIA_CSP_Work_Claims_Estimate_CaseExtract_201120_1/VMIA_CSP_Work_Claims_Estimate_CaseExtract_1.xml
```

</div>

</div>

For different layers, due to the various operations, not all the **audit
fields** are required for all tables.

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Data Lake - Curated (Time Varying)
-</strong></p>
<p><strong>(Insert &amp; Update)</strong></p></th>
<th class="confluenceTh"><p><strong>Data Vault - Hub</strong></p>
<p><strong>(Insert Only)</strong></p></th>
<th class="confluenceTh"><p><strong>Data Vault - Satellite</strong></p>
<p><strong>(Insert &amp; Update)</strong></p></th>
<th class="confluenceTh"><p><strong>Data Vault - Link</strong></p>
<p><strong>(Insert Only &amp; update unless link satellites are
introduced)</strong></p></th>
<th class="confluenceTh"><p><strong>Star Schema - Dimension</strong></p>
<p><strong>(Insert &amp; Update)</strong></p></th>
<th class="confluenceTh"><p><strong>Data Vault - Bridge</strong></p>
<p><strong>(Insert &amp; Update)</strong></p></th>
<th class="confluenceTh"><p><strong>Data Vault - Fact</strong></p>
<p><strong>(Insert &amp; Update, or Insert Only depending on fact
type)</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p><strong>INSERT_RUN_ID</strong></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p><strong>UPDATE_RUN_ID</strong></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
</tr>
<tr class="odd">
<td
class="confluenceTd"><p><strong>RECORD_INSERT_DATETIME</strong></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
</tr>
<tr class="even">
<td
class="confluenceTd"><p><strong>RECORD_UPDATE_DATETIME</strong></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p><strong>FILE_PATH</strong></p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/check.png"
class="emoticon emoticon-tick" data-emoji-id="atlassian-check_mark"
data-emoji-shortname=":check_mark:" data-emoji-fallback=":check_mark:"
data-emoticon-name="tick" width="16" height="16"
alt="(tick)" /></p></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
</tr>
</tbody>
</table>

</div>

## Rollback Sequence in Data Lake and Data Warehouse (High-level)

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio988936581775550612"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio988936581775550612"
class="ap-content">

</div>

</div>

The top diagram illustrates how we can use data feed to figure out
affected tables in the data lake and data warehouse. The bottom graph
shows a proposed sequence of rollback we can perform on various tables.
Generally, even though there are dependencies between tables when
loading data in, when rolling back, tables can all be rolled out without
a specific sequence as the rolling back is based on the record insert
and/or insert time. However, the proposed rolling back sequence is
useful as it ensures that

-   a user can always reproduce the result of loading for validation
    purpose if needed, and

-   useful information doesn’t disappear before consumed (e.g., the
    FILE_PATH records won’t be removed until the file is removed.)

## Rollback Process Step-by-Step in Snowflake (Insert & Update)

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio2751861355076076581"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio2751861355076076581"
class="ap-content">

</div>

</div>

1.  Having been informed about which data feed on which day the content
    containing an error, a user identify the **RunIds **of pipelines
    (incl. **hyper pipelines and sub-pipelines** for both data lake and
    data warehouse load) associated to this batch from the ADF Monitor
    logs. Use the hyper pipeline RunId to ensure all sub-pipeline RunIds
    are associated with one batch run. This can be done in ADF Monitor
    by eyeballing or run Kusto query in log analytics workspace.

2.  Identify the affected table(s) that require a rollback.

    1.  If we’re going to roll back records in a curated table, since a
        data feed links to one curated table directly, can
        quickly **identify the curated table** based on the data feed
        name.

    2.  If we’re going to roll back records in a Data Vault table, use
        the affected curated table name, query the Data Vault dependency
        rules stored in the cosmosDB, and **identify a list of Data
        Vault tables** affected by the data feed.

    3.  If we’re going to roll back records in a star schema table (e.g.
        dimension, bridge, fact), use the affected data vault or curated
        table name(s), query the star schema dependency rules stored in
        the cosmosDB, and **identify a list of Star Schema
        tables** affected by the data feed.

3.  For each identified affected table, query the table by
    the **RunId **of the sub-pipeline and identify the **earliest time
    among all RECORD_INSERT_DATETIME and RECORD_UPDATE_DATETIME**, the
    number of records to delete, and the number of records to revert the
    update.

4.  **Run Time Travel** statement in snowflake to go back **1
    millisecond earlier** than the minimum of RECORD_INSERT_DATETIME and
    RECORD_UPDATE_DATETIME together.

    1.  <div class="code panel pdl" style="border-width: 1px;">

        <div class="codeContent panelContent pdl">

        ``` java
        # example code to create a restored_table with history of a specific time from my_table 
        create table restored_table clone my_table
          at(timestamp => 'Mon, 09 May 2015 01:01:00 +0300'::timestamp_tz);
        ```

        </div>

        </div>

5.  **Validate **the restored_table contains the correct records, e.g.
    removed expected number of records, revert the expected number of
    records.

6.  If validation is successful, **rename **the restored_table to the
    original table name. This operation will replace the original table
    with a past snapshot and continue the evolvement of the table.
    Because the restoration essentially is to **create a new
    version** and load it on top of the original table, a user **can
    still go back to history** using time travel to access anything
    before the replacement, as long as the history is within the
    retention period.

    1.  <div class="code panel pdl" style="border-width: 1px;">

        <div class="codeContent panelContent pdl">

        ``` java
        # can use show table history to view table status change
        show tables history;
        ```

        </div>

        </div>

**Note**: Rollback doesn’t depend on the pattern of load (i.e. delta,
full, transaction) but only rely on the time to travel back to.

## Rollback Process Step-by-Step in Snowflake (Data Vault, Insert Only)

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio8159663105301920789"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio8159663105301920789"
class="ap-content">

</div>

</div>

1.  Having been informed about which data feed on which day the content
    containing an error, a user identify the **RunIds **of pipelines
    (incl. **hyper pipelines and sub-pipelines** for both data lake and
    data warehouse load) associated to this batch from the ADF Monitor
    logs. Use the hyper pipeline RunId to ensure all sub-pipeline RunIds
    are associated with one batch run.

2.  Since a data feed links to one curated table, query the Data Vault
    dependency rules stored in runtime config in the cosmosDB
    and **identify a list of Data Vault tables** affected by the data
    feed.

3.  If a Data Vault table has the Insert and Update operations (e.g.
    Satellite), refer to the <a
    href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/498270277/Data+Lake+and+Data+Warehouse+Rollback+and+Reload#Rollback-Process-Step-by-Step-in-Snowflake"
    rel="nofollow"><strong>Rollback Process in Snowflake (Insert &amp;
    Update)</strong></a> section. For Insert Only tables (e.g. Hub and
    Link), query the table by the **RunId **of the Data Vault pipeline
    and identify the **earliest time among all RECORD_INSERT_DATETIME**,
    and the number of records to delete.

4.  **Run Time Travel** statement in snowflake to go back **1
    millisecond earlier** than the minimum of RECORD_INSERT_DATETIME.

5.  **Validate **the restored_table contains the correct records, e.g.
    removed expected number of records.

6.  If validation is successful, **rename **the restored_table to the
    original table name. This operation will replace the original table 

## Rollback Process Step-by-Step in Storage Account

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio4708603027345017706"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio4708603027345017706"
class="ap-content">

</div>

</div>

1.  Having been informed about which data feed on which day the content
    containing an error, a user identify the **RunIds **of pipelines
    (incl. **hyper pipelines and sub-pipelines** for both data lake and
    data warehouse load) associated to this batch from the ADF Monitor
    logs. Use the hyper pipeline RunId to ensure all sub-pipeline RunIds
    are associated with one batch run.

2.  Since a data feed links to one curated table, query the curated
    table by the **RunId **of the curated pipeline and identify
    the **FILE_PATH**.

3.  **Locate **the file in the raw layer in the storage
    account, **confirm **it is the right file, and **move **it to a
    DELETED container with an appropriate lifetime management policy.
    Moving the file rather than delete immediately is for any further
    investigation.

# Reload

## Pre-requisite for Reload

Columns needed to support reload.

-   **RECORD_UPDATE**. This is the timestamp to reflect the time the
    records are last updated from a source system. It reflects that the
    records are valid as at time. It is critical when we process data
    from the supposed past windows during backfill and reload. If this
    field doesn’t exist in the data, we default to use the expected
    pipeline start time. **Note**: some tables contain this field in the
    data.

## Load the New Batch with Correction for Delta and Full

Audit columns needed (See the description in the <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/498270277/Data+Lake+and+Data+Warehouse+Rollback+and+Reload#Pre-requisite-for-Rollback"
data-card-appearance="inline"
rel="nofollow">https://vmia.atlassian.net/wiki/spaces/SDI/pages/498270277/Data+Lake+and+Data+Warehouse+Rollback+and+Reload#Pre-requisite-for-Rollback</a>):

-   **INSERT_RUN_ID**

-   **UPDATE_RUN_ID**

-   **RECORD_INSERT_DATETIME**

-   **RECORD_UPDATE_DATETIME**

Load as per usual - loading the current batch from adding the new file
in the raw layer of the storage account, to loading data in all the
affected tables in Snowflake.

1.  based on the primary key, identify a record is a new or existing

    1.  if new,

        1.  insert the record with the current INSERT_RUN_ID and
            RECORD_INSERT_DATETIME. UPDATE. UPDATE_RUN_ID and
            RECORD_UPDATE_DATETIME are NULL if applicable.

    2.  if existing (for Insert and Update tables),

        1.  identify if any attribute has updates based on the hash
            value of all attributes.

            1.  if has update,

                1.  insert the records with the current INSERT_RUN_ID
                    and RECORD_INSERT_DATETIME. UPDATE. UPDATE_RUN_ID
                    and RECORD_UPDATE_DATETIME are NULL.

                2.  end date the existing record and fill UPDATE_RUN_ID
                    and RECORD_UPDATE_DATETIME with the current RunId
                    and timestamp.

            2.  if no update,

                1.  do nothing.

## Load the New Batch with Correction for Transaction

Audit columns needed (See the description in the <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/498270277/Data+Lake+and+Data+Warehouse+Rollback+and+Reload#Pre-requisite-for-Rollback"
data-card-appearance="inline"
rel="nofollow">https://vmia.atlassian.net/wiki/spaces/SDI/pages/498270277/Data+Lake+and+Data+Warehouse+Rollback+and+Reload#Pre-requisite-for-Rollback</a>):

-   **INSERT_RUN_ID**

-   **RECORD_INSERT_DATETIME**

As for transaction data, each record has a transaction ID uniquely
identifying a transaction. When some erroneous attributes get corrected
in the new batch, based on the transaction ID, the reload process should
be:

1.  based on the transaction ID, identify if a record is a new or
    existing

    1.  if new,

        1.  insert the record with the current INSERT_RUN_ID and
            RECORD_INSERT_DATETIME.

    2.  if existing,

        1.  if apply delta load method

            1.  insert the records with the current INSERT_RUN_ID and
                RECORD_INSERT_DATETIME. UPDATE. UPDATE_RUN_ID and
                RECORD_UPDATE_DATETIME are NULL.

            2.  end date the existing record and fill UPDATE_RUN_ID and
                RECORD_UPDATE_DATETIME with the current RunId and
                timestamp.

        2.  if apply transaction load method

            1.  delete the bad record, and

            2.  insert the record with the current INSERT_RUN_ID and
                RECORD_INSERT_DATETIME.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Untitled
Diagram.svg.tmp](attachments/498270277/505741313.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Untitled
Diagram.svg.tmp](attachments/498270277/505774081.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Untitled
Diagram.svg.tmp](attachments/498270277/505741322.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Untitled
Diagram.svg.tmp](attachments/498270277/505806849.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Untitled
Diagram.svg.tmp](attachments/498270277/505708545.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/559349784.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/556138684.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/559284243.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/556138693.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/556171483.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/559218714.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/559153188.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/559054872.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/559218723.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_ADF.svg.tmp](attachments/498270277/559251467.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg](attachments/498270277/561545237.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg.png](attachments/498270277/559317117.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg](attachments/498270277/561774717.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg.png](attachments/498270277/561840211.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg](attachments/498270277/559153593.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg.png](attachments/498270277/561709374.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/561610894.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/561742006.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/561742015.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/561610903.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/559153395.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/561545402.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/561610912.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~rollback_storage_account.svg.tmp](attachments/498270277/561545397.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_storage_account.svg](attachments/498270277/559153425.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_storage_account.svg.png](attachments/498270277/561807577.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_storage_account.svg](attachments/498270277/561611200.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_storage_account.svg.png](attachments/498270277/561742251.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg](attachments/498270277/561709521.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg.png](attachments/498270277/559055487.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561611172.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561611182.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561545665.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561840475.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561709434.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561840489.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561807820.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561545687.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/561676692.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_storage_account.svg.tmp](attachments/498270277/559218947.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_storage_account.svg](attachments/498270277/559153404.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_storage_account.svg.png](attachments/498270277/561709223.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DV_Insert_Only.svg.tmp](attachments/498270277/559219221.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DV_Insert_Only.svg.tmp](attachments/498270277/561709456.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DV_Insert_Only.svg.tmp](attachments/498270277/561774917.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DV_Insert_Only.svg](attachments/498270277/559055526.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DV_Insert_Only.svg.png](attachments/498270277/561709585.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561807916.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/559219315.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561840547.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561676782.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561774989.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561709503.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561840561.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561840571.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/559055477.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~rollback_ADF.svg.tmp](attachments/498270277/561676304.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg](attachments/498270277/574849286.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg.png](attachments/498270277/567116247.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561840606.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561545795.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/559219351.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561676824.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561709563.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561709572.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561840619.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561742331.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/561545816.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DV_Insert_Only.svg.tmp](attachments/498270277/559219329.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DV_Insert_Only.svg](attachments/498270277/574882065.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DV_Insert_Only.svg.png](attachments/498270277/567116261.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561808020.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561742402.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561840715.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561775064.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561676859.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561545910.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/559153833.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561808029.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561742413.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~Rollback_Sequence.svg.tmp](attachments/498270277/561840636.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg](attachments/498270277/559055580.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg.png](attachments/498270277/559317637.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg](attachments/498270277/564887589.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg.png](attachments/498270277/564527194.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg](attachments/498270277/564592794.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg.png](attachments/498270277/564494476.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg](attachments/498270277/564527364.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg.png](attachments/498270277/564560088.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564396241.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564592836.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564461773.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564887729.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564396259.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg](attachments/498270277/564396269.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg.png](attachments/498270277/564363420.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564592850.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564428954.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564560102.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/564887743.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Rollback_Sequence.svg.tmp](attachments/498270277/561840732.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg](attachments/498270277/561742421.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Rollback_Sequence.svg.png](attachments/498270277/561709639.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg](attachments/498270277/556105907.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[rollback_ADF.svg.png](attachments/498270277/559218732.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DV_Insert_Only.svg](attachments/498270277/561774922.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DV_Insert_Only.svg.png](attachments/498270277/559153665.png)
(image/png)  

</div>
