# Data storage variants

Summary of pros and cons for different data storage approaches based on currently highly discussed questions.


## Extending tt_content

This approach is currently the default way to create additional fields to the fields provided by tt_content.
It allows re-using already existing fields in tt_content, but forces the creator to pay attention to field definitions 
in TCA for existing fields.

In the case of Content Blocks, fields defined in the EditorInterface.yaml would be mapped to the tt_content table 
via a virtual ext_tables.sql file. Therefore, a hook to inject SQL into the database schema manager wil be needed. 
New fields would be automatically created on installation. Changes need to be made visible in the Install Tool, 
also an accidental deletion of fields from the Install Tool has to be avoided.

| Pros | Cons |
| ------------- |-------------|
| Fields can be freely organized in the editing interface | Field definitions need to be named carefully to avoid duplicates (e.g. by using a vendor prefix) |
| Supports “native” CType switching | Size of tt_content is widely extended |
| Native DBMS support (performance, query/order single field with index) | In FSC: data processing always loads all fields of a tt_content row, no matter if they are needed by the CType or not (but can be configured, but you have to know the fields to exclude or include, what is with 3rd party extensions) |
| | Currently no validation schema for the editing interface definition / no validation feedback happens during the registration process |

## Custom table per content block

In the case of Content Blocks, fields defined in the EditorInterface.yaml are mapped to a new table. 
That way, each content block stores its data in an own table. To connect the content block data with tt_content 
a specified tt_content fields would be used to store the connected table's name and the uid of the content block data.

| Pros | Cons |
| ------------- |-------------|
| Field identifiers need to be unique only within a content block definition avoiding conflicts on database level | Currently no validation schema for the editing interface definition |
| Size of tt_content isn’t widely extended | Increases complexity for data handling |
| Native DBMS support (performance on single field query/order) |  |


## Entity-Attribute-Value (EAV)

See following links for more information:
* https://designpatternsphp.readthedocs.io/de/latest/More/EAV/README.html
* https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model

## Storing Blobs

In the case of Content Blocks, fields defined in the EditorInterface.yaml of a particular content block would be stored 
as a JSON or XML blob (nosql-like) in a specified tt_content field (e.g. content). The content block automatically 
receives a DataProcessor that retrieves the data from the blob as array of values and objects.

The blob storage approach would mean tt_content could be looking like this (in the future):

* uid
* pid
* ctype
* **content** → actual content information/ fields defined in the editing interface
* colPos
* starttime
* endtime
* fe_group
* editlock
* list_type
* categories
* category_field
* hidden
* deleted
* records
* pages
* row_description
* language fields  (sys_language_uid, l10n_parent, ..)
* history fields
* versioning fields

This basically means there are still meta-data fields (“enable fields”) still stored as separate fields, 
next to the actual payload (“content”) stored as blob.

### XML blob

| Pros | Cons |
| ------------- |-------------|
| Field identifiers need to be unique only within a content block definition avoiding conflicts on database level | Currently no support by doctrine/dbal |
| Size of tt_content isn’t widely extended | CType switching required mapping of content blocks |
| All of the relevant database management systems have support for XML storage and handling | |
| Works als with older databases | ExtractValue only returns values. Thus the whole FlexForm XML would have to be retrieved and extracted in PHP - using a DOM parser without DTD/XSD - see [reference](https://dev.mysql.com/doc/refman/5.6/en/xml-functions.html). |
| Reduces complexity for content dimensions (workspaces, language) → also is a con | Currently no validation of editing interface definition (schema definition) |
| Allows limiting editor permissions for a certain field (appears as an exclude-field) | Blob handled as one unit limits to make use of content dimensions (see pros) |
|  | Performance: full table scan required to retrieve data |


### JSON blob

| Pros | Cons |
| ------------- |-------------|
| Field identifiers need to be unique only within a content block definition avoiding conflicts on database level | Currently no support by doctrine/dbal |
| Size of tt_content isn’t widely extended | CType switching requires mapping of content blocks |
| Most/all of the relevant database management systems have support for JSON storage and handling | | 
| JSON_EXTRACT returns all relevant indices | Requires a modern database, e.g. [MySQL >= v5.7](https://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html), [PostgreSQL >=v9.3](https://www.postgresql.org/docs/9.3/functions-json.html), [SQLite >= v3.9.0 via json1 extension](https://www.sqlite.org/json1.html), [SQL Server >= v13.x (SQL Server 2016)](https://docs.microsoft.com/en-us/sql/t-sql/functions/json-functions-transact-sql?view=sql-server-2016) |
| JSON data type has more functions for manipulating or merging results in a database query than XML- see [reference](https://dev.mysql.com/doc/refman/5.7/en/json.html). | Performance: full table scan required to retrieve data |
| JSON information can easily be handed over to a client consumer (e.g. “some JavaScript” ) → very interesting for headless approach | Limiting editor permissions for a certain field required extra logic. |
