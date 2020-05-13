# Content Blocks Registration in TYPO3

This is a draft for registration of content blocks as (composer) packages. Please read the whole document and pay 
attention to the [FAQs](FAQ.md) before creating issues.

## The Motivation

> Defining "Content Elements" in TYPO3 can be hard and the learning curve is steap.
> You need to learn PHP, TCA, TypoScript and Fluid and maybe other languages.

So the goal of this initiative is to provide an easy and reliable way to register content blocks.

A content block is defined as a small chunk of informatation, which is connected to a view, which is then rendered in the TYPO3 frontend.

Additionally this we hope to encourage people to start making custom content elements by providing a clean and easy to understand way. This might open TYPO3 to new users and target groups.

## The idea

The goal is to define content elements (further called content block) easier by simplifying and reducing the configuration
via abstraction and convention.

To achieve that goal, the idea is to define a content block as a reusable composer package, which should be usable 
standalone and needs all its dependencies to be defined.

This idea is heavily inspired by the web components approach, even if here it is done on a different level.



## Directory structure of a content block

| Directory / File | mandatory | could be created by kickstarter |
| ------------- |:-------------:| :-----:|
| composer.json | x | x |
| ContentBlockIcon.(svg/png/gif) | x | x |
| EditorInterface.yaml | x | x |
| src/Language/Default.xlf | x | x |
| src/EditorPreview.html | x | x |
| src/Frontend.html |  | x |
| dist/EditorPreview.css |  | x |
| dist/Frontend.css |  | x |
| dist/Frontend.js |  | x |

## Storage of content block in the TYPO3 directory structure

Each content block is described in a separate composer package. These packages must define their type property as 
“typo3-contentblock”. TYPO3 then uses a custom composer installer to place these packages in a location different 
to normal libraries and also not in the typo3conf/ext folder.

This approach makes it easier to separate the working directories for “classic extensions” (plugins, …) and content blocks.

This is also compatible with the local package repository approach you would normally use if you ship packages, 
which are very specific to a single project.

The suggested structure is:

|  |  |
| ------------- |-------------|
| For non composer installations | typo3conf/contentBlocks/<packages> |
| For composer installations | config/contentBlocks/<packages> |


### Positive side effects of this approach

* You can render the content blocks without having a complete TYPO3 installed yet
* You may reuse the content blocks in other projects
* You can define review rules in common vcs so that the frontend engineers are notified for changes



## Content block package files explained

### composer.json
refers to: https://getcomposer.org/doc/04-schema.md 

The content block ID (CType) derives from the package name. Therefore one composer package represents exactly one content block.

**You must**
* provide that file
* set the type property to: typo3-cms-contentblock

**You may**
* use the full composer.json config and define autoloading for ViewHelpers etc.

### EditorInterface.yaml

refers to: https://github.com/yaml/summit.yaml.io/wiki/YAML-RFC-Index 

**You must**
* provide that file
* define the editor interface of exactly one content block
* define all the fields and their position in the editing interface


### ContentBlockIcon.(svg|png|gif)

There is no fallback by intention, but it is easy to generate an SVG with the content block name as a graphical representation.


**You must**
* provide that file
* provide that file in the format svg or png or gif
* provide a file with 1:1 dimensions


### src/Language/Default.xlf

**You may**
* provide that file
* define your labels with the xlf links in the configuration file   


## Abstraction Requirements
To achieve the goal of reducing the complexity of content block registration 
the [facade pattern](https://en.wikipedia.org/wiki/Facade_pattern) approach needs to be used for some of TYPO3s internal APIs.

These are

*   mapping to the database
* TCA generation for 
    * ext_tables.php
    * Configuration/TCA/….
    * registration of the icon in the CType field in TCA
* registration of the plugin to display the content for
    * frontend rendering including DataProcessors
* new content element wizard (pageTS)
    * registration of the icon in the new content element wizard
* configuration of the template path(s)
* registration for the preview in the backend
    * https://review.typo3.org/c/Packages/TYPO3.CMS/+/50389


## Processes that happen during content block registration

### Detecting a content block

The detection of content blocks depends on the composer package type. The custom composer installer then retrieves all packages, which are of the above defined type.

### Mapping to the database
There are several variants of how data of a content block can be stored and retrieved from the database. 
After careful evaluation the approach of a JSON blob using a modern database was chosen, 
however the alternatives are listed below in case the favoured approach would turn out bad in the proof of concept.

#### JSON blob
All fields defined in the EditorIterface.yaml of a particular content block are stored as JSON blob (nosql-like) in a specified tt_content field. The content block automatically receives a DataProcessor that retrieves the data from the JSON blob as array of values and objects.

This approach avoids that fields with the same identifier of different content block cause conflicts on database level. Also, the size of tt_content isn’t widely extended, if a huge amount of custom content blocks defining new fields are registered, thus giving up the rDBMS approach for those use cases.

#### Alternative: virtual SQL generation for ext_tables.sql extending tt_content
Fields defined in the EditorIterface.yaml are mapped to the tt_content table via a virtual ext_tables.sql file. Therefore a hook to inject SQL into the database schema manager is needed. New fields are automatically created on installation. Changes need to be made visible in the install tool, also an accidental deletion of fields from the install tool has to be avoided.

This approach allows re-using already existing fields in tt_content, but forces the creator to pay attention to field definitions in TCA for existing fields.

#### Alternative: virtual SQL generation for ext_tables.sql creating custom table
Fields defined in the EditorIterface.yaml are mapped to a new table. That way, each content block stores its data in an own table. To connect the content block data with tt_content a specified tt_content fields are used to store the connected table name and the uid of the content block data.

This approach avoids that fields with the same identifier of different content block cause conflicts on database level. Also, the size of tt_content isn’t widely extended, if a huge amount of custom content blocks defining new fields are registered, thus giving up the rDBMS approach for those use cases.

#### Alternative: Entity-Attribute-Value (EAV)

See following links for more information:
* https://designpatternsphp.readthedocs.io/de/latest/More/EAV/README.html
* https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model


### Virtual generation of TCA (ext_tables.php)

Requirements:
* Has to be after non override TCA loading
* Has to be before the caching of the TCA
* Has to be before merging the overrides for TCA

TCA is virtually generated from the class implementing a content element block field type.

### Generate registration of the plugin

Requirements:
* Register icon
* Add TCA entry in CTypes list including the icon
* Register plugin in TYPO3
* Add TypoScript to render the content plugin
* Add PageTS for the content block
    * Define where to display (group / location) the content block in the new content element wizard
