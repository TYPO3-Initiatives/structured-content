# Fluid

You can use these variables in Fluid templates. They are defined in `EditorPreview.html` as well as in `Frontend.html`.

## Variables from fields defined in `EditorInterface.yaml`

These fields are on the top level. 

### Example

#### Simple fields

This field definition in `EditorInterface.yaml`

    fields:
      - identifier: headline
        type: Text

would let you access the variable in your Fluid templates like this:

    <html xmlns:f="http://typo3.org/ns/TYPO3/CMS/Fluid/ViewHelpers" data-namespace-typo3-fluid="true">
    …
    {headline}
    …
    </html>

#### Combined fields

This field definition in `EditorInterface.yaml`

    fields:
      - identifier: slides
        type: Collection
        properties:
          fields:
            - identifier: text
              type: Text

would give you the `slides` variable as an array:

    <html xmlns:f="http://typo3.org/ns/TYPO3/CMS/Fluid/ViewHelpers" data-namespace-typo3-fluid="true">
    …
    <f:for each="{slides}" as="slide">
        {slide.text}
    </f:for>
    …
    </html>

If an `Image` and `Icon` field has `maxItems: 1` (default) it is not delivered as array bit as a single value.

## ContentBlock configuration

// TODO: this is not API yet and the available properties are not decided upon yet

The Fluid variable `{cb}` holds information about the current content block.

This is currently:
* `{cb.key}` - The ContentBlock key works like the extension key of a TYPO3 extension. It is the directory name (e.g. ‹call-to-action›) from where the ContentBlock was loaded. It can be used as a unique key (e.g. for building CSS classes) or for loading assets from the ContentBlocks's directory.

## Values from the `tt_content` record

The current `tt_content` record is available as `{data}` in Fluid.
