# Ingestion Logging (Data Source/ Data Feeds)

Pipeline runs in ADF for ingestion feeds will have logs stored in Cosmos
DB.  

### Ingestion Log Keys

<div class="table-wrap">

|                      |                                                                                                                                                                                                                                     |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Key**              | **Description**                                                                                                                                                                                                                     |
| `pipelineName`       | Name of pipeline that was run (child pipeline)                                                                                                                                                                                      |
| `RunId`              | ID of child pipeline that was run                                                                                                                                                                                                   |
| `isIngestion`        | Default value set to `true` as the logging pipeline is only used for ingestion feeds. As both ingestion and data warehouse logs are stored in the same container, this key allows us to determine that it is an ingestion feed log. |
| `forcedSuccess`      | If set to `true` , a forced success has occurred with the value being passed down from an activity in the pipeline run.                                                                                                             |
| `tableName`          | Name of table within the data source/ data feed                                                                                                                                                                                     |
| `sourceSystem`       | Source system of data source/ data feed                                                                                                                                                                                             |
| `timestamp`          | Time of the run in UTC                                                                                                                                                                                                              |
| `windowEndTime`      | End time of the run in UTC                                                                                                                                                                                                          |
| `hyperPipelineRunId` | ID of the hyper pipeline run (parent pipeline)                                                                                                                                                                                      |

</div>

PARKED NOTES:

-   \[Parent page\]: Anything to do with logging can be found in Cosmos
    Observability DB

-   DWH deployment needed for datasets + linked services

-   Updates needed foy any ingestion tests that points to ADF

-   

  
TODO:

Data source → data feed

-   Parent page for log docs

    -   \[Section\] Structure of logs - JSON objects \*\*\* to be added
        to parent page for log docs

    -   \[Screenshot\] Ingestions vs DWH logging

    -   \[Section\]: How to amend keys in logs

-   Ingestion logging

    -   \[Summary\] Summary + problem statement for LA and rationale for
        transition + link for MS forums w/ same issue

    -   \[Location\] where is the repo

    -   \[Process flow map\] Process flow map of how pipeline run stores
        logs in cosmos DB

        -   \[Screenshot\] Example of ADF pipeline run w/ new logging
            pipeline

    -   \[Table\] Current keys in logs + definition

    -   \[Section\] making changes to log keys - include hyperlink for
