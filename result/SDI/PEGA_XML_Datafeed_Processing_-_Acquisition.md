# PEGA XML Datafeed Processing - Acquisition

This section contains a summary of Acquisition components as part of the
PEGA hyper pipeline, and the patterns it will follow to pull data from
SFTP and sink it in data lake.

PEGA BIX extracts are independent of their downstream tables into which
they eventually merge with, allowing data feed extracts to be processed
regardless of the underlying schema of the curated tables. PEGA BIX
publishes datasets in XML and an associated schema of both of these can
be received into the landing and raw layers without being effected by
schema changes. Below is an overview of the PEGA acquisition pipeline:

<img src="attachments/1262256154/1262419977.png" class="image-center"
loading="lazy" data-image-src="attachments/1262256154/1262419977.png"
data-height="513" data-width="821" data-unresolved-comment-count="0"
data-linked-resource-id="1262419977" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20211012-232717.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="1262256154"
data-linked-resource-container-version="2"
data-media-id="2c4d4671-9cf4-42e7-aff0-2b6b00848784"
data-media-type="file" />

#### Pre acquisition pipeline activities

1.  Upon commencing the pipeline, a parameter parsing function will
    consume the trigger parameters and create the required parameters
    for consumption of the daily file.

2.  Two checks will be performed before proceeding with the current
    day’s execution

a\. Check for unprocessed files from previous days in the data lake
will, and if found trigger an alert

b\. Check for missed files from previous days in the PEGA SFTP server,
and if found trigger an alert

#### SFTP Pull

-   SFTP pull pattern will pull all zip files for the given extract
    class from SFTP server based on the naming pattern and copy into the
    landing container of the data lake

-   A copy statement will unzip these files and store the contents in
    the landing unzipped container.

    -   If any of files that are unzipped are invalid, this step will
        fail and should trigger another copy step which removes the
        files from the folder in the landing container and moves them to
        an error directory for the extract class.

#### Integrity Check

-   The integrity check will then run on the file to be loaded for the
    given day to ensure the data is valid

    -   If the integrity check passes, files will be moved from the
        landing to raw layer.

    -   If the integrity check fails, files will be moved into the error
        folder in the data lake.

-   A check for multiple files for an extract will occur, if multiple
    files are found then an error will be raised and alerts will be
    sent.

#### Clean Up

-   Data is moved to the processed layer, and the extracts in the
    landing unzipped folder are deleted.

-   Files are deleted from the SFTP server after they have been
    processed in raw.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20211012-232717.png](attachments/1262256154/1262419977.png)
(image/png)  

</div>
