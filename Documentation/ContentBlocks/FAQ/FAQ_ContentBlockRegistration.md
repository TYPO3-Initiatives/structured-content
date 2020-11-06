# FAQ for Content Block registration

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

## What happens if a content block definition is not valid?

The content block won’t be available in the TYPO3 backend for editors. An error message is available in the “Check for broken content blocks” tool in the maintenance area.
Additional information to composer could be added via a composer plugin to validate the definition during installation.

## Is it possible to use content blocks without composer?

Yes. This is part of the MVP. However, the first implementation will be the composer approach.


## Are content blocks stored in TER, are they of type “typo3-cms-extension” and are they put into typo3conf/ext/ ?

No. They will be of type  “typo3-cms-contentblock”. We will need a website like TER to show what content blocks are publically available. The main registry for content blocks is packagist.org, as we are using composer.
  
