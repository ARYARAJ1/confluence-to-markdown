# Interim use of risk console pipeline to load excel reference data

Until the final design can be prioritised for the reference data load,
the agreed decision is to use the existing Risk Console XLSX load
pattern for reference data management.

Alike RC XLSX load, we

-   Place the XLSX file in the K drive

-   Create config files - Deploy changes into COSMOS DB (Filename =
    Entity name = Table name)

    -   trigger config,

    -   data-feed config,

    -   table config

-   Generate create table from table config, Deploy in database

-   Generate triggers (manual and tumbling) and delete tumbling trigger
    for that.

-   Use manual trigger to load data from xlsx file into curated layer

-   make changes to the file and trigger again to test change data
    capture.

In future, when the Risk Console and the other legacy systems are
decommissioned, the RC pipeline would be repurposed for reference data
management specifically.

At that point we will have better visibility of the actual/exact changes
to be made in TDQ functions and the Hyper-pipeline to better suit the
load process.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20210910-033136.png](attachments/1180598310/1211826179.png)
(image/png)  

</div>
