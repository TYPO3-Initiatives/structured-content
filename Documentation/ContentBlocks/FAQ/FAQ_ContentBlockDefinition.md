# FAQ for Content Block definition  

## Why do we use src/dist instead of structures of a TYPO3 extension?

Although the classic structure of a TYPO3 extension is easier to understand for TYPO3 developers,
we chose to use the [Symfony](https://symfony.com/) compliant src/dist structure instead.
As TYPO3 uses more and more of Symfony it could be beneficial, especially for people new to TYPO3, to get used to these structures.
Also, src/dist is more easy to understand by frontend developers as this naming is also common there.

## How can content block declarations be validated?

Basically a YAML schema validation (based on JSON schema) is needed here. Exchange with the Form Framework team is targeted.

## Why are the field types for the EditingInterface.yaml based on Symfony? Why not just use TCA?

In our opinion it is important to separate a field's view related properties from it's database related properties, 
unlike it is done in TCA. Having a strict schema for field types also eases up the validation process for field definitions 
which is currently not really possible with TCA. Using a field type mapping also allows to use defaults for field properties (e.g. default size for input is 30). 
That way not every property needs to be defined in the EditingInterface.yaml making it slim and easier to overview.

Bacause Symfony is quite mainstream, well-established and documented it makes it easier to understand those types for 
TYPO3 newcomers/ beginners/ frontend-only devs than TYPO3's exclusive TCA, thus providing a kind of ubiquitous language.

Last but not least: With Symfony based field types the content blocks could be integrated into a different CMS or database or file based system.

Notes on TCA:
Currently there is a long term goal to refactor TCA, but it is unknown when this will happen. 
With Symfony based field types we wouldn’t have a breaking change in the configuration, but only “under the hood” then.


## Why is the editing interface for Content Blocks defined in YAML? Why not use PHP like in TCA?

As mentioned above the editing interface configuration only contains view related properties of the fields. It should be
slim and easy to read. Therefore, descriptive language is sufficient.
Also, using PHP would open up a possible security flaw.

## What are the conventions for label keys (frontend/ backend)?

In general the [coding guidelines of the TYPO3 core](https://docs.typo3.org/m/typo3/reference-coreapi/master/en-us/ApiOverview/Internationalization/XliffFormat.html#xliff-id-naming) 
for labels are to be applied here as well.

For backend labels that would be: `<code-block-identifier>.<field-identifier>`

Although the fields are not actual database fields we recommend to use the snake_case for the field identifier.
For frontend labels developers are free to use the structure they like, although we recommend to comply 
with the [coding guidelines of the TYPO3 core](https://docs.typo3.org/m/typo3/reference-coreapi/master/en-us/ApiOverview/Internationalization/XliffFormat.html#xliff-id-naming).

## Is it possible to use different language files for frontend and backend?

Yes, in that case please provide `src/Language/EditorInterface.xlf` and `src/Language/Frontend.xlf` instead of `src/Language/Default.xlf`. 
So, for automated backend label detection the registration process will look for` Default.xlf` first and, if not present, for `EditorInterface.xlf` as well.

## How can fields be grouped to palettes?

Use YAML grouping, e.g.
```yaml
palettes:
   - identifier: palette1
     label: …
     fields:
      …
```

## Is it possible to use templates other than fluid?

Fluid templates will be the default (.html will be interpreted as fluid). However, automated template engine detection via file ending (e.g. ".twig") is possible as a future feature.

## How can we pre-define data processing for a content block having inline/ collection elements or images?

We could automate the registration of data processors for certain field types (as those mentioned above). But this would be a feature.
