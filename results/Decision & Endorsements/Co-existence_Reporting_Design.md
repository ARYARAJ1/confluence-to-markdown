# Co-existence Reporting Design

<div>

<div>

OUTCOME DECIDED Add your comments directly to the page. Include links to
any relevant research, data, or feedback.

</div>

</div>

<div class="plugin-tabmeta-details">

<div class="table-wrap">

|              |                                                                                                                                                                                                                                                                                                                                                                                                                       |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Status       | OUTCOME DECIDED                                                                                                                                                                                                                                                                                                                                                                                                       |
| Impact       | MEDIUM                                                                                                                                                                                                                                                                                                                                                                                                                |
| Driver       | Jason Dang (Deactivated) h.lindsaysmith@vmia.vic.gov.au Lindsay-Smith Ana Voinou (Deactivated)                                                                                                                                                                                                                                                                                                                        |
| Approver     | Hendry Susilo Maria Mota (Deactivated)                                                                                                                                                                                                                                                                                                                                                                                |
| Contributors |                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Informed     | Lasath Kahingala                                                                                                                                                                                                                                                                                                                                                                                                      |
| Due date     | 03/07/2020                                                                                                                                                                                                                                                                                                                                                                                                            |
| Outcome      | VMIA agreed that Option 1 is the best option moving forward which sees reporting being produced out of the strategic data platform. The implication of going with Option 1 is that for some reports the full data set may not be available during co-existence. In those instances Option 2 will be considered for some reports depending on complexity and importance which will be decided on a case by case basis. |

</div>

</div>

## Background

As data (such as client list) is migrated and mastered from legacy
systems (Risk Console/Client Connect) into PEGA, there is a requirement
to ensure that any business reports dependent on this data are still
accurate and representative of the performance of the whole business.
This decision explores potential design options to mitigate the impact
on downstream reports as data is being migrated.

### Current state dataflows

As a point of reference the existing dataflows identified in the
analysis undertaken are represented in the diagram below.

<img src="attachments/8978505/52101171.png?width=544"
class="image-center" loading="lazy"
data-image-src="attachments/8978505/52101171.png" data-height="545"
data-width="1054" data-unresolved-comment-count="0"
data-linked-resource-id="52101171" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20200629-214244.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="8978505"
data-linked-resource-container-version="54"
data-media-id="4d9a6e15-e057-4785-85b4-0ce989578ade"
data-media-type="file" width="544" />

An extensive analysis and assessment (9 meetings over a week) was
undertaken to identify all existing reports that will be impacted during
co-existence, broken down by release based on the functionality that
PEGA will drop in each release.  

The following table summarises the findings:

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Source System</strong></p></th>
<th class="confluenceTh"><p><strong>Impacted reports per
release</strong></p></th>
<th class="confluenceTh"><p><strong>Complexity and
importance</strong></p></th>
<th class="confluenceTh"><ul>
<li><p><strong>Solution Considerations</strong></p></li>
</ul></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>SAS</p></td>
<td class="confluenceTd"><ul>
<li><p>16 reports affected for Rel 1 &amp; 2. Of these reports:</p>
<ul>
<li><p>10 reports relate to claims using different aggregations and
filters</p></li>
<li><p>3 reports relate to CRM data out of RC and will potentially be
handled by PEGA</p></li>
<li><p>1 is a mailing list</p></li>
<li><p>1 Cognos report only formatted by SAS</p></li>
<li><p>2 claim transaction reports one per month and YTD
respectively</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Complexity:</p>
<ul>
<li><p>7 reports small</p></li>
<li><p>9 reports medium (contain some aggregations)</p></li>
</ul></li>
<li><p>The claim transactions reports may join to TechOne</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Actuary team should continue to receive data from RC and PEGA to
use for actuary purposes</p></li>
<li><p>Business will have 2 different mailing list extracts and will
have to merge them manually</p></li>
<li><p>Reports using similar datasets and different filters may be
consolidated into a single report</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Cognos</p></td>
<td class="confluenceTd"><ul>
<li><p>7 reports affected for Rel 1 &amp; 2. Of these reports:</p>
<ul>
<li><p>5 relate to payments</p></li>
<li><p>1 to collections</p></li>
<li><p>1 to premium</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>These are mainly finance reports around claim payments and
premium invoiced. There will be further workshops between PEGA and
finance team for Release 2 to identify the scope and the functionality
provided directly in PEGA for these</p></li>
<li><p>Reports have small to medium complexity</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Data from PEGA and RC needs to be merged</p></li>
<li><p>Business will have 2 different report formats for some reports,
one for Cognos and one for migrated data</p></li>
<li><p>Reports using similar datasets and different filters may be
consolidated into a single report</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Tableau</p></td>
<td class="confluenceTd"><ul>
<li><p>2 dashboards sourcing from RC via SAS into Tableau</p></li>
<li><p>Reports relate to tickets</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Medium complexity due to multiple extracts used to build each
dashboard</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Datasets either to be merged or complex joined and logic to be
added to Tableau reports</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Adhoc Risk Console reports</p></td>
<td class="confluenceTd"><ul>
<li><p>5 reports (Westpac load file, certificate of currency, policies
to endorsements and policy schedule, invoice premium)</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>These reports should be operational reports produced by
PEGA</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Business will have 2 different report formats for some reports,
one for RC and one for migrated data</p></li>
<li><p>Reports using similar datasets and different filters may be
consolidated into a single report</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Adhoc Client Connect reports</p></td>
<td class="confluenceTd"><ul>
<li><p>3 types of adhoc report requests with several permutations of the
same data with different filters. Subject areas include:</p>
<ul>
<li><p>Active Clients</p></li>
<li><p>Contacts - Renewals</p></li>
<li><p>Tickets</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Queries simple but according to user there are around 300 active
queries using various filters and combinations of the same data</p></li>
<li><p>Requests come in waves but could be a few requests per
month</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Data may be refreshed in Client Connect via feed from PEGA. In
that case reports will not need remediation</p></li>
<li><p>Reports using similar datasets and different filters may be
consolidated into a single report</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>IDR2</p></td>
<td class="confluenceTd"><ul>
<li><p>Currently 4 existing reports</p></li>
<li><p>A fifth report is currently build</p></li>
<li><p>Reports difficult to use due to complexity and lack of
visualisation tool (Tableau) for VMIA clients</p></li>
<li><p>1 python script that calculates PL Taxonomy and may potentially
be affected if PEGA doesn’t include that functionality</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Reports (Insurance Profile and Claims Profile) are still being
rolled out to clients but some clients sites still don’t have access to
Tableau on client side. This should be potentially available in the next
3-6 months</p></li>
<li><p>Reports contain sensitive data and need rework so can’t be use as
they are</p></li>
<li><p>Rest of reports not relevant for Release 1 &amp; 2 (Health
related)</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Existing data marts may be able to cater for producing existing
SAS reports that were migrated to IDR2 due to other reporting
priorities</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Total</p></td>
<td class="confluenceTd"><p>30 reports</p></td>
<td class="confluenceTd"><ul>
<li><p>More that half of the reports are a similar extract with
different filters</p></li>
<li><p>Many of the reports especially around payments and CRM data still
to be determined if source directly via PEGA</p></li>
</ul></td>
<td class="confluenceTd"></td>
</tr>
</tbody>
</table>

</div>

## Relevant data

The information below provides details of the design on the proposed
options.

**Option 1: Reporting out of the Data Warehouse on the Strategic Data
Platform**

This option would involve accelerating the development of the Data
Warehouse portion of the strategic Data Platform. The data marts and
star schemas will be reused or remodeled and enhanced based on the VMIA
business processes. Existing reports that are currently sourced from
IDR2, SAS, Cognos or directly from Risk Console or Client Connect will
be rebuild using the new Data Platform.

For the co-existence, logic will be incorporated in the integration
layer to combine data from legacy and PEGA BIX. Gradually data will be
sourced more and more from the PEGA BIX channel until the data migration
project is completed. The migration metadata will be used to know which
data in reports should be sourced from PEGA BIX and which from the
legacy sources . The above solution would have the following steps:

-   Create / adapt star schemas to existing / improved business
    processes due to use of the new PEGA application

-   Create strategic acquisition pipelines to load data into the
    platform

-   Create strategic ETL pipelines with logic that combines the legacy
    and PEGA data for load into the new data warehouse structures

-   Creation of reports out of the new Data platform

The dotted lines on the diagram below describe the data flows during
co-existence.

<img src="attachments/8978505/41256786.png?width=544"
class="image-center" loading="lazy"
data-image-src="attachments/8978505/41256786.png" data-height="566"
data-width="1074" data-unresolved-comment-count="0"
data-linked-resource-id="41256786" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20200629-060038.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="8978505"
data-linked-resource-container-version="54"
data-media-id="3ab153f2-af4f-4e8b-8699-065654619b84"
data-media-type="file" width="544" />

**Option 2: Report from Data lake but rebuild reports to not use SAS**

This option would involve rebuilding the SAS reports outside SAS using
data directly from the Data Lake. Data flows will be developed to
combine the relevant data from RC and PEGA and recreate the reports in
the required format.

Data extracts will still be sent to SAS only for use of the actuary
team. Reports will be created containing consolidated PEGA and legacy
data.

This process would required the following data flows to be developed:

-   Acquisition and Ingestion of Risk Console and Client Connect data
    into the raw layer of the data platform

-   Acquisition and Ingestion of PEGA BIX data into the Data Lake of the
    data platform

-   Creation of data flows to combine and prepare data for the extracts

The dotted lines on the diagram below describe the data flows during
co-existence.

<img src="attachments/8978505/52035859.png?width=544"
class="image-center" loading="lazy"
data-image-src="attachments/8978505/52035859.png" data-height="558"
data-width="1068" data-unresolved-comment-count="0"
data-linked-resource-id="52035859" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20200629-232150.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="8978505"
data-linked-resource-container-version="54"
data-media-id="0d56de91-226c-4775-9256-c992fed1e85c"
data-media-type="file" width="544" />

**Option 3: Use interim common table structures to combine legacy and
PEGA data**

As per option 1, this would require legacy and PEGA data to be ingested
into the Data Platform. A number of common data structures would be
created that including all necessary fields for the existing reports
from legacy. Data from PEGA for the migrated clients will be merged into
these structures. Once the data is combined, extracts could be created
that feed into IDR2 and SAS similar to the old extract feeds. SAS and
IDR2 can be re-pointed to use the new extracts instead of the current
sources and all reporting out of these systems will stay intact. The
migration metadata will be used to know which data in reports should be
sourced from PEGA BIX and which from the legacy sources. This option
will require the following data flows:

-   Strategic acquisition and Ingestion of Risk Console and Client
    Connect data into the raw layer of the data platform as required by
    the old data extracts

-   Acquisition and Ingestion of PEGA BIX data into the data platform

-   ETL processes to combine the data and load into the common data
    structures

-   Creation of extracts to simulate the previous legacy extracts

-   Re-pointing of new extracts to IDR2 and SAS to be used as a source
    now

Once the data flows above have been developed and a common data
structure has been populated, this will then act as the new data source
for downstream reports allowing VMIA to maintain visibility of their
clients across both systems.

The dotted lines on the diagram below describe the data flows during
co-existence.

<img src="attachments/8978505/50463055.png?width=544"
class="image-center" loading="lazy"
data-image-src="attachments/8978505/50463055.png" data-height="567"
data-width="1065" data-unresolved-comment-count="0"
data-linked-resource-id="50463055" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20200629-064923.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="8978505"
data-linked-resource-container-version="54"
data-media-id="88cb93c3-2fe5-4096-805e-a29a2e3116d0"
data-media-type="file" width="544" />

**Option 4: Report on Data Lake and IDR2 and no SAS**

This option would involve rebuilding the SAS reports outside SAS using
data ingested into IDR2. New data flows would be developed sending the
PEGA BIX data from the data platform to IDR2. New SSIS load packages
would be developed to combine the relevant data from RC and PEGA and
populate the star schemas appropriately. Any additional fields needed to
enable the functionality of the reports would be added to the IDR2
solution. Reports will be then recreated out of IDR2 in the required
format.

Data extracts will still be sent to SAS only for use of the actuary
team. Reports will be created containing consolidated PEGA and legacy
data.

This process would required the following data flows to be developed:

-   Acquisition and Ingestion of Risk Console and Client Connect data
    into the raw layer of the data platform

-   Acquisition and Ingestion of PEGA BIX data into the Data Lake of the
    data platform

-   Development of SSIS packages to load new data into IDR2 staging

-   Creation of SSIS packages to combine and load data into the star
    schemas

-   Creation of new reports out of IDR2

The dotted lines on the diagram below describe the data flows during
co-existence.

<img src="attachments/8978505/50070287.png?width=544"
class="image-center" loading="lazy"
data-image-src="attachments/8978505/50070287.png" data-height="563"
data-width="1069" data-unresolved-comment-count="0"
data-linked-resource-id="50070287" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="image-20200629-080734.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="8978505"
data-linked-resource-container-version="54"
data-media-id="e085dedb-6d86-4193-a170-1331e9cdaefa"
data-media-type="file" width="544" />

## Options considered

<div class="table-wrap">

<table class="confluenceTable" data-layout="wide">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Option 1: Reporting out of the Data
Warehouse on the Strategic Data Platform</strong></p></th>
<th class="confluenceTh"><p><strong>Option 2: Report from Data lake but
rebuild reports to not use SAS</strong></p></th>
<th class="confluenceTh"><p><strong>Option 3: Use interim common table
structures to combine legacy and PEGA data</strong></p></th>
<th class="confluenceTh"><p><strong>Option 4 Report on Data Lake and
IDR2 and no SAS</strong></p></th>
</tr>

<tr class="odd">
<th class="confluenceTh"><p>Description</p></th>
<td class="confluenceTd"><p>This option would involve accelerating the
development of the Data Warehouse portion of the strategic Data
Platform. The Data marts and star schemas will be reused or remodeled
and enhanced based on the VMIA business processes. Existing reports that
are currently sourced from IDR2, SAS, Cognos or directly from Risk
Console or Client Connect will be rebuild using the new Data
Platform.</p>
<p>For the co-existence, logic will be incorporated in the integration
layer to combine data from legacy and PEGA BIX. Gradually data will be
sourced more and more from the PEGA BIX channel until the data migration
project is completed. The migration metadata will be used to know which
data in reports should be sourced from PEGA BIX and which from the
legacy sources. The above solution would have the following steps:</p>
<ul>
<li><p>Create / adapt star schemas to existing / improved business
processes due to use of the new PEGA application</p></li>
<li><p>Fast track strategic data acquisition patterns</p></li>
<li><p>Create ETL pipelines with logic that combines the legacy and PEGA
data for load into the new data warehouse structures</p></li>
<li><p>Creation of reports out of the new Data platform</p></li>
</ul></td>
<td class="confluenceTd"><p>This option would involve rebuilding the SAS
reports outside SAS using data directly from the Data Lake. Data flows
will be developed to combine the relevant data from RC and PEGA and
recreate the reports in the required format.</p>
<p>Data extracts will still be sent to SAS only for use of the actuary
team. Reports will be created containing consolidated PEGA and legacy
data.</p>
<p>This process would required the following data flows to be
developed:</p>
<ul>
<li><p>Acquisition and Ingestion of Risk Console and Client Connect data
into the raw layer of the data platform</p></li>
<li><p>Acquisition and Ingestion of PEGA BIX data into the Data Lake of
the data platform</p></li>
<li><p>Creation of data flows to combine and prepare data for the
extracts</p></li>
</ul></td>
<td class="confluenceTd"><p>This option would require legacy and PEGA
data to be ingested into the Data Platform separately. A common data
structure would be created that includes all necessary fields for
reporting from legacy. Data from PEGA for the migrated clients would be
merged into these structures. Once the data is combined, extracts could
be created that feed into IDR2 and SAS similar to the old extract feeds.
SAS and IDR2 can be re-pointed to use the new extracts instead and all
reporting out of these systems will stay intact. This option will
require the following data flows:</p>
<ul>
<li><p>Acquisition and Ingestion of Risk Console and Client Connect data
into the raw layer of the data platform as required by the old data
extracts</p></li>
<li><p>Acquisition and Ingestion of PEGA BIX data into the data
platform</p></li>
<li><p>ETL processes to combine the data and load into the common data
structures</p></li>
<li><p>Creation of extracts to simulate the previous legacy
extracts</p></li>
<li><p>Re-pointing of new extracts to IDR2 and SAS to be used as a
source now</p></li>
</ul>
<p>Once the data flows above have been developed and a common data
structure has been populated this will then act as the new data source
for downstream reports allowing VMIA to maintain visibility of their
clients across both systems.</p></td>
<td class="confluenceTd"><p>This option would involve rebuilding the SAS
reports outside SAS using data ingested into IDR2. New data flows would
be developed sending the PEGA BIX data from the data platform to IDR2.
New SSIS load packages would be developed to combine the relevant data
from RC and PEGA and populate the star schemas appropriately. Any
additional fields needed to enable the functionality of the reports
would be added to the IDR2 solution. Reports will be then recreated out
of IDR2 in the required format.</p>
<p>Data extracts will still be sent to SAS only for use of the actuary
team. Reports will be created containing consolidated PEGA and legacy
data.</p>
<p>This process would required the following data flows to be
developed:</p>
<ul>
<li><p>Acquisition and Ingestion of Risk Console and Client Connect data
into the raw layer of the data platform</p></li>
<li><p>Acquisition and Ingestion of PEGA BIX data into the Data Lake of
the data platform</p></li>
<li><p>Development of SSIS packages to load new data into IDR2
staging</p></li>
<li><p>Creation of SSIS packages to combine and load data into the star
schemas</p></li>
<li><p>Creation of new reports out of IDR2</p></li>
</ul></td>
</tr>
<tr class="even">
<th class="confluenceTh"><p>Pros and cons</p></th>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /></p>
<ul>
<li><p>No technical debt or throw away work</p></li>
<li><p>SAS decommissioned as an ETL tool, bringing mappings and business
logic into the platform</p></li>
<li><p>Strategic reporting capability is available in the shortest
possible time frame</p></li>
</ul>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /></p>
<ul>
<li><p>It will not be possible to deliver a full data warehouse solution
in 3 months, so there will be a period of 3+ months where existing
reports loose sight of migrated clients and there are no reports
available for the migrated clients</p></li>
</ul></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /></p>
<ul>
<li><p>SAS decommissioned as an ETL tool, bringing mappings and business
logic into the platform</p></li>
<li><p>Quick time to deliver as metric definition can be done in
reporting tool</p></li>
</ul>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /></p>
<ul>
<li><p>Business logic for metrics locked up in reporting tool</p></li>
<li><p>Strategic data warehouse delayed</p></li>
<li><p>Re-work will be needed to re-point resources once the strategic
solution is finished and there maybe no driver for this</p></li>
</ul></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /></p>
<ul>
<li><p>High likelihood there will be no reporting discontinuity as
cohorts are migrated to PEGA</p></li>
</ul>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /></p>
<ul>
<li><p>ETL to merge datasets will not be reusable for other reports and
business logic remains in SAS</p></li>
<li><p>Re-work will be needed to re-point resources once the strategic
solution is finished and there maybe no driver for this</p></li>
<li><p>Strategic data warehouse delayed</p></li>
</ul></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /></p>
<ul>
<li><p>SAS that can use IDR are pointed to a common data model</p></li>
</ul>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /></p>
<ul>
<li><p>More tech debt on legacy on-premise environment</p></li>
<li><p>Strategic Data warehouse delayed</p></li>
</ul></td>
</tr>
<tr class="odd">
<th class="confluenceTh"><p>Estimated effort</p></th>
<td class="confluenceTd"><p>MEDIUM</p></td>
<td class="confluenceTd"><p>MEDIUM</p></td>
<td class="confluenceTd"><p>MEDIUM</p></td>
<td class="confluenceTd"><p>MEDIUM</p></td>
</tr>
</tbody>
</table>

</div>

## Action items

-   

## Outcome

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-055315.png](attachments/8978505/11370553.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-063746.png](attachments/8978505/11468919.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-225850.png](attachments/8978505/13303809.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-231405.png](attachments/8978505/11305184.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-231936.png](attachments/8978505/11337945.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-231953.png](attachments/8978505/11337951.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-232016.png](attachments/8978505/13434881.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-232600.png](attachments/8978505/13434900.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-232616.png](attachments/8978505/11469089.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-232618.png](attachments/8978505/13303817.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200611-233217.png](attachments/8978505/11305198.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-060038.png](attachments/8978505/41256786.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-064923.png](attachments/8978505/50463055.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-075832.png](attachments/8978505/50201118.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-075936.png](attachments/8978505/50463092.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-080151.png](attachments/8978505/50201124.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-080409.png](attachments/8978505/50168437.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-080607.png](attachments/8978505/44041047.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-080637.png](attachments/8978505/50201144.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-080734.png](attachments/8978505/50070287.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-214244.png](attachments/8978505/52101171.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-215108.png](attachments/8978505/52035627.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-232108.png](attachments/8978505/52428861.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20200629-232150.png](attachments/8978505/52035859.png)
(image/png)  

</div>
