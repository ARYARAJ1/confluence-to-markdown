# PEGA XML Datafeed Processing - Ingestion

The section contains the high level pseudocode design for how the PEGA
XML data extracts will be processed and transformed. Below provides an
overview of the overall XML Datafeed ingestion pipeline:

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio8741671513719176211"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio8741671513719176211"
class="ap-content">

</div>

</div>

## Pega XML XSD difference checker

Before creating new Configuration Data Objects there will be a check to
see if the schema provided by PEGA has changed since the last time the
Configuration Data Objects were created. If there is a change in the
schema, this will trigger the component responsible for creating an
updated version of the configuration data objects with the new schema
file before proceeding with the rest of the XML ingestion pipeline up
until loading and structuring the data in Snowflake. Loading the data
into the curated layer will not be occur until the schema of the curated
tables have been updated to align with the schema changes. Once this is
complete, the pipeline can then recommence with the reset of the
ingestion pipeline.

### Building XML XSD difference checker

Below is the pseudocode diagram that defines how the old and new XSD
files will be compared to check if there is a difference.

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio7216254035216635035"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio7216254035216635035"
class="ap-content">

</div>

</div>

## Pega XML Schema File to Configuration Data Objects

Alongside the XML files, there will also be a schema file (the xsd
mentioned above) provided by PEGA that will be used to validate that the
data complies with the expected schema. This schema comes in the form of
an XSD file and it is a key metadata object that will be used to build
the configuration data objects for the acqusition of the PEGA data
source into the data platform. The defusedxml
(<a href="https://pypi.org/project/defusedxml/#defusedxml-elementtree"
rel="nofollow">https://pypi.org/project/defusedxml/#defusedxml-elementtree</a>)
python package will be used to deliver the required functionality.

### Building Configuration Object for the SOR Master Table

Below is the pseudocode diagram that defines how the XSD will be
processed to identify the metadata for the SOR Master Table.

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio4364266258449029669"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio4364266258449029669"
class="ap-content">

</div>

</div>

### Building Configuration Objects for the SOR Sub Tables

Below is the pseudocode diagram that defines how the XSD will be
processed to identify the metadata for all the subtables of the
respective SOR.

<div
id="ap-com.mxgraph.confluence.plugins.diagramly__drawio8323293182970202475"
class="ap-container">

<div
id="embedded-com.mxgraph.confluence.plugins.diagramly__drawio8323293182970202475"
class="ap-content">

</div>

</div>

## Stripping outer element of XML file in Snowflake

As some of the XML files from PEGA have multiple levels of nested
elements, this will convert into multiple tables for a single XML file.
Additionally some of the files will have large amounts of data and by
default Snowflake would load all the contents of the data into a single
row and column. Although Snowflake has a size limit of 16 MB for a
Variant column type, it does have the option to strip the outer element
of the XML data as part of transformation process to copy the data into
a table. Therefore a single item element from the XML file will
translate into a single row within Snowflake. Below is the sample SQL
code that will be used to enable this:

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeContent panelContent pdl">

``` sql
Copy into table
  from @MY_XML_STAGE/TestBIX.xml.gz
  file_format = (type = 'XML' STRIP_OUTER_ELEMENT = true);
```

</div>

</div>

## Structuring XML file with XSD and configuration objects

Below is a sample of what the dynamic SQL query would look like in
Snowflake to structure XML data for a single table. The number of
lateral flatten statements will depend on how nested the data elements
for the respective table is within the XML. This will be identified via
the XSD to configuration conversion process above.

<div class="code panel pdl" style="border-width: 1px;">

<div class="codeContent panelContent pdl">

``` sql
select 
    GET( XMLGET( xmldata, 'MasterCustomerID'), '$') as "MasterCustomerID",
    GET(xml_doc.value, '@id'),
    XMLGET(target_table.value, 'Column1'):"$",
    XMLGET(target_table.value, 'Column2'):"$",
    XMLGET(target_table.value, 'Column3'):"$",
    XMLGET(target_table.value, 'Column4'):"$"
from xmltable,
lateral flatten(to_array(xmltable.xmldata:"$")) xml_doc,
lateral flatten(to_array(xml_doc.VALUE:"$")) nested_level_1
lateral flatten(to_array(nested_level_1.VALUE:"$")) nested_level_2
.
.
lateral flatten(to_array(nested_level_n-1.VALUE:"$")) nested_level_n
lateral flatten(to_array(nested_level_n.VALUE:"$")) target_table
where GET(target_table.value, '@') = '<Element Tag Name>'
```

</div>

</div>

### Design Notes:

-   The number of lateral flatten commands to run will be determined
    based on how deeply nested the target table is. This will be
    determined by the Azure function that parses the XML Schema file
    (XSD file) and stores this value within the specific configuration
    objects for each table.

-   In order to link the child element tables to the root element
    tables, some of the master root column fields will be propagated to
    the child tables. In the example above, this column is the
    “MasterCustomerID”.

<div class="pageSectionHeader">

## Attachments:

</div>

<div class="greybox" align="left">

<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/205520902.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio](attachments/199131137/198574114.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio.png](attachments/199131137/205684768.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/205520897.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/205717541.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/205520942.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio](attachments/199131137/205586486.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio.png](attachments/199131137/198869027.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio](attachments/199131137/205717590.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio.png](attachments/199131137/205586496.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/198508588.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio](attachments/199131137/216367162.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio.png](attachments/199131137/216563782.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~XML
Splitter.drawio.tmp](attachments/199131137/210698243.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Splitter.drawio](attachments/199131137/210632709.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Splitter.drawio.png](attachments/199131137/210960385.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5dc205b07dcf390c61845cd1\~XML
Splitter.drawio.tmp](attachments/199131137/210665473.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Splitter.drawio](attachments/199131137/210829334.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Splitter.drawio.png](attachments/199131137/210894938.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210960404.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210567220.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210632721.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210632731.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210927694.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210927704.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210567230.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210731045.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210894928.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210862180.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Splitter.drawio](attachments/199131137/210731011.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Splitter.drawio.png](attachments/199131137/210763777.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Splitter.drawio.tmp](attachments/199131137/210599939.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio](attachments/199131137/210894987.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio.png](attachments/199131137/210731074.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/208470056.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio](attachments/199131137/210829359.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio.png](attachments/199131137/210895006.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210927733.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210960435.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210600047.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210862228.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210862238.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210567297.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210731105.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/208470087.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210862248.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210731115.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio](attachments/199131137/210763856.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio.png](attachments/199131137/210829406.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~XML
Structuring.drawio.tmp](attachments/199131137/210731060.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio](attachments/199131137/210993226.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [XML
Structuring.drawio.png](attachments/199131137/210829350.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210665589)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210763920)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210600184)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210763930)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210632857)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210927788)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/208470138)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210763940)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/208470148)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[blob](attachments/199131137/210698264)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/210698475.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/210895140.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/210698466.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/210895159.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/210567408.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/210567388.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/212205569.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/211943435.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/212074497.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/211976219.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/211943445.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/212238360.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/212336655.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/211943455.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/212008997.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/211943477.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/212074576.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/212172875.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/211943467.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/216334339.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/216530945.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/216498187.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/216498197.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/216498221.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/216301616.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216563727.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216530958.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216367131.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216498211.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216465413.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216301606.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216367141.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216530968.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/216563747.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio](attachments/199131137/210632867.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline.drawio.png](attachments/199131137/210665599.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/301728378.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216301630.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216334399.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216301640.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216465437.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216531003.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216334409.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216301650.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216563772.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216301660.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio](attachments/199131137/205422599.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/216301670.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[Pseudocode XSD to
Config.drawio.png](attachments/199131137/198508564.png) (image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~Pseudocode XSD to
Config.drawio.tmp](attachments/199131137/198672402.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Pega
Sub Table Config
Pseudocode.drawio](attachments/199131137/216334473.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Pega
Sub Table Config
Pseudocode.drawio.png](attachments/199131137/216367217.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/216334464.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/216367199.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/216563817.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/216367208.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/216334487.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Pega
Sub Table Config
Pseudocode.drawio](attachments/199131137/696254527.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Pega
Sub Table Config
Pseudocode.drawio.png](attachments/199131137/696451180.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/696451166.tmp)
(application/octet-stream)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [xml
diff checker.drawio](attachments/199131137/301695586.drawio)
(application/vnd.jgraph.mxfile.cached)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [xml
diff
checker.drawio-4742c164ab2d148582ab0d7574dbb134037588bc.png](attachments/199131137/301793871.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [xsd
checker pseudo.drawio](attachments/199131137/301662744.drawio)
(application/vnd.jgraph.mxfile.cached)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [xsd
checker
pseudo.drawio-4742c164ab2d148582ab0d7574dbb134037588bc.png](attachments/199131137/301662749.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[1602039014935-xsd checker
pseudo.drawio](attachments/199131137/301793880.drawio)
(application/vnd.jgraph.mxfile.cached)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[1602039014935-xsd checker
pseudo.drawio-6bd802b23e3de3e9e2733d37ac0bde213c77009a.png](attachments/199131137/301892100.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline.drawio.tmp](attachments/199131137/210665604.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline2.drawio](attachments/199131137/301892300.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline2.drawio.png](attachments/199131137/301892310.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f76b931e31b69006f9a1175\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301761102.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f76b931e31b69006f9a1175\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301498966.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XSDdifferenceChecker.drawio](attachments/199131137/301662773.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XSDdifferenceChecker.drawio.png](attachments/199131137/301826627.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~drawio\~5f76b931e31b69006f9a1175\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301761097.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XSDdifferenceChecker.drawio](attachments/199131137/301794263.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XSDdifferenceChecker.drawio.png](attachments/199131137/301400965.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Screen
Shot 2020-10-07 at 4.11.36 pm.png](attachments/199131137/301761225.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Screen
Shot 2020-10-07 at 4.12.08 pm.png](attachments/199131137/301335200.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301728608.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301794080.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301892283.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301728617.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301400832.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301761231.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301695789.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301335231.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XML_Ingestion_Pipeline2.drawio.tmp](attachments/199131137/301400819.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline2.drawio](attachments/199131137/423920815.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline2.drawio.png](attachments/199131137/431816903.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301794229.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301892456.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301761335.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301335354.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301794242.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301400940.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301663123.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301663132.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301663141.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[\~XSDdifferenceChecker.drawio.tmp](attachments/199131137/301794101.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XSDdifferenceChecker.drawio](attachments/199131137/301662761.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XSDdifferenceChecker.drawio.png](attachments/199131137/301498962.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline2.drawio](attachments/199131137/301892109.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" />
[XML_Ingestion_Pipeline2.drawio.png](attachments/199131137/301400660.png)
(image/png)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/696287350.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/696156259.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/688488616.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/696385598.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [\~Pega
Sub Table Config
Pseudocode.drawio.tmp](attachments/199131137/216498235.tmp)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Pega
Sub Table Config
Pseudocode.drawio](attachments/199131137/216563795.drawio)
(application/vnd.jgraph.mxfile)  
<img src="images/icons/bullet_blue.gif" width="8" height="8" /> [Pega
Sub Table Config
Pseudocode.drawio.png](attachments/199131137/216563803.png)
(image/png)  

</div>
