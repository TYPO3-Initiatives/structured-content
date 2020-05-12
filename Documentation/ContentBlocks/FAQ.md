# FAQ to Content Blocks Registration in TYPO3


The purpose of this FAQ is to answer why certain approaches were chosen in the [Content block registration draft](ContentBlockRegistration.md).
And to list issues that still need discussion.

## src/dist vs. structures of a TYPO3 extension

Although src/dist was chosen for the files overview there is no final decision on how the directory structure 
of the content block package should look like yet. See argumentation for both approaches below.


### src/dist
* [Symfony](https://symfony.com/) compliant
    * As TYPO3 uses more and more of Symfony it could be beneficial, especially for people new to TYPO3, to get used to these structures
* Might be reusable for other systems
* More easy to understand by frontend developers as this naming is also common there


### Structures of TYPO3 extension (Configuration, Ressources/Private Ressources/Public)
* Easier to understand for TYPO3 developers
* Access to ‘Private’ folder is denied by .htaccess and TYPO3 suggested [ngnx-configuration](https://docs.typo3.org/m/typo3/reference-coreapi/master/en-us/Security/GuidelinesAdministrators/Index.html#restrict-access-to-files-on-a-server-level)


## Is it possible to provide content block bundles - having multiple content block in one Composer package?

One composer package represents exactly one content block. Bundles can be realized as distributions (e.g. like TYPO3 minimal distribution) 
or within a bundling extension. This decision was made to reduce complexity, anyway if it proves bad in test phase, it will have to be adopted.

## How can I use the same CSS/ JavaScript or library assets across multiple content blocks?

There is no answer to it yet. As this problem does not only refer to this content block approach it won't be handled here.
However, it is #1 issue the Rendering Group of the Structured Content Initiative is tackling with. 
Check out the [github repository](https://github.com/TYPO3-Initiatives/structured-asset-rendering) and be welcome to help.

## How can existing content blocks be overridden?

**Variant a**: Duplicate declaration to new package

**Variant b**: Introduce inheritance (which makes maintenance more complex again)

Things like CSS, JS, HTML, XLF labels can be overridden via site extensions. 
Due to complexity inheritance will be no option for now. This might be discussed later again if the necessity proves unavoidable.

## How can I switch from one CType to another without having to re-enter the content? 

This is not possible with different prefixes per CType. Perhaps a manual prefix could be defined so that, 
for example, an agency can define all its own content blocks on its own responsibility.

For now switching the CType without data loss won't be enabled. However, if the survey for editors proves that it is necessary, 
a strategy for this will be needed first.

## How can content block declarations be validated?

Basically a YAML schema validation (based on JSON schema) is needed here. Exchange with the Form Framework team is targeted.


## How will content blocks become compatible to extensions like indexed_search, ke_search, solr?

Introducing the content blocks package approach will be a breaking change. We offer working together with the extension authors.

## How can the content elements be imported and exported for localization? (Meaning content - not labels)

The default for a field will be localize-able, making this property only necessary if special localization method is required.

## What are the conventions for label keys (frontend/ backend)?

In general the [coding guidelines of the TYPO3 core](https://docs.typo3.org/m/typo3/reference-coreapi/master/en-us/ApiOverview/Internationalization/XliffFormat.html#xliff-id-naming) for labels are to be applied here as well.

For backend labels that would be: `<code-block-identifier>.<field-identifier>`

Although the fields are not actual database fields we recommend to use the snake_case for the field identifier.
For frontend labels developers are free to use the structure they like, although we recommend to comply with the [coding guidelines of the TYPO3 core](https://docs.typo3.org/m/typo3/reference-coreapi/master/en-us/ApiOverview/Internationalization/XliffFormat.html#xliff-id-naming).

## Is it possible to use different language files for frontend and backend?

Yes, in that case please provide `src/Language/EditorInterface.xlf` and `src/Language/Frontend.xlf` instead of `src/Language/Default.xlf`. So, for automated backend label detection the registration process will look for` Default.xlf` first and, if not present, for `EditorInterface.xlf` as well.

## How can fields be grouped to palettes?

Use YAML grouping, e.g.
```yaml
palettes:
   - identifier: palette1
     label: …
     fields:
      …
```

## What happens if a content block definition is not valid?

The content block won’t be available in the TYPO3 backend for editors. An error message is available in the “Check for broken content blocks” tool in the maintenance area.
Additional information to composer could be added via a composer plugin to validate the definition during installation.

## Is it possible to use content blocks without composer?

Yes. This is part of the MVP. However, the first implementation will be the composer approach.

## Is it possible to use templates other than fluid?

Fluid templates will be the default (.html will be interpreted as fluid). However, automated template engine detection via file ending (e.g. .twig) is possible as a future feature.

## Are content blocks stored in TER, are they of type “typo3-cms-extension” and are they put into typo3conf/ext/ ?

No. They will be of type  “typo3-contentblock”. We will need a website like TER to show what content blocks are publically available. The main registry for content blocks is packagist.org, as we are using composer.

## How can we pre-define data processing for a content block having inline/ collection elements or images?

We could automate the registration of data processors for certain field types (as those mentioned above). But this would be a feature.
