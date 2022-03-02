# Tableau vs Power BI

<div>

<div>

Add your comments directly to the page. Include links to any relevant
research, data, or feedback.

</div>

</div>

<div class="plugin-tabmeta-details">

<div class="table-wrap">

|              |                                              |
|--------------|----------------------------------------------|
| Status       | IN PROGRESS                                  |
| Impact       | MEDIUM                                       |
| Driver       | p.lim@vmia.vic.gov.au Lim                    |
| Approver     |                                              |
| Contributors | Jason Dang (Deactivated)                     |
| Informed     | h.lindsaysmith@vmia.vic.gov.au Lindsay-Smith |
| Due date     |                                              |
| Outcome      |                                              |

</div>

</div>

## Background

VMIA’s business intelligence framework aims to support fully automated
reporting, dashboarding and monitoring of key client metric data. The
selected platform should appropriately facilitate reporting and visual
analytics on top of the Enterprise Data Warehouse reporting access
layer. Candidate technologies Tableau and PowerBI will be evaluated
against VMIA’s requirements and needs, particularly in the areas of
analytics and visualization, data sources, supported data and
user-friendliness.

## Relevant data

### Tableau

Tableau is a data visualization and analytics tool that helps simplify
raw data into easily understandable formats. Data analysis and
visualizations can be quickly generated in the form of dashboards and
worksheets. Dashboards are easy to customize with drag-and-drop
interfaces that do not require complex scripting. Key features of
Tableau include

-   Data Blending

-   Real time analysis

-   Collaboration of data

-   Centralization of data sources

Data analytics in Tableau can be classified into two areas:

1.  Developer Tools. Tableau tools used for development such as the
    creation of dashboards, charts, report generation, visualization
    fall into this category. Products that fall into this category are
    Tableau Desktop and Tableau Public.

2.  Sharing Tools. The purpose of the tool is for sharing
    visualizations, reports and dashboards that were created using the
    developer tools. Products that fall into this category are Tableau
    Online, Server, and Reader.

Tableau connects and extracts the data stored in various places. It can
pull data from myriad platforms, from an excel workbook to an Azure SQL
database. When a workbook or data source is published, a version is
saved in the revision history for Tableau Server and Tableau Online. You
can revert to a previous version at any time.

### Power BI

Power BI is a Business Intelligence and Data Visualization tool which
helps convert data from various data source into interactive dashboards
and BI reports. The Power BI suite provides multiple software,
connectors and services - Power BI desktop, Power BI service based on
SaaS, and mobile Power BI apps available for different platforms. Key
features of PowerBI include:

-   Seamless integration with the Microsoft Office ecosystem

-   API access and pre-built dashboards

-   Real-time data access

-   PowerQuery functionality

Power BI comes in three forms: desktop, mobile, and service. The most
basic set up is an Azure tenant that is connected to Power BI through an
Office365 Admin interface. Power BI is fairly easy to use, and you can
quickly connect existing spreadsheets, data sources, and apps via
built-in connections and APIs.

## Tableau vs Power BI feature comparison

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="odd">
<td class="confluenceTd"><p><strong> Feature</strong></p></td>
<td class="confluenceTd"><p><strong>Tableau</strong></p></td>
<td class="confluenceTd"><p><strong>Power BI</strong></p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Ease-of-use</p></td>
<td class="confluenceTd"><p>Tableau has an easier look and feel for a
first time user. Tableau is able to build basic charts and graphs fast,
and its “Show Me“ menu offers automatic visuals for whatever data you
have and is a great learning tool.</p>
<p>However, as complexity grows the required knowledge and skill set
increases along with it, with more advanced visualizations taking much
more effort to learn.</p></td>
<td class="confluenceTd"><p>Power BI leverages the PowerQuery tool which
uses the expression languages M and DAX. While these languages have a
steeper learning curve, they allow users to build more complex
graphs.</p>
<p>Power BI has its roots in the Microsoft Office 365 suite,
particularly MS Excel, making it generally easier for user
adoption.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Integration of custom 3<sup>rd</sup> party
charts</p></td>
<td class="confluenceTd"><p>Tableau Dashboard Extensions are web
applications that have two-way communication with the dashboard,
allowing you to integrate third-party APIs such as d3.js charting
libraries.</p></td>
<td class="confluenceTd"><p>There are a number of approaches to create
custom visuals in Power BI. The User Market Place allows users to
download charts provided by partners. Custom visuals can also be created
with JavaScript, .NET as well as R and Python scripts in the Power BI
Desktop.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>User Interface</p></td>
<td class="confluenceTd"><p>Tableau charts and visualizations are
cleaner. Drag-and-drop features are better in Tableau allow business
users to create a customized dashboard easily. Advanced analysts can
also leverage data exploration and transformation functionalities to
build advanced visualizations.</p></td>
<td class="confluenceTd"><p>PowerBI has a more intuitive interface that
is simpler to learn than Tableau. Business users will also be able to
leverage drag-and-drop reporting and visualization capabilities without
needing to be too technical.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Data connections</p></td>
<td class="confluenceTd"><p>Tableau can be deployed on-premises and
public cloud via Tableau Server on Azure, AWS and Google Cloud, or
fully-managed SaaS via Tableau Online hosting. </p>
<p>Tableau supports over 65+ native connections for real-time insights,
including Azure SQL Data Warehouse, Dropbox, Google Cloud SQL, MSQL,
OneDrive, and Oracle. </p></td>
<td class="confluenceTd"><p>Power BI can be deployed in Microsoft Azure
public cloud via Power BI Desktop, Pro and Premium (SaaS), or
on-premises via Power BI Report Server.</p>
<p>Because Power BI is a Microsoft SaaS product hosted in the cloud,
it’s seamlessly integrated Office 365 applications and other Microsoft
Azure services currently supports up to 70+ connectors
out-of-the-box. </p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Syntax</p></td>
<td class="confluenceTd"><p>Tableau has its own syntax for functions,
fields, operators and literals. It also uses scripting languages such as
R and Python for complex table calculations.</p></td>
<td class="confluenceTd"><p>PowerQuery uses the expression language M
for data preparation and DAX for data analysis.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Internal data model</p></td>
<td class="confluenceTd"><p>Tableau’s data model has a logical layer
(relationship definition), and the physical layer (data combination
using unions and joins).</p>
<p>Much of building the data model utilities drag-and-drop
functionality, and may require Tableau Prep for proper data blending for
data preparation.</p></td>
<td class="confluenceTd"><p>With the modeling feature, you can build
custom calculations on the existing tables and these columns can be
directly presented into Power BI visualizations.</p>
<p>PowerQuery’s M can be used as 'transformation code' when defining
data models and their relationships.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Performance</p></td>
<td class="confluenceTd"><p>Tableau can handle a huge volume of data
with better performance. Tableau does not set the limit for data volume
- with its column-based structure, it can easily fit billions of data
rows.</p></td>
<td class="confluenceTd"><p>Can handle a limited volume of data. Power
BI provides each workspace with 10 GB of data storage, and a Premium
account storage limit of 100 TB.</p></td>
</tr>
</tbody>
</table>

</div>

## Tableau Pricing

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Tableau Creator</strong></p></th>
<th class="confluenceTh"><p><strong>Tableau Explorer</strong></p></th>
<th class="confluenceTh"><p><strong>Tableau Viewer</strong></p></th>
<th class="confluenceTh"><p><strong>Add-ons</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"
data-highlight-colour="#f4f5f7"><p><strong>Features</strong></p></td>
<td class="confluenceTd"><p>Intended for an individual user. It includes
licenses to:</p>
<ul>
<li><p>Tableau Desktop</p></li>
<li><p>Tableau Prep Builder</p></li>
<li><p>One Creator license of Tableau Server.</p></li>
</ul></td>
<td class="confluenceTd"><p>This plan is designed for users that want
self-service analytics without the data prepping and cleaning. It
includes a license to:</p>
<ul>
<li><p>One Explorer license of Tableau Server.</p></li>
</ul></td>
<td class="confluenceTd"><p>This plan is intended for users that want to
access already-created visualizations. It includes a license to: </p>
<ul>
<li><p>One Viewer license of Tableau Server.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Tableau Data Management</p></li>
<li><p>Tableau Server Management</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"
data-highlight-colour="#f4f5f7"><p><strong>Price</strong></p></td>
<td class="confluenceTd"><ul>
<li><p>$70 user/month, billed annually</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>$35 user/month, billed annually</p></li>
<li><p>Min. 5 Explorers required</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>$12 user/month | billed annually</p></li>
<li><p>Min. 100 Viewers required</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Tableau Data Management: $5.50 user/month, billed annually<br />
required for all Creators, Explorers, and Viewers on an account</p></li>
<li><p>Tableau Server Management: $3.00 user/month | billed
annually<br />
required for all Creators, Explorers, and Viewers on an account</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

## Power BI Pricing

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p><strong>Power BI Desktop</strong></p></th>
<th class="confluenceTh"><p><strong>Power BI Pro</strong></p></th>
<th class="confluenceTh"><p><strong>Power BI Premium</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"
data-highlight-colour="#f4f5f7"><p><strong>Features</strong></p></td>
<td class="confluenceTd"><p>Includes data cleaning and preparation,
custom visualizations and the ability to publish to the Power BI
service. Power BI Free enables you to connect to 70+ data sources,
publish to the web, and export to excel. There are some limitations to
the free version (e.g. no peer-to-peer sharing or App workspaces).</p>
<p>The free version is good for when there is no need to distribute the
analysis to other end users. There are also connectivity options such as
DirectQuery, live connection, and the use of the gateway. The same
visualizations that are available in Power BI Pro are available in Power
BI Free.</p></td>
<td class="confluenceTd"><p>Includes data collaboration, data
governance, building dashboards with a 360-degree real-time view and the
ability to publish reports anywhere. Other features include:</p>
<ul>
<li><p>Self-service and modern BI in the cloud </p></li>
<li><p>Collaboration, publishing, sharing, and ad-hoc analysis </p></li>
<li><p>Fully managed by Microsoft</p></li>
</ul>
<p>For small and large deployments, Power BI Pro works great to deliver
full Power BI capabilities to all users. Employees across roles,
departments, etc. can engage in ad-hoc analysis, dashboard sharing and
report publishing, collaboration and other related activities.</p></td>
<td class="confluenceTd"><p>With Power BI Premium, VMIA are licensing
capacity for their content rather than licensing all users of that
content. Content (datasets, dashboards, and reports) is stored in
Premium and can then be viewed by as many users as you want, without
additional per-user costs.</p>
<p>These users can only view content, not create it. Viewing includes
looking at dashboards and reports on the web, in our mobile apps, or
embedded in your organization’s portals or apps. The creators of content
in Premium still need their own Pro licenses.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"
data-highlight-colour="#f4f5f7"><p><strong>Price</strong></p></td>
<td class="confluenceTd"><p>This offering is free to any single
user.</p></td>
<td class="confluenceTd"><p>VMIA will have E5 licensing, and Power BI
pro will be offered under this plan.</p></td>
<td class="confluenceTd"><p>The Premium plan starts at $4,995 a month
per dedicated cloud compute and storage resource.</p></td>
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
<th class="confluenceTh"><p>Option 1:Tableau</p></th>
<th class="confluenceTh"><p>Option 2: Power BI</p></th>
</tr>

<tr class="odd">
<th class="confluenceTh"><p>Description</p></th>
<td class="confluenceTd"><p>Tableau offers interactive, visual-based
data exploration capabilities and more robust visualization options that
advanced analysts can leverage to the fullest.</p></td>
<td class="confluenceTd"><p>Power BI offers augmented analytics, data
preparation, interactive dashboards, visual-based data discovery in one
self-service business intelligence platform.</p></td>
</tr>
<tr class="even">
<th class="confluenceTh"><p>Pros and cons</p></th>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Cleaner
user interface, better user experience</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Users
are free to use any number of data points; good for exploratory data
analysis</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> High
performance with large datasets</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Tableau
offers better support for connecting to a distinct data warehouse</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Allows
users to save revision history</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> Lacks
functionality required for full-fledged, large scale reporting [CJ this
point contradicts the second and third points you have raised above]</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Relatively high cost, compared to other BI Tools</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> It’s
user interface is harder to use in comparison to PowerBI [CJ Can we
clarify if this is for the producer/publisher or the end user please -
as this point is a direct contradiction of the top point]</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Data
exploration using natural language query.</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" />
Pre-built plug-and-play dashboards</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Ability
to reuse queries for different file imports</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" />
Visualisations are more heavily focused on predictive modelling and
reporting</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> User
interface is more intuitive than Tableau [CJ this point contradicts the
top point you have made about Tableau]</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> No
built-in version control</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Limited data storage; unable to handle large data volumes [CJ are our
data needs considered ‘large’?]</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> Can
only define simple data model relationships, not complex ones [CJ Are
our data model relationships ‘complex’?]</p></td>
</tr>
<tr class="odd">
<th class="confluenceTh"><p>Estimated cost</p></th>
<td class="confluenceTd"><p>LARGE</p></td>
<td class="confluenceTd"><p>LOW</p></td>
</tr>
</tbody>
</table>

</div>

## Action items

-   

**Additional Questions**

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="odd">
<td class="confluenceTd"><p>Consideration</p></td>
<td class="confluenceTd"><p>Description/Input criteria</p></td>
<td class="confluenceTd"><p>Tableau</p></td>
<td class="confluenceTd"><p>Power BI</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Cost</p></td>
<td class="confluenceTd"><p>Number of users is expected to be 200.</p>
<p>·        This includes VMIA users as well as client users</p>
<p>·        Not all clients will be users of the self- service/online
version</p></td>
<td class="confluenceTd"><p><u><strong>Questions</strong></u></p>
<ol>
<li><p>What is Tableau version/product is required to allow
self-service/interactive reporting/on?</p></li>
<li><p>What is the license cost?</p></li>
</ol>
<p>Tableau Server - <a
href="https://clicktime.symantec.com/36xfzANz2yyhPe4iGwne53r7Vc?u=https%3A%2F%2Fwww.tableau.com%2Fproducts%2Fserver"
rel="nofollow">https://www.tableau.com/products/server</a> (Creator /
Explorer / Viewer roles)</p>
<ol>
<li><p>Explorer: licence $35 user/month - <a
href="https://clicktime.symantec.com/38qrHkx6pS4U3KNDqqMayLF7Vc?u=https%3A%2F%2Fwww.tableau.com%2Fpricing%2Fteams-orgs"
rel="nofollow">https://www.tableau.com/pricing/teams-orgs</a></p>
<ul>
<li><p>Viewer: licence $12 user/month - <a
href="https://clicktime.symantec.com/38qrHkx6pS4U3KNDqqMayLF7Vc?u=https%3A%2F%2Fwww.tableau.com%2Fpricing%2Fteams-orgs"
rel="nofollow">https://www.tableau.com/pricing/teams-orgs</a></p></li>
</ul></li>
</ol>
<p>Explorer is required for creating new dashboards. Viewer sufficient
for just viewing and filtering on existing dashboards</p>
<p>[CJ question] Has this considered the use of third party hosting to
provide access to clients, or is this only an internal costing
issue?</p></td>
<td class="confluenceTd"><p>Power BI pro will be included (free of
charge) in the VMIA will E5 Microsoft licensing</p>
<p> </p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Geo-Spatial analysis</p>
<p> </p></td>
<td class="confluenceTd"><p><strong>Visualise</strong>,
<strong>analyse</strong> and <strong>share</strong> spatial information.
One real use case from the current environment is “VMIA Client Asset
Map” which display/visualise all insured properties from the “Client
Asset register” There are other real examples that can be seen on
Internet at below link</p>
<p> </p>
<p><a
href="https://intranet.vmia.vic.gov.au/Useful-Things/VMIA-Maps/VMIA-Maps"
rel="nofollow">https://intranet.vmia.vic.gov.au/Useful-Things/VMIA-Maps/VMIA-Maps</a></p>
<p> </p></td>
<td class="confluenceTd"><p>Has capability and is being currently used
to visualise, analyse and share spatial information</p></td>
<td class="confluenceTd"><p><u><strong>Questions</strong></u></p>
<p>What capabilities does Power BI have to satisfy VMIA use case?</p>
<ul>
<li><p>PowerBI has ability to visualise spatial information through
multiple options:</p>
<ul>
<li><p>Map - <a
href="https://clicktime.symantec.com/3V9PfNtJFDCLhqJaBqc2p7f7Vc?u=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fpower-bi%2Fvisuals%2Fpower-bi-map-tips-and-tricks"
rel="nofollow">https://docs.microsoft.com/en-us/power-bi/visuals/power-bi-map-tips-and-tricks</a></p></li>
<li><p>Filled (choropleth) map - <a
href="https://clicktime.symantec.com/3UUAnQPUADPCDr5rU2BsTD67Vc?u=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fpower-bi%2Fvisuals%2Fpower-bi-visualization-filled-maps-choropleths"
rel="nofollow">https://docs.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-filled-maps-choropleths</a></p></li>
<li><p>3<sup>rd</sup> party (e.g. ArcGIS Maps for Power BI) - <a
href="https://clicktime.symantec.com/3D6Z7BV6vXRxT8mL4zoBmBX7Vc?u=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fpower-bi%2Fvisuals%2Fpower-bi-visualizations-arcgis"
rel="nofollow">https://docs.microsoft.com/en-us/power-bi/visuals/power-bi-visualizations-arcgis</a></p></li>
</ul></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Power BI constraints</p></td>
<td class="confluenceTd"><p>·        No built-in version control</p>
<p>·        Limited data storage; unable to handle large data
volumes</p>
<p>·        Can only define simple data model relationships, not complex
ones</p>
<p> </p></td>
<td class="confluenceTd"><p>N/A</p></td>
<td class="confluenceTd"><p><u><strong>Questions</strong></u></p>
<p>How will these constraints impact VMIA based on the architecture
(Azure/Snowflake) and the requirements/use case(e.g. size of the data
and processing speeds required).</p>
<p>version control → Can use PowerBI enterprise deployment best
practice. pg 164</p>
<p><a
href="https://docs.microsoft.com/en-us/power-bi/guidance/whitepaper-powerbi-enterprise-deployment"
rel="nofollow">https://docs.microsoft.com/en-us/power-bi/guidance/whitepaper-powerbi-enterprise-deployment</a></p>
<p>Data storage → can go beyond default capacity (10GB?) if we use live
querying this would not be a limitation. if we want more dataset
capacity it can be purchased</p>
<p>Suggest a Proof of Concept to fully confirm dataset sizing. the
expectation is VMIA’s volumes are not large and a solution that works
can be found.</p>
<p>Data model → not necessary is all sources are snowflake.</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>[CJ issue] External
access/publishing</p></td>
<td class="confluenceTd"><ul>
<li><p>Understanding the ability to distribute the output
(visualisations) to clients</p></li>
</ul></td>
<td class="confluenceTd"><p>Currently distribute to clients via third
party hosting / Infonyx</p>
<p>This model was expected to change once we had a platform providing
single point login/authentication (like C360).</p>
<p>Has the thinking around this changed or is this assumption still
valid?</p></td>
<td class="confluenceTd"><p><u><strong>Questions</strong></u></p>
<p>How do we provide access to third parties in a protected
(authenticated and authorised) environment?</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>[CJ issue] Duplication of
effort/rework</p></td>
<td class="confluenceTd"><ul>
<li><p>Consideration of incumbancy vs change issue</p></li>
</ul></td>
<td class="confluenceTd"><p>Not applicable</p></td>
<td class="confluenceTd"><p><u><strong>Questions</strong></u></p>
<p>Have we considered the cost to transfer skills to new platform / and
effort to recreate existing reporting in new platform?</p></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>[CJ issue] User testing</p></td>
<td class="confluenceTd"><ul>
<li><p>Applying lessons learnt from other projects - more of a comment
but it appears that we are basing this on specs with a ‘compare x with
y’ type information seeking, rather than actually applying this to VMIA
problems and building solutions. We already know how Tableau performs
for solving a problem, why don’t we run the same problem through Power
BI and compare apples with apples. A lot of the assumptions we made
based on previous projects were not supported once we did an actual
proof of concept.</p></li>
</ul></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>[CJ issue] Transferability across Victorian
Government</p></td>
<td class="confluenceTd"><ul>
<li><p>Several System Owner clients have switched to Tableau - have we
considered the wider ecosystem to encourage transferability and skill
sharing across public sector and our key clients.</p></li>
</ul></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
</tr>
</tbody>
</table>

</div>

## Recommendation

From a reporting and usability perspective, Power BI’s toolset allows
for better reporting capabilities that are more suited to VMIA’s needs.
Power BI’s ability to create different charts, views, and dashboards
offers varying levels of granularity for a number of different
audiences. While Tableau offers a clean interface and lets users create
different types of baseline visualizations, it lacks the functionality
to build large amounts of information tables and static layouts that are
required in large scale reporting. \[CJ comment - this is inconsistent
with the cons included about PowerBI above?\] Tableau has several merits
in the area of data exploration and analysis, but its overall ease of
use and integration with the Microsoft ecosystem is incomparable to
Power BI. \[CJ question- We are not talking about integrating with
Microsoft though, are we - isn’t the visualisation tool going to be
talking to Snowflake?\]

Power BI’s PowerQuery tool allows users to reuse queries and apply them
to multiple files, supporting the repeatability and re-usability of
Applied Steps to different file imports. This takes the complexity out
of periodic reporting, promoting simplicity and ease of use for business
users who need not learn M or DAX from scratch in order to perform
tasks. Additionally, VMIA will soon have Office 365 E5 licenses which
includes PowerBI Pro, thus reducing the cost of purchasing licenses. It
is therefore recommended that VMIA adopt Power BI as their primary BI
tool, given its advantages are better poised to support VMIA’s
requirements.

## Outcome
