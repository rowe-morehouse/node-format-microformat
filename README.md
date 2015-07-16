# Format Microformat

[![Build Status](https://travis-ci.org/voxpelli/node-format-microformat.svg?branch=master)](https://travis-ci.org/voxpelli/node-format-microformat)
[![Coverage Status](https://coveralls.io/repos/voxpelli/node-format-microformat/badge.svg)](https://coveralls.io/r/voxpelli/node-format-microformat)
[![Dependency Status](https://gemnasium.com/voxpelli/node-format-microformat.svg)](https://gemnasium.com/voxpelli/node-format-microformat)

Formats a Microformat JSON representation into eg. a Jekyll post
## Requirements

Requires io.js or Node 0.12

## Installation

```bash
npm install format-microformat --save
```

## Current status

**Alpha**

Currently formats data into a hardcoded Jekyll style. This is intended to become more dynamic and configurable in the future to adapt to more styles.

## Usage

```javascript
var MicropubFormatter = require('format-microformat');

var formatter = new MicropubFormatter();

formatter.preFormat(micropubDocument)
  .then(function (preFormatted) {
    return Promise.all([
      formatter.formatFilename(preFormatted),
      formatter.format(preFormatted),
      formatter.formatURL(preFormatted),
    ]);
  })
  .then(function (lotsOfResolvedStuff) {
    // Lots of formatted data that one can do lots of fun stuff with. Publish somewhere or such perhaps?
  });
```

## Methods

* **preFormat(micropubDocument)** – takes a `micropubDocument` and ensures that all necessary parts are there. **Currently required** to run a `micropubDocument` through this method before handing it to the rest of the methods.
* **formatFilename(preformattedMicropubDocument)**  – returns a filename based on the data in the `micropubDocument`. Includes the relative path to the file – which currently is always `_posts/`
* **formatURL(preformattedMicropubDocument, [relativeTo])**  – returns the url the formatted content is expected to live on when published. Either a relative URL or an absolute one if `relativeTo` is set
* **format(preformattedMicropubDocument)** – formats the actual content. Currently it's formatted as an HTML-file with [Jekyll Front Matter](http://jekyllrb.com/docs/frontmatter/). The content of the file is the `properties.content` of the `micropubDocument` – the rest of the data is put into the *front matter*.

## Output formats

For exact implementation of how things are formatted, look at the code and the tests, this is just to get an overview of what one can expect the output to be.

### Filename

The target is to get a filename similar to `_posts/2015-06-30-awesomeness-is-awesome.html`. The date first and then either the defined slug or a slug derived from title or content.

### URL

The target is to get a relatuve URL similar to `2015/06/awesomeness-is-awesome/` where the slug is calculated in the same way as it is for the `filename`. In some cases the URL will be prefixed with a category name – that's the case for eg. replies, likes and bookmarks – the first two are prefixed with `interaction/` and the last one with `bookmark/`.

### Content

The actual content of the file is the HTML of the `properties.content` of the `micropubDocument`. All other properties are added to the [Jekyll Front Matter](http://jekyllrb.com/docs/frontmatter/) together with a couple of other defaults.

Some `micropubDocument` properties receive special handling, the rest are included raw as arrays with an `mf-` prefix to avoid collisions with pre-existing Jekull properties.

The ones with special handling are:

* **content** – isn't part of the front matter but is the actual HTML content of the file
* **name** – inserted as `title` and if not specified, then a value of `null` will be used to avoid automatic generation of titles
* **slug** – inserted as `slug`
* **category** – inserted as `tags`
* **published** – inserted as `date` and formatted as ISO-format

There's also some defaults and derived values:

* **layout** – always set to `micropubpost`
* **category** – derived from other `micropubDocument` properties and only used in some cases, like eg. for replies, likes and bookmarks – it's set to  `interaction`  for the first two and to `bookmark` for the last one

An example of a generated Jekyll Front Matter:

```yaml
layout: micropubpost
date: '2015-06-30T14:34:01.000Z'
title: awesomeness is awesome
slug: awesomeness-is-awesome
category: interaction
mf-like-of:
  - 'http://example.com/liked/page'
```

## Format of `micropubDocument`

The format closely matches the [JSON-representation](http://indiewebcamp.com/Micropub#JSON_Syntax) of Micropub.

See the [micropub-express](https://github.com/voxpelli/node-micropub-express#format-of-micropubdocument) module for documentation of this basic object.

In addition to the properties defined by the `micropub-express` module, the `preFormat()` method adds a top level `derived` key that's used internally for values derived from 

## Other useful modules

* [micropub-express](https://github.com/voxpelli/node-micropub-express) – an Express 4 Micropub endpoint that accepts and verifies Micropub requests and calls a callback with a parsed `micropubDocument` that can be used with this module
* [github-publish](https://github.com/voxpelli/node-github-publish) – a module that takes a filename and content and publishes that to a GitHub repository. A useful place to send the formatted data that comes out of this module if one wants to add it to a GitHub hosted Jekyll blog of some kind, like eg. [GitHub Pages](https://pages.github.com/).
