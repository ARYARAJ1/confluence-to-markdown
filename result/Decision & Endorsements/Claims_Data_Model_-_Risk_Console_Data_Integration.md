# Claims Data Model - Risk Console Data Integration

<div>

<div>

Add your comments directly to the page. Include links to any relevant
research, data, or feedback.

</div>

</div>

<div class="plugin-tabmeta-details">

<div class="table-wrap">

|              |                                                                                                                                                   |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| Status       | DECIDED                                                                                                                                           |
| Impact       | MEDIUM                                                                                                                                            |
| Driver       | j.stones@vmia.vic.gov.au (Deactivated)                                                                                                            |
| Approver     | Chris Jackson                                                                                                                                     |
| Contributors | Bob Zhu h.lindsaysmith@vmia.vic.gov.au Lindsay-Smith                                                                                              |
| Informed     | Lasath Kahingala                                                                                                                                  |
| Due date     | 02 Aug 2021                                                                                                                                       |
| Outcome      | Risk console is integrated into the modelled layer, whilst data is still being updated and then will be turned off once there are no more changes |

</div>

</div>

## Background

During transition to C360 there is a period of time where both Risk
Console and C360 will contain the data required for the claims reporting
data model depending on the status of the roll out.

Since the eventual plan is for Risk Console to be retired entirely, the
effort to integrate Risk Console into the data pipelines (which would
require conforming Risk Console & C360 data within the stored procedures
that populate the data warehouse) would be throwaway in the first place
and would also require unravelling at a later date, causing potential
regression issues.

Instead, a hybrid approach where C360 is treated as the master and is
the only fully engineered source for the data model layer is preferred.
In order to bring in the required data into the reporting model for the
interim state, we can create a view layer on top of the star schema
which will augment with Risk Console data.

An additional benefit of taking this approach besides effort savings, is
that the star schema will never need to be altered, only added to as the
functionality is migrated to Risk Console. This means reduced complexity
and risk in regression and reconciliation testing.

As components of Risk Console are decommissioned, the corresponding
parts of the view layer can simply be removed.

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio-sketch5463211677508206481"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio-sketch5463211677508206481"
class="ap-content">

</div>

</div>

The trade off for this approach is query performance & execution cost
against the model as the complex logic to integrate Risk Console is
executed each time the data model is accessed, rather than during an
overnight batch.

These costs should be offset by savings in build, reconciliation &
regression testing effort.

## Relevant data

## Options considered

<div class="table-wrap">

<table class="confluenceTable" data-layout="default"
data-local-id="186c5220-3d00-49ac-99f4-ad4063be3a23">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p>RC Data Integrated into Modelled
Layer</p></th>
<th class="confluenceTh"><p>RC Data Augmented @ Consumption
Layer</p></th>
</tr>

<tr class="odd">
<th class="confluenceTh"><p>Description</p></th>
<td class="confluenceTd"><p>In build phase, risk console tables will be
integrated into the stored procedures to load the claims data model and
each nightly load will add risk console deltas.</p></td>
<td class="confluenceTd"><p>In build phase, only C360 data will be
engineered into the stored procedures and the physicalised claims data
model. Risk console data will be augmented on top of this model in a
temporary view layer.</p></td>
</tr>
<tr class="even">
<th class="confluenceTh"><p>Pros and cons</p></th>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Better
query time performance.</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Higher build effort both in initial build and as C360 roll out requires
further changes to the data model.</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> More
complex to transition from Risk Console to C360 in data model.</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Quicker
(and cheaper) to build.</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" />
Decouples risk console &amp; client 360 data integration rules, making
the transition easier.</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Reduces
risk of issues in regression testing.</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Performance impact as data model query will be more complex, sourcing
data from both modelled layer and risk console staging layer.</p></td>
</tr>
<tr class="odd">
<th class="confluenceTh"><p>Estimated cost</p></th>
<td class="confluenceTd"><p>MEDIUM</p></td>
<td class="confluenceTd"><p>LOW</p></td>
</tr>
</tbody>
</table>

</div>

## Action items

-   j.stones@vmia.vic.gov.au (Deactivated) to set up meeting to discuss.

## Outcome

The design has settled with Risk Console being mapped as a source into
both the staging layer and the warehouse tables. Whilst the system is
live the data will update, once the system is static the union queries
for riskconsole can be deleted.

Up to today 21 Sep 2021 there have been no data quality issues that
would block an onoing live feed of data into the warehouse from risk
console.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Design Decision - Model
Integration.tmp](attachments/1151697561/1227620367.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Design Decision - Model
Integration.tmp](attachments/1151697561/1227489298.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Design Decision - Model
Integration.tmp](attachments/1151697561/1228275715.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Design
Decision - Model Integration](attachments/1151697561/1228111881)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Design
Decision - Model Integration.png](attachments/1151697561/1228111886.png)
(image/png)  

</div>
