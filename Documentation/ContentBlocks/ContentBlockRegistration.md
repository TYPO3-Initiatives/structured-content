# Content Blocks Registration in TYPO3

This is a draft for registration of content blocks as (composer) packages. Please read the whole document and pay 
attention to the [FAQs](FAQ/FAQ_index.md) before creating issues.

## The Motivation

> Defining "Content Elements" in TYPO3 can be hard and the learning curve is steep.
> You need to learn PHP, TCA, TypoScript and Fluid and maybe other languages.

So the goal of this draft is to provide an easy and reliable way to register content blocks.

A content block is defined as a small chunk of information, which is connected to a view and then rendered in the TYPO3 frontend.

Additionally we hope to encourage people to start making custom content elements by providing a clean and easy to understand way. This might open TYPO3 to new users and target groups.

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
`typo3-cms-contentblock`. TYPO3 then uses a custom composer installer to place these packages in a dedicated location.
 
To separate the working directories for “classic extensions” (plugins, …), normal libraries and content blocks, 
the target folder is `typo3conf/contentBlocks/`. This is also compatible with the local package repository approach you 
would normally use if you ship packages, which are very specific to a single project.


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
* set the type property to: `typo3-cms-contentblock`
* require `"typo3-contentblocks/contentblocks-reg-api": "*"`

**You may**
* use the full composer.json config and define autoloading for ViewHelpers etc.

### EditorInterface.yaml

refers to: https://github.com/yaml/summit.yaml.io/wiki/YAML-RFC-Index 

**You must**
* provide that file
* define the editor interface of exactly one content block
* define all the fields and their position in the editing interface

The field types for the EditorInterface.yaml are heavily inspired by the [Symfony field types](https://symfony.com/doc/current/reference/forms/types.html) and will be mapped to TCA. See [here](FieldTypeMapping.md) for the mapping overview.

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
* Registration of the plugin to display the content for
    * frontend rendering including DataProcessors
* New content element wizard (pageTS)
    * registration of the icon in the new content element wizard
* Configuration of the template path(s)
* [Registration for the preview in the backend](https://review.typo3.org/c/Packages/TYPO3.CMS/+/50389)


## Processes that happen during content block registration

### Detecting a content block

The detection of content blocks depends on the composer package type. The custom composer installer then retrieves all 
packages, which are of the above defined type.

### Mapping to the database

There are [several variants](DataStorageVariants.md) of how data of a content block can be stored and retrieved from the database. 

Currently, there is no decision on the desired storage method, because performance research is still in progress.

See [possible variants](DataStorageVariants.md) to store date in the database for pros and cons.

### Virtual generation of TCA (ext_tables.php)

Requirements:
* Has to be after non override TCA loading
* Has to be before the caching of the TCA
* Has to be before merging the overrides for TCA

TCA is virtually generated from the class implementing a content block field type.

### Generate registration of the plugin

Requirements:
* Register icon
* Add TCA entry in CTypes list including the icon
* Register plugin in TYPO3
* Add TypoScript to render the content plugin
* Add PageTS for the content block
    * Define where to display (group / location) the content block in the new content element wizard


## Implementation roadmap

In order to speed up the development process, being able to get tester's feedback faster and react on it, the  
implementation process is split into following phases. Those phases represent a rough roadmap and may 
be adapted during the development process.

### Phase 1

* Composer installer for Content Blocks
* Validation of editing interface YAML
* Extend tt_content with a field content_block
* Use Flexforms and XML blob to store data
    * or eventually and individual use of the FormEngine
* Generate TSConfig
* Generate TypoScript

### Phase 2

This phase might be skipped. The decision depends on the research results for data storage.

* Use JSON blob instead of XML blob ([FlexForm storage method driver](https://review.typo3.org/c/Packages/TYPO3.CMS/+/53813))

Phase 2 may, if deemed vital, be done as part of phase 1. The necessary patch is nearly complete and can easily be adapted
to, for example, writing a more condensed JSON blob.

### Phase 3

* Refactor Flexforms to freely organize fields in the editing interface (new feature, ability to extract field 
definitions from a DS and render them as part of the “showitems” instruction from TCA)

Phase 3 also could be created right now as a feature request on [forge](https://forge.typo3.org/projects/typo3cms-core/issues).

### Phase 4

* Based on investigation of performance, decide on a different storage strategy for the data that (according to phase 1)
is stored as blobs based on FlexForm fields.
* FlexForm storage driver potentially allows EAV or flat table implementations (actual storage strategy is arbitrary)
