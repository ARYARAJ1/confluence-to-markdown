# ADF Hyper Pipeline - PEGA - Invalid BIX extracts handling

This section contains an overview of the different options on how the
ADF Pipeline for PEGA can be updated to allow for easier handling and
remediation of PEGA datafeeds when an invalid BIX extract has been
detected. Details of the interface agreement when an invalid file for
PEGA can be found at <a
href="https://vmia.atlassian.net/wiki/spaces/SDI/pages/636584046/Invalid+Schema+BIX+Extracts+Process+Handling"
data-linked-resource-id="636584046" data-linked-resource-version="3"
data-linked-resource-type="page">Invalid Schema BIX Extracts Process
Handling</a> .

# Options for handling the invalid file

The table below provides options on how an invalid BIX extract can be
handled. It is important to note that more than one option can be
implemented if required

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="odd">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Delete the file from the SFTP Server
as it is landed into the data lake</strong></p></th>
<td class="confluenceTd" data-highlight-colour="#f4f5f7"><p><strong>Only
remove the file from the SFTP Server if it has been processed all the
way to Raw</strong>DECIDED</p></td>
<td class="confluenceTh"><p><strong>Move the file into an error
container in the data lake if it is invalid</strong>DECIDED</p></td>
<td class="confluenceTh"><p><strong>Move the file into an error folder
in the SFTP Server if it is invalid</strong>DECIDED</p></td>
</tr>
<tr class="even">
<th class="confluenceTd"><p>Description</p></th>
<th class="confluenceTd"><p>As ADF copies the file from the SFTP Server
into the landing layer of the data lake, it will also delete the file
from the source.</p></th>
<td class="confluenceTd"><p>A seperate delete activity in the ADF
pipeline will be created and added to the end of the acquisition
pipeline once the file has been processed all the way to the raw
layer</p></td>
<td class="confluenceTd"><p>Once an invalid file has been detected, a
seperate activity will be added to the ADF Pipeline that will move the
file from the landing layer to the error container in the SDI Data
Lake</p></td>
<td class="confluenceTd"><p>Once an invalid file has been detected, a
seperate activity will be added to the ADF Pipeline that will move the
file from the original location to a seperate error folder in the SFTP
Server.</p></td>
</tr>
<tr class="odd">
<th class="confluenceTd"><p>Advantages</p></th>
<th class="confluenceTd"><ul>
<li><p>Reduces build-up of files on the SFTP Server</p></li>
</ul></th>
<td class="confluenceTd"><ul>
<li><p>Do not have to send the invalid file to Pega for review and
debugging as it is still available at the source</p></li>
<li><p>The invalid file will still be stored in a persistent storage
layer (SFTP) if there is any delay in remediating/reviewing the
file</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Can keep track of all invalid files in the SDI Data Lake</p></li>
<li><p>The invalid file will be stored in a persistent storage layer in
the data lake</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Can keep track of all invalid files in the PEGA SFTP
Server</p></li>
<li><p>Both the SDI and PEGA teams will be able to access the invalid
extracts for review and debugging</p></li>
<li><p>A remediated file can be created and stored in the original
location, as the old file has been moved</p></li>
</ul></td>
</tr>
<tr class="even">
<th class="confluenceTd"><p>Disadvantages</p></th>
<th class="confluenceTd"><ul>
<li><p>The SDI team will need to resend the file to the PEGA team
manually for review if an error is detected</p></li>
<li><p>If a file is invalid, it will not proceed to raw layer which is
the first persistent layer in the data lake. Hence, there is a risk that
the file may be lost if it is only stored in a transient storage
layer.</p></li>
</ul></th>
<td class="confluenceTd"><ul>
<li><p>Need a process to remove the file once the issue has been
remediated.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>PEGA team will not be able to access the invalid files in the
data lake</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Need a process that will clean up old invalid files that have
already been remediated</p></li>
<li><p>Need to update the ADF Pipeline to move the file from the
original location to an error folder on the SFTP Server</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

# Options for restarting the pipeline

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Update the ADF Pipeline to override
dependency checker and allow for processing of an out of sequence
file</strong></p></th>
<th class="confluenceTh"><p><strong>Create a seperate ADF Pipeline that
automatically renames the remediated file to the date the error had
occurred with an _2 sequence value</strong></p></th>
<th class="confluenceTh"><p><strong>Create a seperate ADF Pipeline that
automatically replaces the original invalid file (both with the same
date and sequence value)</strong>DECIDED</p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>Description</p></td>
<td class="confluenceTd"><p>Once Pega has uploaded a new file to the
SFTP server with the valid records, the pipeline will need to be
manually triggered to acquire and ingest the data into the SDI Data
Platform. As the previous day execution has not yet succeeded, the
dependency checker may need to be ignored for manually executed
pipelines in order for the pipeline to be restarted</p></td>
<td class="confluenceTd"><p>Once Pega has uploaded a new file to the
SFTP server with the valid records, there will need to be a new ADF
pipeline created that is able to rename the file to the original date
the initial invalid file contains with an _2 sequence value. The PEGA
data pipeline will then need to be manually triggered with custom values
(i.e. _2 sequence value). Once this pipeline execution has succeeded,
the tumbling window trigger can then be restarted.</p></td>
<td class="confluenceTd"><p>Once Pega has uploaded a new file to the
SFTP server with the valid records, there will need to be a new ADF
pipeline created that is able to rename the file to the same file name
that contain the invalid records (i.e. same date and sequence value)
assuming the original file no longer exists in the original location
(either removed or relocated). The specific triggered run where the
invalid file had occurred can then be rerun and if successful, the
tumbling window trigger can then be restarted.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Advantages</p></td>
<td class="confluenceTd"><ul>
<li><p>Does not require any change to BIX file names</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Will not overwrite existing invalid file</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Least amount of manual work when it comes to rerunning the failed
pipeline</p></li>
<li><p>In comparison to option 2, where a new pipeline run will appear
in the monitoring screen, this option will instead trigger a rerun of
the the triggered run, therefore making it easier to review a history of
triggered runs that were remediated in the ADF monitoring UI</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Disadvantages</p></td>
<td class="confluenceTd"><ul>
<li><p>May have to manually trigger more than 1 days worth of pipeline
runs before the tumbling window trigger can be restarted in order to
satisfy the previous day success execution dependency</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>A seperate ADF pipeline will need to be developed</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>A seperate ADF pipeline will need to be developed</p></li>
<li><p>Ideally the original invalid file should not still exist in the
original location to avoid overwriting files</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>
