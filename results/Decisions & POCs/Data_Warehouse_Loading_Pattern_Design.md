# Data Warehouse Loading Pattern Design

# Introduction

When loading data from the curated layer to any data warehouse table,
the procedure becomes more bespoke due to specific tables and operations
required for each data warehouse table. To better manage and simplify
the procedure for many data warehouse tables, the following design
intends to decouple the procedure and extract generic and reusable
components.

<div class="toc-macro rbtoc1639539586667">

-   1 [Introduction](#DataWarehouseLoadingPatternDesign-Introduction)
-   2 [Pre-requisite](#DataWarehouseLoadingPatternDesign-Pre-requisite)
-   3 [Assumptions](#DataWarehouseLoadingPatternDesign-Assumptions)
-   4 [Technical
    Design](#DataWarehouseLoadingPatternDesign-TechnicalDesign)
    -   4.1 [Key Element
        Description](#DataWarehouseLoadingPatternDesign-KeyElementDescription)
    -   4.2 [Loading
        Process](#DataWarehouseLoadingPatternDesign-LoadingProcess)
-   5 [Name
    Convention](#DataWarehouseLoadingPatternDesign-NameConvention)

</div>

# Pre-requisite

Several **audit columns** are required in the **data warehouse tables**
(for the detail definition of the first four fields, please refer to <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/498270277/Data+Lake+and+Data+Warehouse+Rollback+and+Reload#Pre-requisite-for-Rollback"
data-card-appearance="inline"
rel="nofollow">https://vmia.atlassian.net/wiki/spaces/SDI/pages/498270277/Data+Lake+and+Data+Warehouse+Rollback+and+Reload#Pre-requisite-for-Rollback</a>):

-   **INSERT_RUN_ID** and **UPDATE_RUN_ID** are used to link to ADF
    pipelines.

-   **RECORD_INSERT_DATETIME** and **RECORD_UPDATE_DATETIME** are used
    for time travel in rollback.

-   **IS_DELETE_FLAG**. It is a field used to indicate if a record is
    deleted in the source system, both hard and soft delete. If the
    source system has an attribute used to capture soft deletion, in the
    data warehouse data, standardise it to using this field. This field
    also needs to exist in the **time-varying tables** at the curated
    layer (see rationale in the **Assumption** section, point 1).

-   **EXP_BATCH_DATETIME**. It is a field used to capture the expected
    process time if everything run on time. It is different from
    RECORD_INSERT_DATETIME or RECORD_UPDATE_DATETIME as when back
    filling, EXP_BATCH_DATETIME will be times in the past while
    RECORD_INSERT_DATETIME and RECORD_UPDATE_DATETIME will be the time
    when changes are made to a record during the loading process. This
    field is used to flag records that are inserted or updated in a
    batch and enable us to extract only those new rows (i.e. delta) for
    the given data warehouse batch load. This field also needs to exist
    in the **time-varying tables** at the curated layer (see rationale
    in the **Assumption** section, point 2). The field will be populated
    when insertion happens.

# Assumptions

-   **Inferred deletion** has been handled at the curated layer.
    IS_DELETE_FLAG is added in time-varying tables to indicate a record
    is deleted from the source system. Therefore, no inferred deletion
    needs to be handled in the data warehouse load.

-   **EXP_BATCH_DATETIME** exists and is populated properly in the
    **time-varying tables** and **data vault tables**. The column
    enables queries to only pick up the new changes rather a full
    snapshot of a particular period based on the SYSTEM_VALID_FROM and
    SYSTEM_VALID_TO.

# Technical Design

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio6044496505152003118"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio6044496505152003118"
class="ap-content">

</div>

</div>

## Key Element Description

-   The **Source Tables** are the starting point of the data warehouse
    loading process, and they depend on if we have a data vault besides
    a star schema consumption layer
    (see <a href="Data_Warehouse_Pipeline_-_3._Process.md"
    data-linked-resource-id="469860353" data-linked-resource-version="42"
    data-linked-resource-type="page">https://vmia.atlassian.net/wiki/spaces/SDI/pages/469860353/Data+Warehouse+Pipeline+-+3.+Process</a> for
    detail).

-   if we **have a data vault and a star schema consumption layer**,

    -   When** loading into the data vault**, the Source Tables are a
        number of** time-varying tables **from the curated layer.

    -   When **loading into the star schema**, the Source Tables are a
        number of** data vault tables**.

-   if we only have a star schema consumption layer

    -   When **loading into the star schema**, the Source Tables are a
        number of** time-varying tables **from the curated layer or a
        staging layer.

-   The **Transient Table **is an **Intermediary Table** that has a
    one-to-one relationship with a data warehouse table. It is
    a **transient **table in Snowflake existing across sessions (e.g.
    created by a stored procedure and used by another) and needs to be
    explicitly dropped. It’s different from a permanent table as it
    doesn’t have
    a <a href="https://docs.snowflake.com/en/user-guide/data-failsafe.html"
    rel="nofollow">Fail-safe</a> period (for recover data outside of
    time travel retention period) and therefore saves additional cost.
    It is used to hold the** delta snapshot of the current round** from
    all the joins and filtering on the Source Tables.

-   The **Data Warehouse Table** can be either a data vault table or a
    star schema table. Each Data Warehouse Table has one specific
    Transient Table of the delta snapshot of the current round. It is
    a transient table that contains all the **audit columns** mentioned
    in the <a
    href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/567148575/Data+Warehouse+Loading+Pattern+Design#Pre-requisite"
    rel="nofollow"><strong>Pre-requisite</strong></a>.

## Loading Process

1.  Check the **designation table** if the current batch has been
    **successfully ingested already** based on the
    **EXP_BATCH_DATETIME**. If yes, exit the stored procedure and the
    data warehouse pipeline gracefully. If no, continue to the
    subsequent steps. This is to handle for 1) **reload after
    rollback**, and 2) **backfill** when there are only a subset of
    tables that need to load into the data warehouse and the majority of
    tables which have already loaded don’t need to rerun the data
    warehouse loading process.

2.  A Lookup Activity in ADF triggers the **bespoke stored
    procedure** in Snowflake.

    1.  Based on the scheduled batch time, **only grab records with
        changes** from the **driving source table** using
        **EXP_BATCH_DATETIME**. Perform the required **joins and
        filtering** to form the **delta snapshot of the current
        execution.**

    2.  **Drop (if exist) and re-create **a **Transient Table** on the
        fly to store the **delta snapshot of the current round.** The
        reason not dropping the transient table at the end of the last
        execution is for easy debugging.

3.  The **generic stored procedure **in Snowflake is triggered by
    another Lookup Activity in ADF and perform a **type 2 load**.

    1.  Compare records in the Transformation Object and the Transient
        Table and identify any **new, existing and deleted
        records **(i.e. records exist in the Transformation Object but
        not in the Transient Table) based on the primary key.

    2.  Perform Insertion and/or updating.

        1.  for **new **records

            1.  **Insert **into the destination table and fill in the
                RECORD_INSERT_DATETIME, INSERT_RUN_ID and
                EXP_BATCH_DATETIME.

        2.  for **existing **records **with updates**

            1.  **End date** the existing record in the destination
                table and fill the RECORD_UPDATE_DATETIME, UPDATE_RUN_ID
                and EXP_BATCH_DATETIME.

            2.  **Insert **into the destination table and fill in the
                RECORD_INSERT_DATETIME and INSERT_RUN_ID.

        3.  for deleted records

            1.  Because deletion is already handled in the time-varying
                tables by indicating in the IS_DELETE_FLAG field,
                deleted records would be treated the same as attributes
                update in ii.

# Name Convention

<div class="table-wrap">

|                 |                          |                          |                                       |                                                                                                                                      |
|-----------------|--------------------------|--------------------------|---------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
|                 | **Database**             | **Schema**               | **Table Name**                        | **Rationale**                                                                                                                        |
| Transient Table | As per destination table | As per destination table | \<destination table name\>\_TRANSIENT | Add a suffix rather than a prefix is to organise those tables together with their associated destination table. It is easy to debug. |

</div>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/567148611.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/567148620.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/567083237.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/567083246.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/567181405.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/564527557.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/564560311.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/564429122.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/567214146.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/567017545.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_loading.svg.tmp](attachments/567148575/564494887.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/564494995.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/567083297.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/564429182.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/564560395.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/567181462.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/574914725.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/574849079.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/574980199.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/575045651.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_load_2.svg.tmp](attachments/567148575/574849095.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205aeb6e6b50c58aeca8b\~DW_load_2.svg.tmp](attachments/567148575/574980192.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_load_2.svg](attachments/567148575/574947427.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_load_2.svg.png](attachments/567148575/567116037.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/564495208.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/574914797.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/574849162.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/567116081.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/567116233.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/574849258.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/574980297.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/574882049.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/564495393.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/574882121.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/583303383.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/583368907.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/583139566.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/583041191.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/600637481.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/600637491.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/600899624.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/600571989.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/600080494.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/600572029.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/624657436.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/624722812.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624394397.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624623693.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624394407.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624328833.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624492650.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624459859.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624525413.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624492660.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/624394421.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~DW_loading.svg.tmp](attachments/567148575/567017540.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg](attachments/567148575/564527566.svg)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[DW_loading.svg.png](attachments/567148575/567214154.png) (image/png)  

</div>
