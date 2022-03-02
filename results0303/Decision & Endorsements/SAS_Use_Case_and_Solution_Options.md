# SAS Use Case and Solution Options

# <u>Current State</u>

## Organisational Usage View

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio3144073424958488178"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio3144073424958488178"
class="ap-content">

</div>

</div>

## System Contextual View

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio6753553277266837706"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio6753553277266837706"
class="ap-content">

</div>

</div>

## Legacy System End of Life Plan

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Name</strong></p></th>
<th class="confluenceTh"><p><strong>Reaching End of
Life</strong></p></th>
<th class="confluenceTh"><p><strong>Comments</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>Risk Console;</p>
<p>Client Connect</p></td>
<td class="confluenceTd"><p>Yes</p></td>
<td class="confluenceTd"><p>Planned to be decommissioned second half of
2021</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>ST Synergy</p></td>
<td class="confluenceTd"><p>Yes</p></td>
<td class="confluenceTd"><p>As per current plan all data is planned to
be migrated from ST Synergy to Sales<em>f</em>orce by end of 2021 and ST
Synergy will be decommissioned (tbc with DBI)</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>TechOne</p></td>
<td class="confluenceTd"><p>No</p></td>
<td class="confluenceTd"></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>VELS &amp; VWA</p></td>
<td class="confluenceTd"><p>?</p></td>
<td class="confluenceTd"></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>RMA Online</p></td>
<td class="confluenceTd"><p>Yes</p></td>
<td class="confluenceTd"><p>time frame is unknown</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Delighted</p></td>
<td class="confluenceTd"><p>?</p></td>
<td class="confluenceTd"></td>
</tr>
</tbody>
</table>

</div>

## Purpose and source of data

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Team</strong></p></th>
<th class="confluenceTh"><p><strong>Purpose</strong></p></th>
<th class="confluenceTh"><p><strong>Source of data</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>Insights &amp; Analytics (I&amp;A)</p></td>
<td class="confluenceTd"><p>SAS is used to extract data from various
source systems and produce canned reports using predefined SAS programs.
These reports are delivered to various consumers across VMIA using two
delivery mechanism as:</p>
<p>a) Scheduled to run at a predefined date &amp; time and report is
sent to consumer via an email</p>
<p>b) Consumer login to SAS portal and run the report using predefined
parameters (Self-Serve).</p></td>
<td class="confluenceTd"><ol>
<li><p>Risk Console</p></li>
<li><p>Client Connect</p></li>
<li><p>VELS &amp; VWA (for DDWC - Dust Disease Workers
Compensation)</p></li>
<li><p>Delighted (for NPS)</p></li>
<li><p>RMA Online</p></li>
<li><p>TechOne</p></li>
</ol></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Pricing &amp; Capital Management</p></td>
<td class="confluenceTd"><p>SAS is used to connect to data sources,
query and bring data into a work environment where we can investigate,
do some transformations, do some analysis, produce result summary
datasets to use for something else etc.</p>
<p> </p>
<p>We use it for analysis/exploration/investigation.</p></td>
<td class="confluenceTd"><ol>
<li><p>VELS (for DDWC)</p></li>
<li><p>Risk Console</p></li>
<li><p>Client Connect</p></li>
<li><p>Saleforce</p></li>
<li><p>Data Lake</p></li>
<li><p>Files sitting on K drive (e.g. data received from
clients)</p></li>
</ol></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Domestic Building Insurance (DBI)</p></td>
<td class="confluenceTd"><p>SAS is used to produce a DBI Performance
report from ST Synergy system that list all claims matters (open +
closed since 2010) and their details including claim cause, costs
,payments and recoveries.</p>
<p>ST Synergy is the legacy DBI system and there are still claims that
are managed in this system. As at March’21, there are ~100 open
(ongoing) claims in ST Synergy. There is plan to migrate all claims to
salesforce by end of 2021</p></td>
<td class="confluenceTd"><ol>
<li><p>ST Synergy</p></li>
</ol></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Business Performance</p></td>
<td class="confluenceTd"><p>As part of TPAR (Taxable Payment Annual
Report) requirement of ATO, VMIA need to report all outgoing payments to
ATO annually. ATO has a predefined template in which VMIA has to report
this data to ATO. SAS brings all outgoing payments data from various
systems (listed here) and produces the report in the template specified
by ATO.</p></td>
<td class="confluenceTd"><ol>
<li><p>Risk Console</p></li>
<li><p>VELS (for DDWC)</p></li>
<li><p>Salesforce &amp; ST Syenrgy (for DBI)</p></li>
<li><p>TechOne</p></li>
</ol></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Insurance Services</p></td>
<td class="confluenceTd"><p>Insurance Questionnaire (IQ) consolidation
process which run each year to around end of February.</p>
<p>The process involves extracting data from IQ for current year and
comparing it to the IQ for the previous year (to highlight changes and
differences).</p></td>
<td class="confluenceTd"><p>RMA Online</p></td>
</tr>
</tbody>
</table>

</div>

## Solution Options

### Constraint:

1.  Hard deadline for SAS phase out (Sep 2022)

### Guiding Principals

1.  Ingestion of data sources in SDI is based on its alignment to value
    it provides to the strategic I&A needs.

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Option ID</strong></p></th>
<th class="confluenceTh"><p><strong>Option title</strong></p></th>
<th class="confluenceTh"><p><strong>Explain the option</strong></p></th>
<th class="confluenceTh"><p><strong>Pros &amp; Cons</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>1</p></td>
<td class="confluenceTd"><p>Natural phase</p></td>
<td class="confluenceTd"><p>Retain SAS until its naturally phased out.
Natural phase out will happen because of one of the following
reasons:</p>
<p>1. Legacy system that is feeding data in SAS is no longer in use at
VMIA (decommissioned) e.g. Risk Console, Client Connect</p>
<p>2. The business need for reports/data extract coming out of SAS is no
longer there. This will most likely happen due an alternate way to meet
the business need from Client 360 or SDI platform.</p>
<p><strong>Prerequisite</strong>: All the source systems that have been
identified as “feeding into SAS” (see list above), required to assessed
for candidacy for decommissioning</p></td>
<td class="confluenceTd"><p>Pros:</p>
<ul>
<li><p>No disruption to business</p></li>
</ul>
<p>Cons:</p>
<ul>
<li><p>Cost of ownership (License + Support)</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>2a</p></td>
<td class="confluenceTd"><p>Intervened phase out - Governed
approach</p></td>
<td class="confluenceTd"><p>Prioritise the ingestion of required sources
into SDI which follows all SDI ingestion and data orchestration
framework.</p></td>
<td class="confluenceTd"><p>Pros:</p>
<ul>
<li><p>Data is brought into SDI is governed e.g. you can identify data
lineage</p></li>
</ul>
<p>Cons:</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>2b</p></td>
<td class="confluenceTd"><p>Intervened phase out - Ungoverned
approach</p></td>
<td class="confluenceTd"><p>Provide adhoc tool sets to connect to these
sources and an ungoverned playground in SDI environment to analyse the
data. This option is only relevant to Pricing &amp; Capital Management
team’s requirement as they are the only team doing scenario based data
analysis.</p>
<p>Some options that needs to be POC’ed are:</p>
<ol>
<li><p>Tableau Prep</p></li>
<li><p>Azure Notebook</p></li>
<li><p>Jupiter Notebook</p></li>
<li><p>Power BI (if Power BI is available)</p></li>
</ol></td>
<td class="confluenceTd"></td>
</tr>
</tbody>
</table>

</div>

# <u>Future State</u>

Current state and non SAS based solution options for actuarial are
discussed with Pricing & Capital Management team and it was agreed that
Pricing & Capital Management team will continue to use SAS. With that
decision there is no immediate need to do any further work on the
alternate solution options (especially 2b above).

Sep 2022 is still considered as hard deadline for SAS to be phased out
for users other than Pricing & Capital Management.

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio3657477338871423332"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio3657477338871423332"
class="ap-content">

</div>

</div>

## Recommendations

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>#</strong></p></th>
<th class="confluenceTh"><p><strong>Recommendation</strong></p></th>
<th
class="confluenceTh"><p><strong>Considerations/Assumptions</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>1</p></td>
<td class="confluenceTd"><p>Pricing &amp; Capital Management team to
keep using SAS.</p></td>
<td class="confluenceTd"><ol>
<li><p>Post decommissioning of Risk Console &amp; Client Connect, Data
lake to be used as a source of truth for historical data that is not
available via Client360.</p></li>
<li></li>
</ol></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>2</p></td>
<td class="confluenceTd"><p>One of the following required for all other
reports/data extract produced by Insights &amp; Analytics team:</p>
<ol>
<li><p>Decommission the report/data extract.</p></li>
<li><p>Produce the report/data extract from SDI environment.</p></li>
</ol></td>
<td class="confluenceTd"><ol>
<li><p>Additional data sources listed in current environment section
above to be ingested in SDI environment.</p></li>
</ol></td>
</tr>
</tbody>
</table>

</div>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.drawio.tmp](attachments/704839695/704774157.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.drawio.tmp](attachments/704839695/704806923.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704544819.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704774184.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704544828.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704708650.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704315467.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704675923.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704708659.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704806955.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704741435.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~Untitled
Diagram.png.tmp](attachments/704839695/704675864.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png](attachments/704839695/704806964.png)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png.png](attachments/704839695/704544841.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png](attachments/704839695/718045476.png)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png.png](attachments/704839695/717455743.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png](attachments/704839695/720437962.png)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png.png](attachments/704839695/720437972.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/722993212.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/720437920.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/720175437.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/720437930.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/720437940.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/720372287.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/813957164.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png](attachments/704839695/704741444.png)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Untitled Diagram.png.png](attachments/704839695/704675932.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/720372317.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/720175467.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/717259700.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/720372326.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/723353630.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/723353639.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/722993257.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/723353648.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/720438015.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~system
contex.drawio.tmp](attachments/704839695/720438010.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio](attachments/704839695/723353784.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio.png](attachments/704839695/717259828.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio](attachments/704839695/720438291.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio.png](attachments/704839695/722928198.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/717259961.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/720372669.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/722895269.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/720175671.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/720438277.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/722928188.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/722796979.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/722895283.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/720175727.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio](attachments/704839695/720175737.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio.png](attachments/704839695/720372716.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~system contex.drawio.tmp](attachments/704839695/723353661.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio](attachments/704839695/722993266.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [system
contex.drawio.png](attachments/704839695/717259709.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780926985.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780795974.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780828722.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780795983.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780763153.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780959767.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780926994.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780730398.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780795992.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5d706da1ab129d0c306db2d5\~test.drawio.tmp](attachments/704839695/780763139.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[test.drawio](attachments/704839695/780894222.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[test.drawio.png](attachments/704839695/780894227.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/813629463.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/813334540.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Untitled Diagram.png.tmp](attachments/704839695/717259016.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~test.drawio.tmp](attachments/704839695/814055445.tmp)
(application/vnd.jgraph.mxfile)  

</div>
