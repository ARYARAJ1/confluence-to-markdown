# Schema evolution

Schema evolution is the ability to manage changes to a table’s data
structure whenever changes to data occurs over time. Most commonly, it’s
used when performing an append or overwrite operation, to automatically
adapt the schema to include one or more new columns. This section
provide a high level overview of the different options and approaches
that are being considered when designing for and implementing schema
evolution for VMIA.

## Background

### State Based Approach

The idea behind the State Based Approach is to maintain the source code
that represents how the desired state of the database structure. For
e.g. if a new column is to be added to an existing table, the Create DDL
table script should be updated to reflect the new column. A compare tool
will then be used to analyse the difference between what the desired
structure is and what is currently in the database, and from this
automatically generate the delta scripts required to upgrade the
existing database to mimic the desired state.

<img src="attachments/93421584/301663407.png" class="image-center"
loading="lazy" data-image-src="attachments/93421584/301663407.png"
data-height="294" data-width="664" data-unresolved-comment-count="0"
data-linked-resource-id="301663407" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="State based approach.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="93421584"
data-linked-resource-container-version="19"
data-media-id="5966fd2e-b5ea-4bb5-accd-0cc53c8a2574"
data-media-type="file" />

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Advantages</strong></p></th>
<th class="confluenceTh"><p><strong>Disadvantages</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><ul>
<li><p>All the development work of creating alter ddl scripts is done by
the compare tool, therefore freeing the developers of the additional
work required to write the delta scripts</p></li>
<li><p>Can ensure that the delta scripts are generated consistently
based on defined best practices</p></li>
<li><p>Gives developers access to the current state of the databases and
allows for developers to quickly generate the code to achieve the
desired target state</p></li>
<li><p>Easier to audit and view history of changes to the database
structure in source control</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Some database changes cannot be programmatically be computed
correctly. For e.g. how to differentiate renaming of columns vs
combination of adding and removing of a column</p></li>
<li><p>Additional requirement to design and develop the compare tool to
generate the delta scripts</p></li>
<li><p>Requires the compare tool to be thoroughly tested and
reviewed</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

### Migration based approach

In this approach, the first state would be to create an initial
database. After that the scripts that are generated should only reflect
how the database should change from its current state. Therefore this
requires the developer to understand what change is required to migrate
from the current to target state and manually generate this script
themselves.

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Advantages</strong></p></th>
<th class="confluenceTh"><p><strong>Disadvantages</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><ul>
<li><p>Full control of the delta scripts</p></li>
<li><p>Allows for early review process as the delta script is generated
prior to build execution</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Effectiveness of this approach is dependent on the script
creator's knowledge on the context of the change</p></li>
<li><p>Onboarding process of new developers will require more time of
understanding how the database upgrade works and the hisotry of changes
that have occurred.</p></li>
<li><p>over time to restore the schema to the required state will
required the application of more and more alter statements and the
current state is not stored</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

<img src="attachments/93421584/304283682.png" class="image-center"
loading="lazy" data-image-src="attachments/93421584/304283682.png"
data-height="290" data-width="633" data-unresolved-comment-count="0"
data-linked-resource-id="304283682" data-linked-resource-version="1"
data-linked-resource-type="attachment"
data-linked-resource-default-alias="Migration Based Approach.png"
data-base-url="https://vmia.atlassian.net/wiki"
data-linked-resource-content-type="image/png"
data-linked-resource-container-id="93421584"
data-linked-resource-container-version="19"
data-media-id="f26324b1-0dce-4aea-840b-285e6a2c2665"
data-media-type="file" />

## Problem Space

The following types of schema changes are being considered for schema
evolution:

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Scenarios</strong></p></th>
<th class="confluenceTh"><p><strong>Description</strong></p></th>
<th class="confluenceTh"><p><strong>Expected outcome</strong></p></th>
<th class="confluenceTh"><p><strong>Considerations</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"><p>New column added</p></td>
<td class="confluenceTd"><p>Likely to be the most common and simplest
solution, a new field has been detected in the source system that then
needs to be added to the respective table in Snowflake.</p></td>
<td class="confluenceTd"><ul>
<li><p>Either a new table is created with the additional column or the
existing table is modified with an <strong>ALTER</strong> statement to
add the new column to the table.</p></li>
<li><p>Existing rows in the table will have null values in this newly
added column</p></li>
<li><p>The hash values for each row will need to be
recalculated.</p></li>
</ul></td>
<td class="confluenceTd"></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Dropping a column that is nullable</p></td>
<td class="confluenceTd"><p>When a field is no longer available in the
source system, data for this field will therefore not be included in
future loads of the table.</p></td>
<td class="confluenceTd"><ul>
<li><p>The existing table should not be modified when this scenario
occurs. The dropped field will still exist as a column in the table
however future data loads in this table will mean this column will
default to a null value</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Is there a way to load data into a table without having to
specify all the columns in the SQL Statement or will this cause the ETL
to break.</p>
<ul>
<li><p>For e.g. If a table has 10 columns, how can data for just 9
columns be loaded</p></li>
</ul></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Dropping a column that is not
nullable</p></td>
<td class="confluenceTd"><p>When a field is no longer available in the
source system, data for this field will therefore not be included in
future loads of the table. However as this field in the table has been
initially defined as not nullable, it will not allow null values until
its properties are changed</p></td>
<td class="confluenceTd"><ul>
<li><p>The column that no longer exists in the source system should be
altered to allow for the population of null values</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Is there a way to load data into a table without having to
specify all the columns in the SQL Statement or will this cause the ETL
to break.</p>
<ul>
<li><p>For e.g. If a table has 10 columns, how can data for just 9
columns be loaded</p></li>
</ul></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Changing data type of a column</p></td>
<td class="confluenceTd"><p>If the data type of a field changes in a
source system, for e.g. from STRING to INT, the change in the respective
column’s datatype should also be reflected.</p></td>
<td class="confluenceTd"><ul>
<li><p>Either a new table is created with the additional column or the
existing table is modified with an <strong>ALTER</strong> statement to
add the new column to the table.</p></li>
<li><p>Need to confirm if hash value will change if data type has
changed</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>How will errors in casting history of data from one data type to
another be handled?</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Renaming column names</p></td>
<td class="confluenceTd"><p>If a field is renamed in the source system,
for e.g. from Birth Date to DOB.</p></td>
<td class="confluenceTd"><ul>
<li><p>There is no programmatic way of differentiating this scenario
with a combination of a new column added and an existing column
removed</p></li>
<li><p>Either a new table is created with a new column added with the
new field name or the existing table is modified with an
<strong>ALTER</strong> statement to add the new column with the updated
field name to the table.</p></li>
<li><p>Existing rows in the table will have null values in this newly
added column</p></li>
<li><p>The hash values for each row will need to be
recalculated.</p></li>
<li><p>The dropped field will still exist as a column in the table
however future data loads in this table will mean this column will
default to a null value</p></li>
<li><p>History of renamed columns will not be retained</p></li>
<li><p>If the load type into the table is a full snapshot, then all the
records in the table will be reinserted as the hash values would have
changed</p></li>
<li><p>If the load type into the table is a delta load, need to confirm
if a change if field name will also be included in the delta
load</p></li>
</ul></td>
<td class="confluenceTd"><ul>
<li><p>Will delta loads include field name changes in the delta
snapshot?</p>
<ul>
<li><p>If not, then existing active records may always have a null value
in the updated field value until there is a change detected in the
record data</p></li>
</ul></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Changing table properties such as primary
key definitions</p></td>
<td class="confluenceTd"><p>Some metadata properties of a table may
change, such as the definition of the primary key which is a key part in
determining how the type 2 SCD loads is performed</p></td>
<td class="confluenceTd"></td>
<td class="confluenceTd"></td>
</tr>
</tbody>
</table>

</div>

## Solution Design Approach

### Combination of State and Migration Based Approach

The recommended approach would be to utilise and combine the aspects and
benefits from both the State and Migration based approach as discussed
above. Some key aspects that would be incorporated in the design are:

-   The target state of the database structure will be automatically
    derived from the latest configuration objects

-   Developers will have control and visibility of the delta script
    required to migrate the database structure from the current state to
    the target state.

-   A config file is required within the code repository which will
    store the git hash ID of the commit for the database structure from
    the previous state.

    -   A blank config file indicates that there is no previous state of
        the database.

-   Validation and testing is performed to ensure the following:

    -   That the database structure created by the create DDL scripts
        align with the changes instructed by the alter DDL scripts.

    -   Alter DDL scripts can be executed successfully on the current
        state of the database without compromising data loss and
        integrity

-   Delta scripts are applied to the production instance only if the
    validation/test stage is successful and is manually QA and approved
    by the appropriate individuals/teams.

-   An additional table will be created that will store the value of the
    git hash ID of the commit that is currently deployed on the database

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio1181875787849328729"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio1181875787849328729"
class="ap-content">

</div>

</div>

## Solution Design Options

### Assumptions

-   PEGA BIX are delta loads

-   If there is a field name change in the PEGA Source System, this will
    not be included as part of the criteria in the delta load.

### Automated Vs Manual

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Scenarios</strong></p></th>
<th class="confluenceTh"><p><strong>Fully Automated
Pipeline</strong></p></th>
<th class="confluenceTh"><p><strong>Manual Generation of Delta SQL
Scripts with Automated Testing Framework</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"
data-highlight-colour="#f4f5f7"><p>Description</p></td>
<td class="confluenceTd" data-highlight-colour="#f4f5f7"><p>This method
will require a pipeline that is able to programmatically extract the
latest configuration objects and determine what scenario the changes to
the schema are classified as. Then it will generate the SQL scripts
required to deliver the expected behaviour above which should also
include some sort of manual QA process, test the execution of the SQL
Scripts and deploy the changes.</p></td>
<td class="confluenceTd" data-highlight-colour="#f4f5f7"><p>This method
will require a pipeline that is able to generate the Create DDL SQL
scripts based on up to date configuration objects, accompanied with
manual creation of Alter DDL SQL Scripts that will update the schema
from the current state to the target state by a developer. Testing of
the SQL Scripts will be automatically executed before it is deployed to
production.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"
data-highlight-colour="#e3fcef"><p>Pros</p></td>
<td class="confluenceTd" data-highlight-colour="#e3fcef"><ul>
<li><p>All the hard work is done by the compare tool, thus freeing the
developers of the additional work required to write the delta
scripts</p></li>
<li><p>Can ensure that the delta scripts are generated consistently
based on defined best practices</p></li>
</ul></td>
<td class="confluenceTd" data-highlight-colour="#e3fcef"><ul>
<li><p>Full control of the delta scripts provided to the
developers</p></li>
<li><p>Developers are able to utilise more contextual knowledge when
create delta scripts. For e.g. identifying when a column has been
renamed vs when a column has been added/removed</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"
data-highlight-colour="#ffebe6"><p>Cons</p></td>
<td class="confluenceTd" data-highlight-colour="#ffebe6"><ul>
<li><p>Outside of developing the component to generate the SQL commands,
Developers have lack of control over the delta script generator</p></li>
<li><p>Requires the component to be thoroughly tested and
reviewed.</p></li>
</ul></td>
<td class="confluenceTd" data-highlight-colour="#ffebe6"><ul>
<li><p>There may be a high number of schema changes from data sources
such as PEGA which will introduce excessive management and development
overhead.</p></li>
<li><p>Prone to human errors</p></li>
<li><p>Effectiveness of this approach is dependent on the script
creator's knowledge on the context of the change</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>New column added</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Programmatically determine what columns have been added in the
table config item</p></li>
<li><p>Generate both a new DDL statement that creates a new table with
the added column, and an alter statement on an existing table</p></li>
<li><p>Update the hash values or hash value configuration of the
existing table</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Automatically generate the new Create DDL Statement for the table
with the new column</p></li>
<li><p>Manually develop the alter script that will update the schema
from the current state to the target state, this may also include the
commands to update the hash value or hash value configuration</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
<li><p>Commit and push the updated DDL scripts to source code repository
for review and deployment</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Dropping a column that is nullable</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Programmatically determine what columns have been added in the
table config item</p></li>
<li><p>No changes to the structure of the table would be
required</p></li>
<li><p>Need to ensure that whenever inserts statements into Snowflake
tables always specify the columns that it is inserting the data in. This
will ensure that the dropped field in the source system will populate
with null values in Snowflake</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>No changes need to be made to the table</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Dropping a column that is not
nullable</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Programmatically check if the dropped field is a not nullable
field based on the config value</p></li>
<li><p>Generate both a new DDL statement that creates a new table with
the modified attributes on the column, and an alter statement on an
existing table</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
<li><p>Need to ensure that whenever inserts statements into Snowflake
tables always specify the columns that it is inserting the data in. This
will ensure that the dropped field in the source system will populate
with null values in Snowflake</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Manually develop the alter script that will update the schema
from the current state to the target state.</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
<li><p>Commit and push the updated DDL scripts to source code repository
for review and deployment</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Changing data type of a column</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Programmatically check if an existing fields data type has
changed</p></li>
<li><p>Generate both a new DDL statement that creates a new table with
the modified attributes on the column, and an alter statement on an
existing table</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
<li><p>Need to consider how to handles situations where the existing
data prohibits does not conform to the new data type. For e.g. if a
column is to be converted from a string to an int, but there are non
numerical characters in the column</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Manually develop the alter script that will update the schema
from the current state to the target state, this may also include the
commands to update the hash value or hash value configuration</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
<li><p>Commit and push the updated DDL scripts to source code repository
for review and deployment</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Renaming column names</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>There is no programmatic way of differentiating this scenario
with a combination of a new column added and an existing column
removed</p></li>
<li><p>Therefore in this particular scenario, it will be treated as a
combination of the new named field is a new column to be added and the
old named field is a column that is dropped.</p></li>
<li><p>Generate both a new DDL statement that creates a new table with
the added column, and an alter statement on an existing table</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
<li><p>Based on the assumptions above, future delta loads of data will
mean that only some of the active records will contain data in the new
column and null values in the old column, and vice versa. As such when
querying for this data, users may need to reference both the old and new
columns</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>The latest configuration objects are extracted from the
production environment</p></li>
<li><p>Manually develop the alter script that will update the schema
from the current state to the target state, however need to consider if
the alter table statement should create a new column that will now be
used to load the data that previously existed in the old column or
rename an existing column. If the latter option is chosen, it cannot be
automated should this be done in the future.</p></li>
<li><p>As part of the automated testing framework, zero copy clone the
current database, run the alter scripts and validate that they run
successfully and produce the target state as defined by the create sql
scripts</p></li>
<li><p>Commit and push the updated DDL scripts to source code repository
for review and deployment</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

### Recreate & Load Vs Alter existing table

<div class="table-wrap">

<table class="confluenceTable" data-layout="default">
<tbody>
<tr class="header">
<th class="confluenceTh"><p><strong>Scenarios</strong></p></th>
<th class="confluenceTh"><p><strong>Recreate &amp;
Load</strong></p></th>
<th class="confluenceTh"><p><strong>Alter clone of existing
table</strong></p></th>
<th class="confluenceTh"><p><strong>Alter existing
table</strong></p></th>
</tr>

<tr class="odd">
<td class="confluenceTd"
data-highlight-colour="#f4f5f7"><p>Description</p></td>
<td class="confluenceTd" data-highlight-colour="#f4f5f7"><p>This method
of updating the schema would require creating a new temporary table with
the update schema changes (i.e. new columns, updated column attributes
etc) and then loading data from the existing table into new table. Once
all the data is loaded, a manual review and approval process should be
implemented prior to replacing the new table with the existing table,
and then dropping the temporary table.</p></td>
<td class="confluenceTd" data-highlight-colour="#f4f5f7"><p>This method
of updating the schema would require creating a clone of the table,
making the required modifications to the cloned table (i.e. adding
columns, changing column attributes etc) using the generated alter
statements and then replacing the existing current table with the
modified clone table. A manual review and approval process should be
implemented prior to replacing the existing table.</p></td>
<td class="confluenceTd" data-highlight-colour="#f4f5f7"><p>This method
requires the generation of alter statements (either programmatically or
manually) and then applying these statements directly to the existing
table. A manual review and approval process should be implemented prior
to executing the alter statements.</p></td>
</tr>
<tr class="even">
<td class="confluenceTd"
data-highlight-colour="#e3fcef"><p>Pros</p></td>
<td class="confluenceTd" data-highlight-colour="#e3fcef"><ul>
<li><p>May be less error prone as it is the create DDL statements are
easier to generate then alter DDL statements</p></li>
<li><p>Snowflake’s time travel provides the ability to access historical
data (i.e. data that has been changed or deleted) up to 90 days.
Therefore any changes made in the last 90 days can be reverted back if
required.</p>
<ul>
<li><p>However need to consider the implication of how to reload data
back into the table when changes are reverted</p></li>
</ul></li>
</ul></td>
<td class="confluenceTd" data-highlight-colour="#e3fcef"><ul>
<li><p>Snowflake’s zero copy cloning capability enables fast replication
of database objects without replicating the data itself therefore
minimising storage costs</p></li>
<li><p>Snowflake’s time travel provides the ability to access historical
data (i.e. data that has been changed or deleted) up to 90 days.
Therefore any changes made in the last 90 days can be reverted back if
required.</p>
<ul>
<li><p>However need to consider the implication of how to reload data
back into the table when changes are reverted</p></li>
</ul></li>
<li><p>The alter statements generated can be easily reviewed as part of
the QA process</p></li>
<li><p>The alter statements are executed on a clone of the table, which
will have no effect on the existing table</p></li>
</ul></td>
<td class="confluenceTd" data-highlight-colour="#e3fcef"><ul>
<li><p>Snowflake’s time travel provides the ability to access historical
data (i.e. data that has been changed or deleted) up to 90 days.
Therefore any changes made in the last 90 days can be reverted back if
required.</p>
<ul>
<li><p>However need to consider the implication of how to reload data
back into the table when changes are reverted</p></li>
</ul></li>
<li><p>The alter statements generated can be easily reviewed as part of
the QA process</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"
data-highlight-colour="#ffebe6"><p>Cons</p></td>
<td class="confluenceTd" data-highlight-colour="#ffebe6"><ul>
<li><p>Manual QA process may be more difficult as you need to derive
what the schema differences are by comparing two tables</p></li>
<li><p>Additional storage costs will be incurred for temporarily storing
duplicated data as well as the compute costs required to load in data
from one table to the other.</p></li>
<li><p>Need to ensure no data loads are taking place until this process
is complete. If a new batch of data is loaded into the existing table
after the new table was loaded created and loaded with data, if this new
table is replaced, it will not contain the latest batch of data
resulting in data loss</p></li>
</ul></td>
<td class="confluenceTd" data-highlight-colour="#ffebe6"><ul>
<li><p>Need to ensure that the process of generating alter statements
has been rigorously tested, and the logic on how these statements were
generated are easily auditable and transparent.</p></li>
</ul></td>
<td class="confluenceTd" data-highlight-colour="#ffebe6"><ul>
<li><p>Need to ensure that the process of generating alter statements
has been rigorously tested, and the logic on how these statements were
generated are easily auditable and transparent.</p></li>
<li><p>The execution of the alter statements will not be tested and
validated prior to running it on production live tables/data</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>New column added</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a new column added to a table, the table level
configuration objects will be updated to include the new column and its
attributes.</p></li>
<li><p>Using the updated config object, a new DDL create temp table
statement will be generated and executed on snowflake</p></li>
<li><p>Data from the existing table will then be loaded into the new
temp table with the new column having default null values. The hash
values of the existing data will also need to be updated</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the temp table, and then
the temp table is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>Future data loads should behave as normal since it will rely on
the updated config object of the table</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a new column added to a table, the table level
configuration objects will be updated to include the new column and its
attributes.</p></li>
<li><p>The changes that have occurred to the configuration object will
then be used as input to a component that generates the alter statements
required to add the new columns</p></li>
<li><p>A clone of the existing table will then be created on Snowflake
before the generated alter statements are executed on the clone table.
The hash values of the existing data in the clone table will need to be
recalculated.</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the cloned table, and then
the clone is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>Future data loads should behave as normal since it will rely on
the updated config object of the table</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a new column added to a table, the table level
configuration objects will be updated to include the new column and its
attributes.</p></li>
<li><p>The changes that have occurred to the configuration object will
then be used as input to a component that generates the alter statements
required to add the new columns</p></li>
<li><p>Once the QA process has been approved, the alter statements to
add the new columns will be directly executed on the existing
table</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>Future data loads should behave as normal since it will rely on
the updated config object of the table</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Dropping a column that is nullable</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a field that is removed from the source system,
the table level configuration objects will be updated to exclude this
field as part of ETL loads. However this field will still remain in the
Snowflake tables for historical reporting. As such no changes are
required to be made on the Snowflake table</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>It is important that whenever data is being loaded/inserted in a
table in Snowflake, that the query explicitly specifies the columns it
is loading the data in. This will ensure that for columns that are not
specified in the command (i.e. deleted fields from source systems), null
values will be automatically inserted in the column.</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a field that is removed from the source system,
the table level configuration objects will be updated to exclude this
field as part of ETL loads. However this field will still remain in the
Snowflake tables for historical reporting. As such no changes are
required to be made on the Snowflake table</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>It is important that whenever data is being loaded/inserted in a
table in Snowflake, that the query explicitly specifies the columns it
is loading the data in. This will ensure that for columns that are not
specified in the command (i.e. deleted fields from source systems), null
values will be automatically inserted in the column.</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a field that is removed from the source system,
the table level configuration objects will be updated to exclude this
field as part of ETL loads. However this field will still remain in the
Snowflake tables for historical reporting. As such no changes are
required to be made on the Snowflake table</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>It is important that whenever data is being loaded/inserted in a
table in Snowflake, that the query explicitly specifies the columns it
is loading the data in. This will ensure that for columns that are not
specified in the command (i.e. deleted fields from source systems), null
values will be automatically inserted in the column.</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Dropping a column that is not
nullable</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a field that is removed from the source system,
the table level configuration objects will be updated to exclude this
field as part of ETL loads.</p></li>
<li><p>This field will still remain in the Snowflake tables for
historical reporting however if it was a “Not Nullable” field, then a
new DDL create temp table statement will be generated and executed on
Snowflake that contains the column that has been removed from the source
system, with a modification of its nullable attribute from “False” to
“True”</p></li>
<li><p>Data from the existing table will then be loaded into the new
temp table</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the temp table, and then
the temp table is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>To allow null values to be inserted into the column, its
attributes need to be updated prior to future data loads</p></li>
<li><p>It is important that whenever data is being loaded/inserted in a
table in Snowflake, that the query explicitly specifies the columns it
is loading the data in. This will ensure that for columns that are not
specified in the command (i.e. deleted fields from source systems), null
values will be automatically inserted in the column.</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a field that is removed from the source system,
the table level configuration objects will be updated to exclude this
field as part of ETL loads.</p></li>
<li><p>This field will still remain in the Snowflake tables for
historical reporting however if it was a “Not Nullable” field, alter
statements need to be generated to modify the existing column</p></li>
<li><p>A clone of the existing table will then be created on Snowflake
before the generated alter statements are executed on the clone
table.</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the cloned table, and then
the clone is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>To allow null values to be inserted into the column, its
attributes need to be updated prior to future data loads</p></li>
<li><p>It is important that whenever data is being loaded/inserted in a
table in Snowflake, that the query explicitly specifies the columns it
is loading the data in. This will ensure that for columns that are not
specified in the command (i.e. deleted fields from source systems), null
values will be automatically inserted in the column.</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever there is a field that is removed from the source system,
the table level configuration objects will be updated to exclude this
field as part of ETL loads.</p></li>
<li><p>This field will still remain in the Snowflake tables for
historical reporting however if it was a “Not Nullable” field, alter
statements need to be generated to modify the existing column</p></li>
<li><p>Once the QA process has been approved, the alter statements to
add the new columns will be directly executed on the existing
table</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>To allow null values to be inserted into the column, its
attributes need to be updated prior to future data loads</p></li>
<li><p>It is important that whenever data is being loaded/inserted in a
table in Snowflake, that the query explicitly specifies the columns it
is loading the data in. This will ensure that for columns that are not
specified in the command (i.e. deleted fields from source systems), null
values will be automatically inserted in the column.</p></li>
</ul></td>
</tr>
<tr class="odd">
<td class="confluenceTd"><p>Changing data type of a column</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever the datatype of a field has changed, the table level
configuration objects will be updated to update the attributes of the
respective column</p></li>
<li><p>Using the updated config object, a new DDL create temp table
statement will be generated and executed on snowflake</p></li>
<li><p>Data from the existing table will then be loaded into the new
temp table with the updated column type. The hash values of the existing
data may also need to be recalculated.</p></li>
<li><p>If historical data cannot be loaded in the updated column because
of a datatype mismatch, an error will occurred which will need to be
handled manually</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the temp table, and then
the temp table is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>Future data loads should behave as normal since it will rely on
the updated config object of the table</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever the datatype of a field has changed, the table level
configuration objects will be updated to update the attributes of the
respective column</p></li>
<li><p>The changes that have occurred to the configuration object will
then be used as input to a component that generates the alter statements
required to update the existing column</p></li>
<li><p>A clone of the existing table will then be created on Snowflake
before the generated alter statements are executed on the clone table.
The hash values of the existing data in the clone table may need to be
recalculated.</p></li>
<li><p>If the column in the clone table cannot be altered because of a
datatype mismatch with the existing historical data, an error will
occurred which will need to be handled manually</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the cloned table, and then
the clone is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>Future data loads should behave as normal since it will rely on
the updated config object of the table</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>Whenever the datatype of a field has changed, the table level
configuration objects will be updated to update the attributes of the
respective column</p></li>
<li><p>The changes that have occurred to the configuration object will
then be used as input to a component that generates the alter statements
required to modify existing columns</p></li>
<li><p>Once the QA process has been approved, the alter statements to
add the new columns will be directly executed on the existing
table</p></li>
<li><p>If the column in the clone table cannot be altered because of a
datatype mismatch with the existing historical data, an error will
occurred which will need to be handled manually</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>Future data loads should behave as normal since it will rely on
the updated config object of the table</p></li>
</ul></td>
</tr>
<tr class="even">
<td class="confluenceTd"><p>Renaming column names</p></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>If a field in the source system has been renamed, this will be
interpreted as both a new column added and an existing column removed.
The table level configuration objects will therefore add attributes for
a new column and remove the attributes of the replaced column.</p></li>
<li><p>Using the updated config object, a new DDL create temp table
statement will be generated and executed on snowflake that includes the
newly added column, as well as the removed column</p></li>
<li><p>Data from the existing table will then be loaded into the new
temp table with the new column having default null values. The hash
values of the existing data will also need to be updated</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the temp table, and then
the temp table is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>If a full snapshot load batch job has occurred, all the active
records will be reinserted into the table as the hash values have now
changed, since the values from the old column will migrate to the new
column, and the old columns will default to null values. This may cause
an increase in ETL job duration times depending on the data
size</p></li>
<li><p>If it is a delta snapshot, based on the assumptions above, then
future delta loads of data will mean that only some of the active
records will contain data in the new column and null values in the old
column, and vice versa. As such when querying for this data, users may
need to reference both the old and new columns</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>If a field in the source system has been renamed, this will be
interpreted as both a new column added and an existing column removed.
The table level configuration objects will therefore add attributes for
a new column and remove the attributes of the replaced column.</p></li>
<li><p>The changes that have occurred to the configuration object will
then be used as input to a component that generates the alter statements
required to add the new column</p></li>
<li><p>A clone of the existing table will then be created on Snowflake
before the generated alter statements are executed on the clone table.
The hash values of the existing data in the clone table will need to be
recalculated.</p></li>
<li><p>Assume the QA process has been approved, a SQL command will be
generated to replace the existing table with the cloned table, and then
the clone is dropped</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>If a full snapshot load batch job has occurred, all the active
records will be reinserted into the table as the hash values have now
changed, since the values from the old column will migrate to the new
column, and the old columns will default to null values. This may cause
an increase in ETL job duration times depending on the data
size</p></li>
<li><p>If it is a delta snapshot, based on the assumptions above, then
future delta loads of data will mean that only some of the active
records will contain data in the new column and null values in the old
column, and vice versa. As such when querying for this data, users may
need to reference both the old and new columns</p></li>
</ul></td>
<td class="confluenceTd"><p>Schema Change Process &amp;
Considerations:</p>
<ul>
<li><p>If a field in the source system has been renamed, this will be
interpreted as both a new column added and an existing column removed.
The table level configuration objects will therefore add attributes for
a new column and remove the attributes of the replaced column.</p></li>
<li><p>The changes that have occurred to the configuration object will
then be used as input to a component that generates the alter statements
required to add the new column</p></li>
<li><p>Once the QA process has been approved, the alter statements to
add the new column will be directly executed on the existing
table</p></li>
</ul>
<p>Future Data Loads:</p>
<ul>
<li><p>If a full snapshot load batch job has occurred, all the active
records will be reinserted into the table as the hash values have now
changed, since the values from the old column will migrate to the new
column, and the old columns will default to null values. This may cause
an increase in ETL job duration times depending on the data
size</p></li>
<li><p>If it is a delta snapshot, based on the assumptions above, then
future delta loads of data will mean that only some of the active
records will contain data in the new column and null values in the old
column, and vice versa. As such when querying for this data, users may
need to reference both the old and new columns</p></li>
</ul></td>
</tr>
</tbody>
</table>

</div>

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Semi Automated Schema Evolution
Design.drawio.tmp](attachments/93421584/304152592.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Semi Automated Schema Evolution
Design.drawio.tmp](attachments/93421584/304349201.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Semi
Automated Schema Evolution
Design.drawio](attachments/93421584/301892553.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Semi
Automated Schema Evolution
Design.drawio.png](attachments/93421584/304447496.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Manual Schema Evolution Process
Flow Design.drawio.tmp](attachments/93421584/304185359.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Manual Schema Evolution Process
Flow Design.drawio.tmp](attachments/93421584/304152601.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Manual Schema Evolution Process
Flow Design.drawio.tmp](attachments/93421584/304250902.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Manual
Schema Evolution Process Flow
Design.drawio](attachments/93421584/301663203.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Manual
Schema Evolution Process Flow
Design.drawio.png](attachments/93421584/301663211.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[image-20201008-030635.png](attachments/93421584/304447571.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [State
based approach.png](attachments/93421584/301663407.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Migration Based Approach.png](attachments/93421584/304283682.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304185378.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304283724.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio](attachments/93421584/301466688.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio.png](attachments/93421584/304447688.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio](attachments/93421584/304349442.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio.png](attachments/93421584/304152813.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio](attachments/93421584/304414942.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio.png](attachments/93421584/301663653.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio](attachments/93421584/304185572.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio.png](attachments/93421584/304316682.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304185548.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/301663679.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/301892819.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304284010.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304251121.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304349486.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304185562.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304447838.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/301466826.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Schema Evolution
Design.drawio.tmp](attachments/93421584/304185387.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio](attachments/93421584/301663432.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Schema
Evolution Design.drawio.png](attachments/93421584/304316444.png)
(image/png)  

</div>
