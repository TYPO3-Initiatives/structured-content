# Mapping of Symfony types to TCA

TODO


## combined fields with `maxItems` (e.g. `Collection`, `Image`, `Icon`)

If `maxItems: 1` is given or is the default value (e.g. `Image`, `Icon`), the property will be transformed to a single element instead of an array. 