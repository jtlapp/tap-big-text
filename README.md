# crumpler

assertions that reduce text documents to just their differences

## Overview

`crumpler` reduces lengthy text documents to concise representations of their differences, particularly for use in test assertions that compare documents.

Differencing tools are typically able to collapse lengths of identical text to abbreviated forms in order to make differences apparent. Unlike differencing tools, test harnesses cannot be expected to provide sophisticated representations for differing test results. This becomes an issue when the test results are lengthy text documents, as for example when a test compares two books that differ only by one word in the middle -- the user is in trouble when the test displays both books so the user can find their differences. `crumpler` reduces text documents to their differences *prior to* handing them to the test harness, providing the harness with manageable output.

`crumpler` provides a number of facilities for reducing text documents to just their differences. The details of the reduction are configurable. `crumpler` can collapse series of lines into a few lines bracketing an ellipsis, it can truncate lines at a maximum length, it can crop lines at their heads and tails to emphasize a difference found between them, and it can number lines. It does all this in a way that allows the test harness to properly determine equality and that allows a downstream renderer to properly highlight test result differences. When including line numbers, it may be important to flag the presence of line numbers for the downstream difference renderer.

The `crumpler` library includes `textEqual()` and `textNotEqual()` test assertions for [`tap`](https://github.com/tapjs/node-tap), as well as general-purpose functions for use in other test harnesses or applications.

## Installation

```
npm install crumpler --save
```

## Usage

There are two ways to use this library: you can use it for its [`tap`](https://github.com/tapjs/node-tap) test assertions, and you can use the functions it exposes for shortening documents. Unless you're happy with the default configuration, both uses require first configuring an instance of the `Crumpler` class. This instance controls exactly how text gets shortened.

Load the `Crumpler` class and create a configured instance:

```js
var Crumpler = require('crumpler');

var crumpler = new Crumpler({
    normBracketSize: 1, // show only first and last lines of a common series
    diffBracketSize: 0, // don't collapse series of lines that differ
    maxNormLineLength: 180, // truncate common lines at 180 chars
    maxLineDiffLength: 180, // truncate line differences at 180 chars
    sameHeadLengthLimit: 85, // show 85 chars of line preceding difference
    sameTailLengthLimit: 65, // show 65 chars of line following difference
    lineNumberPadding: '0' // left-pad line numbers with zeros
});
```

The following methods are available. Click the links for more detailed information:

Method | Description
--- | ---
Crumpler.addAsserts(tap) | Adds test assertion methods to an instance of tap.
crumpler.shortenDiff(subject, model) | Reduces both subject and model text to their differences in accordance with the configuration. Returns an object {subject, model, lineNumberDelim} containing the shortened text.
crumpler.shortenText(text, maxLineLength) | Abbreviates the provided text according to the configuration, truncating lines at `maxLineLength`. Returns the shortened text.

The `Crumpler` class is stateless, so the methods of an instance can be used repeatedly or concurrently without concern for interference.

`Crumpler.addAsserts()` adds the following assertion methods to [`tap`](https://github.com/tapjs/node-tap). `t' here is a `tap` test:

Method | Description
--- | ---
t.textEqual(f,w,c,m,e) | Compares found text `f` with wanted text `w` using crumpler `c`, requiring that they be identical to pass. Message `m` and extra values object `e` are optional.
t.textEquals(f,w,c,m,e) | Same as `t.textEqual(f,w,c,m,e)`.
t.textInequal(f,w,c,m,e) | Compares found text `f` with wanted text `w` using crumpler `c`, requiring that they be different to pass. Message `m` and extra values object `e` are optional.
t.textNotEqual(f,w,c,m,e) | Same as `t.textInequal(f,w,c,m,e)`.

These assertions are analogous to `tap`'s `t.equal()` and `t.notEqual()` assertions, as explained at [node-tap.org](http://www.node-tap.org/asserts/), except that they also take a `Crumpler` instance for configuration.

## Example

TBD

## Terminology

The subsequent reference material for `crumpler` employs the following terms:

Term | Definition
--- | ---
collapse | Reduction of a series of lines into a a few lines that start the series, a few lines that end the series, and an ellipsis in between.
bracket | The lines that start or end a collapse.
crop | Removal of characters from either the head (start) of a line or the tail (end) of a line.
ellipsis | A sequence of characters that replaces either the lines removed from a collapse or the characters removed from a crop.
subject | Text that is being abbreviated, possibly in comparison to a model for purposes of retaining differences with the model in the abbreviation.
model | Text that is being abbreviated in comparison to a subject for purposes of retaining differences with the subject in the abbreviation.

## Configuration

The Crumpler constructor takes an object of configuration options. The following options govern collapsing sequences of lines:

Collapse Option | Description
--- | ---
normBracketSize | Number of lines to show on each side of a collapsed sequence of lines that does not differ from a comparison text. This is the number of lines in a bracket of text that is common to a comparison text or that is not being compared to another text. Set to 0 to turn off collapsing in these cases. (default 2)
diffBracketSize | Number of lines to show on each side of a collapsed sequence of lines that differs from a comparison text. Set to 0 to disable the collapsing of text that differs from the comparison text. (default 0)
minCollapsedLines | Minimum number of lines that can be removed from a sequence of lines to collapse it. In order for text to collapse it must have at least this number of lines plus twice the applicable bracket size. Set to 0 to disable all collapsing of text, regardless of the values of normBracketSize and diffBracketSize. (default 2)

The following options govern line cropping:

Cropping Option | Description
--- | ---
maxNormLineLength | The maximum number of characters to show from each line of a sequence of lines that does not differ from a comparison text. This the maximum length of a line that is common to a comparison text or that is not being compared to another text. Set to 0 to show the entire line in these cases. Methods that have a maxLineLength parameter employ maxLineLength in place of the configured maxNormLineLength. (default 0)
maxLineDiffLength | The maximum number of differing characters to show from each line of a series of lines that differs from a comparison text. The first line of a series of differing lines may also show non-differing characters before and after the differing characters, according to sameHeadLengthLimit and sameTailLengthLimit, but second and subsequent lines of the series crop at maxLineDiffLength characters. Set to 0 to show all differing characters. (default 0)
sameHeadLengthLimit | The limited number of characters of a line to show before the first character that differs from a comparison text. These characters are common to both subject and model texts at the start of the first line of a series of one or more lines that differ between the texts. They provide preceding context for the differing characters. Characters of the line prior to this context are cropped and replaced with headCropEllipsis, but only if the crop would exceed the length of the replacement. Set to 0 to never show characters that precede line differences. Set to -1 to show all preceding characters. (default -1)
sameTailLengthLimit | The limited number of characters of a line to show after the last character that differs from a comparison text. These characters are common to both subject and model texts at the end of the first line of a series of one or more lines that differ between the texts. They provide trailing context for the first-line characters that differ. Characters of the line after this context are cropped and replaced with tailCropEllipsis, but only if the crop would exceed the length of the replacement. When maxLineDiffLength truncates the number of differing characters shown, no trailing context characters are shown. Set to 0 to never show the characters that follow line differences. Set to -1 to show all following characters. (default -1)

The following options provide replacement text for text that is removed by collapsing or cropping. Their values may optionally contain `{n}`. Within the collapse ellipses, `{n}` is a placeholder for the number of lines removed. Within the crop ellipses, `{n}` is a placeholder for the number of characters removed. By making minCollapsedLines >= 2, the collapse ellipses language can assume a plurality. Because crops never replace fewer characters than their lengths, the crop ellipses language can assume a plurality by ensuring that headCropEllipsis and tailCropEllipsis always contain at least two characters.

Ellipsis Option | Description
--- | ---
- normCollapseEllipsis | One or more lines that replace lines removed in the collapse of text that does not differ from a comparison text. This is the ellipsis for collapsed text that is common to a comparison text or that is not being compared to another text. (default `" ..."`)
subjectCollapseEllipsis | One or more lines that replace lines removed in the collapse of subject text not found in the model text. (default `"   ..."`)
modelCollapseEllipsis | One or more lines that replace lines removed in the collapse of model text not found in the subject text. (default `"  ..."`)
headCropEllipsis | String that replaces characters cropped from the head (start) of a line. (default `"[{n} chars...]"`)
tailCropEllipsis | String that replaces characters cropped from the tail (end) of a line. (default `"[...{n} chars]"`)
indentCollapseEllipses | When true and lines are being numbered, each line of a collapse ellipsis is indented by a number of spaces equal to the offset endured by the immediately prior line due to line numbering. The immediately prior line is the last line of the preceding bracket. The indentation includes the padded width of the line number and the length of lineNumberDelim. (default false)

Any of these ellipses may be blank, but only collapse ellipses may contain LFs (`\n`). A blank collapse ellipsis produces a blank line, while a blank crop ellipsis abruptly truncates the line. A trailing LF of a collapse ellipsis produces a trailing blank line.
  
The collapse ellipses need not all be different, but they should all be different. Making them distinct from one another helps downstream tools that diff the collapsed strings to properly recognize differences. These tools may otherwise assume that collapse ellipses all represent identical lines.

The following options govern line numbering. Line numbering is useful both for mapping lines in abbreviated text to the original text and for readily observing the lengths of collapsed text.

Numbering Option | Description
--- | ---
minNumberedLines | Minimum number of lines that a text must have in order for line numbers to be added to the abbreviated text. 0 disables line numbering. When comparing subject and model texts, the lines of both abbreviated texts are numbered if either text exceeds this minimum line count. (default 2)
lineNumberPadding | The character to use for left-padding line numbers to make them all occupy the same character width. When comparing subject and model texts, line numbers are padded to the greater of their width requirements. Set to null or `''` to disable left-padding. (default null)
lineNumberDelim | Delimeter that follows each line number. May be null or `''` to insert no delimeter. (default `":"`)

## API Reference

* [Crumpler](#Crumpler)

    * [new Crumpler()](#new_Crumpler_new)

    * _instance_
        * [.shortenDiff(subject, model)](#Crumpler+shortenDiff)

        * [.shortenText(text, maxLineLength)](#Crumpler+shortenText)

    * _static_
        * [.addAsserts(The)](#Crumpler.addAsserts)


<a name="new_Crumpler_new"></a>

### new Crumpler()
This reference assumes the module is loaded in the variable `Crumpler`.

<a name="Crumpler+shortenDiff"></a>

### *crumpler*.shortenDiff(subject, model)

| Param | Description |
| --- | --- |
| subject | The subject text string. |
| model | The model text string. |

Shorten the subject and model text strings to minimal representations that clearly show differences between them. Returns an abbreviated subject string and an abbreviated model string that themselves can be compared using a diffing tool to properly highlight their differences. The method reduces both the number of lines and the lengths of individual lines, according to the configuration. It also numbers the lines in accordance with the configuration.

If line numbers are being added to the shortened text, and if the line numbers of the subject and model values disagree on any lines, a diffing tool that subsequently compares the values will have to be smart enough to ignore the line numbers, unless the line numbers are removed prior to diffing. This method also returns the line number delimiter it used, if any, for use by the downstream diffing tool to properly handle line numbers.

**Returns**: An object having properties `subject`, `model`, and `lineNumberDelim`. The first two properties are the abbreviated subject and model texts, shortened to optimize comparing their differences. `lineNumberDelim` is either null to indicate that subject and model lines were not numbered or a string providing the delimiter used between each line number and the rest of the line. Collapsed ellipsis lines are not numbered. Subject or model text that is an empty string has no lines and no line numbers.  
<a name="Crumpler+shortenText"></a>

### *crumpler*.shortenText(text, maxLineLength)

| Param | Description |
| --- | --- |
| text | String of one or more lines of text to shorten. |
| maxLineLength | Maximum characters of a line to include. 0 allows lines of unlimited length. Defaults to the maxNormLineLength option. |

Shorten the provided text in accordance with the configuration. Because the text is not being compared with another text, it shortens as if it were a section of text common to two compared texts.

**Returns**: a String of the text shortened as specified  
<a name="Crumpler.addAsserts"></a>

### *Crumpler*.addAsserts(The)

| Param | Description |
| --- | --- |
| The | instance of the tap module to which to add the assertions. |

Adds test assertion methods to an instance of tap. These assertions call shortenDiff() on their found and wanted values using a provided instance of Crumpler. Each of these assertion methods takes parameters in the form textEqual(found, wanted, crumpler, description, extra). Only the first two parameters are required. The default crumpler is `new Crumpler()`.

When lines are being numbered, these assertion methods attache a `{lineNumbers: true}` option to the tap extra field, which allows tools that process TAP downstream to treat numbered text differently. Subtap does this to ignore line numbers when comparing subject and model text.


## LICENSE

This software is released under the MIT license:

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Copyright © 2016 Joseph T. Lapp


