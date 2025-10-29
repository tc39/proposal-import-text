# Import Text

Champion: [Eemeli Aro](https://github.com/eemeli)  
Stage: 0

In a similar manner to why importing
[JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import/with#importing_json_modules_with_the_type_attribute)
or [raw bytes](https://github.com/tc39/proposal-import-bytes) is useful in JavaScript,
importing text is useful, and should be just as easy.

This benefits users, as a relatively common operation can simultaneous be made simpler and more performant,
leading to a better experience of the web.

As an example use case,
a developer may want to import a YAML file and parse it with a [user library](https://www.npmjs.com/package/yaml).

## Methodology for approaching & evaluating solutions

As mentioned in the context of [site-building ergonomics](https://www.mozilla.org/en-US/about/webvision/full/#site-buildingergonomics),
"The most powerful way to make something easier is to make it simpler,
so we aim to reduce the total complexity that authors need to grapple with to produce their desired result."
Therefore, we should seek to advance a solution that simplifies the developer experience.

Furthermore, as stated under [Extending the Web with JavaScript](https://www.mozilla.org/en-US/about/webvision/full/#extendingthewebwithjavascript),
"we think browsers should learn from what is working, listen to what is needed, and provide the right primitives.
As essential features and abstractions emerge in the ecosystem,
we can standardize and integrate them into the Web Platform directly to make things simpler."
Where possible, the solutions we propose for standardization should not be novel,
but ones that have already proven to work well,
and which integrate well with prior solutions.

## Prior/existing features and proposals

With the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API),
it's possible to load a text file in JavaScript with

```js
let response = await fetch("path/to/file.txt");
let text = await response.text();
```

With import attributes,
we already support importing JSON modules
([spec](https://tc39.es/ecma262/multipage/ecmascript-language-scripts-and-modules.html#sec-HostLoadImportedModule),
[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import/with#importing_json_modules_with_the_type_attribute)):

```js
import json from "path/to/file.json" with { type: "json" };
```

A separate [Import Bytes](https://github.com/tc39/proposal-import-bytes) TC39 proposal
has rather quickly advanced to stage 2.7, and is likely to advance further quite soon.
With it, it becomes possible to also import text as:

```js
import uint8array from "path/to/file.txt" with { type: "bytes" };
let text = new TextDecoder().decode(uint8array);
```

Some serverside JavaScript implementations,
including at least [Deno](https://docs.deno.com/examples/importing_text/) and [Bun](https://bun.com/guides/runtime/import-html),
also support `type: "text"` as an import attribute:

```js
import text from "path/to/file.txt" with { type: "text" };
```

## Flaws or limitations in existing features/proposals

While the existing Fetch API works, it has three distinct limitations:

1. The operations are always async.
2. Relative paths are rooted at the document's base URL
   rather than that of the module from which it's being fetched.
3. The fetch starts only when the the JavaScript is being executed.

If the Import Bytes proposal proceeds while this proposal does not,
importing text files will require an otherwise unnecessary and clumsy step.

## Motivation for this proposal

Loading text files in JavaScript should be easy,
and can be done without adding new complexities to the language.
Existing functionality (loading JSON files)
already effectively internally requires the functionality being proposed here.

## Outline of a proposed solution

Add a new `'text'` value for the `type` import attribute,
with which a file may be imported as a string value:

```js
import text from "path/to/file.txt" with { type: "text" };
```

No affordance for defining the encoding ought to be given,
with all files parsed as UTF-8.

## Shortcomings

Importing non-UTF-8 text will require importing it as an Uint8Array,
and explicitly decoding it:

```js
import uint8array from "path/to/file.txt" with { type: "bytes" };
let text = new TextDecoder("utf-16le").decode(uint8array);
```
