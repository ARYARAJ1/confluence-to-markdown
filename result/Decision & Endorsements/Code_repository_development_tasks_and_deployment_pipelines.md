# Code repository, development tasks and deployment pipelines

<div>

<div>

Add your comments directly to the page. Include links to any relevant
research, data, or feedback.

</div>

</div>

<div class="plugin-tabmeta-details">

<div class="table-wrap">

|              |                                                                                                                                |
|--------------|--------------------------------------------------------------------------------------------------------------------------------|
| Status       | OUTCOME DECIDED                                                                                                                |
| Impact       | HIGH                                                                                                                           |
| Driver       | Jason Dang (Deactivated) p.lim@vmia.vic.gov.au Lim                                                                             |
| Approver     | Hendry Susilo                                                                                                                  |
| Contributors |                                                                                                                                |
| Informed     | Lasath Kahingala                                                                                                               |
| Due date     | 19/06/2020                                                                                                                     |
| Outcome      | Approval has been provided by Hendry to proceed with the recommendation stated above which is to use the full Atlassian Suite. |

</div>

</div>

## Background

VMIA are currently using Jira as their main tool for project management
with a small usage of Bitbucket for source code versioning and
Confluence for documentation included in the Atlassian suite. As the
strategic data platform will be on Azure cloud, there may be an option
to utilise some or all of the Azure DevOps services as an alternative to
some of the existing Atlassian suite usage at VMIA.

## Relevant data

The full Atlassian suite is a strong preference of VMIA’s. Bitbucket is
the preferred source code repository, owing to its seamless integration
with Jira and Confluence. This makes it easy for rapid collaboration,
testing and deploying of code.

Azure DevOps is a considered alternative due to its ability to integrate
with all Azure cloud services, namely the core service domains:

-   Compute

-   Storage

-   Networking

-   Databases

-   Monitoring

Initial barriers to entry of Azure DevOps is low, and its integration
with Microsoft products enable the leveraging of a broader ecosystem.
It’s features and functionalities may initially come across as more
approachable than Bitbucket, but its simplicity can also be a
double-edged sword as it lacks the customization capabilities offer by
Jira, such as built-in road maps and agile reporting tools.

The following table highlights key features of both tools:

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p>Atlassian Suite</p></th>
<th class="confluenceTh"><p><strong>Azure DevOps</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p><u>Jira</u></p>
<ul>
<li><p>The ability to plan agile work from project backlog
to sprints</p></li>
<li><p>Fully customizable Kanban and Scrum boards</p></li>
<li><p>The ability to estimate time for issues for backlogs</p></li>
<li><p>Robust reporting features, ranging from burn down charts to
velocity measurements</p></li>
<li><p>Customizable workflows to fit frameworks</p></li>
</ul>
<p><u>Confluence</u></p>
<ul>
<li><p>Confluence is a collaboration wiki tool for teams to create
meeting notes, project plans, product requirements, at the same time as
other users are editing and see all the changes at once.</p></li>
<li><p>Accelerated feedback loop with inline comments on pages and files
attached.</p></li>
<li><p>Ready-made solutions for daily documentation needs with
Templates.</p></li>
</ul>
<p><u>Bitbucket</u></p>
<ul>
<li><p>Git repository management to manage git repositories, collaborate
on source code and guide you through the development flow.</p></li>
<li><p>Access control to restrict access to your source code.</p></li>
<li><p>Workflow control to enforce a project or team workflow; pull
requests with in-line commenting for collaboration on code
review.</p></li>
<li><p>Bitbucket Pipelines: an integrated CI/CD service, which allows
you to automatically build, test and deploy code.</p></li>
<li><p>Pipelines has integrations with tools like Jira and Microsoft
Teams that provides context on your builds and deployments and full
development traceability.</p></li>
<li><p>Pipelines enables easy scaling of testing: they grow with
requirements, not leaving you restricted based on the hardware you have
available.</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Azure Boards: agile planning, work item tracking, visualization
and reporting tool.</p></li>
<li><p>Azure Pipelines: a language, platform and cloud agnostic CI/CD
platform with support for containers.</p></li>
<li><p>Azure Repos: provides cloud-hosted private git repos.</p></li>
<li><p>Azure Artifacts: provides integrated package management with
support for Maven, npm, Python and NuGet package feeds from public or
private sources.</p></li>
<li><p>Azure Test Plans: provides an integrated planned and exploratory
testing solution.</p></li>
<li><p>CI/CD tooling: Extensible (via a Marketplace) - you can provision
and manage Azure infrastructure using third-party tools such as Ansible,
Chef, Puppet, Terraform and Pulumi.</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

## Options considered

<div class="table-wrap">

<table class="confluenceTable" data-layout="wide">
<tbody>
<tr class="header">
<th class="confluenceTh"></th>
<th class="confluenceTh"><p>Option 1: Full Atlassian Suite</p></th>
<th class="confluenceTh"><p>Option 2: Hybrid Atlassian and Azure
DevOps</p></th>
<th class="confluenceTh"><p>Option 3: Full Azure DevOps</p></th>
</tr>

<tr class="odd">
<th class="confluenceTh"><p>Description</p></th>
<td class="confluenceTd"><p>The Jira platform is a workflow engine that
allows for tracking of issues or tasks through a predefined and
customization workflow. Integration with Confluence and Bitbucket can
enable greater collaboration and optimise feedback loops.</p></td>
<td class="confluenceTd"><p>This option will employ Jira and Confluence
for task and sprint management, and Azure DevOps for CI/CD pipeline
management and source code version control.</p></td>
<td class="confluenceTd"><p>Azure DevOps will be the central platform
for sprint planning, task management and source code version
control.</p></td>
</tr>
<tr class="even">
<th class="confluenceTh"><p>Pros and cons</p></th>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Task,
sprint and testing management: Bitbucket can be configured with web
hooks to use Jira IDs in commits which links and then trigger
builds.</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" />
Customisable dashboard, sprint planning and automatic notifications</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> No
native artifacts/binary repository management solution. Will need to
adopt third-party solutions such as JFrog Artifactory or Nexus</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> High
interface learning curve across whole suite compared to Azure DevOps</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Bitbucket repository size limitations (2GB)</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Currently unable to host UI Data factory projects which generate JSON
automatically &amp; Azure Automation</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" />
Integrated services with the Azure cloud platform and Microsoft
development tools</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Robust
documentation and technical support</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> No
capability to link Jira work items to Azure DevOps commits</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
DevOps boards does not have the same level of customisation as Jira</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> Will
require an additional plugin to integrate Azure pipelines with Jira; the
plugin itself has known issues</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> The
additional plugin above only integrates Azure Pipelines with Jira. There
is a dependency to invest in Github as the Source Code Repository
platform.</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Reported/known bugs with test management module</p></td>
<td class="confluenceTd"><p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Ability
to leverage Microsoft’s extended ecosystem</p>
<p><img src="images/icons/emoticons/add.png"
class="emoticon emoticon-plus" data-emoji-id="atlassian-plus"
data-emoji-shortname=":plus:" data-emoji-fallback=":plus:"
data-emoticon-name="plus" width="16" height="16" alt="(plus)" /> Robust
documentation and technical support</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Integration with non-Microsoft is difficult</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" /> Lack
of visibility around cross-team dependencies would need workaround to
populate required VMIA information in Jira</p>
<p><img src="images/icons/emoticons/forbidden.png"
class="emoticon emoticon-minus" data-emoji-id="atlassian-minus"
data-emoji-shortname=":minus:" data-emoji-fallback=":minus:"
data-emoticon-name="minus" width="16" height="16" alt="(minus)" />
Reported/known bugs with test management module</p></td>
</tr>
<tr class="odd">
<th class="confluenceTh"><p>Estimated cost</p>
<p>Assumptions</p>
<p>3 users</p>
<p>5 parallel jobs based upon 5 repos</p>
<p>1 hour per job per day per user.</p>
<p>Total Estimated Build Minutes/Month:<br />
27900 Minutes</p></th>
<td class="confluenceTd"><p><strong>Standard plan
Description:</strong></p>
<p>$15/mo flat rate for 1-5 users:<br />
Cost A: $15/month</p>
<p>Build Minutes: 2,500 min/mo, $10 for every 1,000 additional build
minutes:<br />
Cost B: $260 for additional 26,000 build minutes per month</p>
<p>File storage for LFS: 5 GB, $10/month for every additional 100
GB.<br />
Cost C: $10 for assumed 105GB file storage</p>
<p>Estimated Cost/Month<br />
(A + B + C): $285/Month</p></td>
<td class="confluenceTd"><p><strong>GitHub Team Plan:</strong></p>
<p>$4 per user/month:<br />
Cost A: $12/month (for 3 users)</p>
<p><strong>Azure Pipelines Individual Service</strong><br />
5 MS Hosted Parallel Jobs with Unlimited Minutes ($54.92 per parallel
job with unlimited minutes)<br />
Cost B: $274.60/month<br />
</p>
<p>Estimated Cost/Month<br />
(A + B ): $287/Month</p></td>
<td class="confluenceTd"><p><strong>Azure Basic Plan (no Test
Plans):</strong><br />
Included Services: Azure pipelines, Boards, Repos and Artifacts</p>
<p>First 5 users free, then $6/mo. per user<br />
Cost A: $0/month (for 3 users)</p>
<p>5 MS Hosted Parallel Jobs with Unlimited Minutes ($54.92 per parallel
job with unlimited minutes)<br />
Cost B: $274.60/month</p>
<p>Estimated Cost/Month<br />
(A + B ): $274.60/Month</p></td>
</tr>
</tbody>
</table>

</div>

## Pricing

-   Azure DevOps

Azure DevOps comes with 5 ‘Basic’ users and unlimited number of
‘Stakeholder’ users for free.

‘Basic’ level is required for any kind of team or system administration,
but a standard user should be fine with a free licence.

The Test Manager licence is pricier but importantly only needs to be
bought by users that will be scripting or managing test plans – not by
users executing tests.

<a
href="https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/"
rel="nofollow">https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/</a>

-   Bitbucket

Bitbucket Cloud allows everyone with a free account an unlimited number
of public and private repositories. You can grant as many users as you
want to have access to your public repositories.

Bitbucket defines cost based on the number of users who have access to
private repositories. Each plan comes with a set amount of build minutes
for Pipelines and file storage for Git LFS, but you can get additional
minutes and storage.

<a
href="https://confluence.atlassian.com/bitbucket/plans-and-billing-224395568.html"
rel="nofollow">https://confluence.atlassian.com/bitbucket/plans-and-billing-224395568.html</a>

## Recommendation

-   The recommended approach is to engage the full Atlassian suite for
    deployment and management of the Strategic Data and Insights
    platform.

-   Bitbucket is not currently able to integrate with the Data Factory
    UI, but nevertheless the workaround of running Python scripts as
    part of Data Factory pipelines is closely aligned to the project
    team’s ways of working.

-   Azure DevOps will be unable to provide the same level of visibility
    over cross-team dependencies compared to a Bitbucket integration.

-   The cost of adopting Azure DevOps is similar to that of the full
    Atlassian suite, making both options almost equivalent in that
    regard. However, integrating Bitbucket to the current tool set will
    enable VMIA and the project team to leverage the existing project
    management capabilities afforded by Jira and Confluence. This will
    encourage better workflows and optimize feedback loops for agile
    sprint management.

## Outcome

Approval has been provided by Hendry to proceed with the recommendation
stated above which is to use the full Atlassian Suite.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [azure
pricing.PNG](attachments/8978475/13435027.png) (image/png)  

</div>
