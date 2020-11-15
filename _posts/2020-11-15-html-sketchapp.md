---
layout: post
title:  "html-sketchapp: under-the-hood of an HTML to Sketch export solution"
date:   2020-11-15 05:00:00 -0600
---

![screenshot](/img/posts/html-sketchapp/screenshot.gif)
<p class="small text-muted">Taking the output of html-sketchapp and importing into Sketch.</p>

> Designers and developers continue to work in entirely different mediums. As a result, without constant, manual effort to keep them in sync, our code and design assets are constantly drifting further and further apart.

<p class="small text-muted">Mark Dalgleish in <i><a href="https://medium.com/seek-blog/sketching-in-the-browser-33a7b7aa0526">Sketching in the Browser</a></i></p>

Even-though the dawn of [Web 2.0](https://en.wikipedia.org/wiki/Web_2.0) 15 years ago, we still can't auto-translate HTML to our design tools. A huge number of improvements to to the product development flow have made the lives of designers and devs easier, but the common task of going from code to design (and back) remains manual.

One NodeJS package that makes it easier to keep your coded components in sync with your design team is the excellent [html-sketchapp](https://github.com/html-sketchapp/html-sketchapp). Released in 2018 and inspired by the more limited [react-sketchapp](https://github.com/airbnb/react-sketchapp), html-sketchapp lets you export HTML to Sketch (with some [limitations](https://github.com/html-sketchapp/html-sketchapp/wiki/What's-supported%3F)). In this post, I'll show how html-sketchapp works and how it can benefit DesignOps at your organization.


## Is html-sketchapp for designers?

Despite the designer being the end-user of an HTML to Sketch solution, html-sketchapp is really a tool for developers. html-sketchapp provides the export engine and requires another tool to provide the user interface. I think it's unlikely a designer will get much value out of downloading the html-sketchapp source code. If you are designer, I suggest you checkout one of the following for a more ready-to-go solution:

* [html-sketchapp-cli](https://github.com/seek-oss/html-sketchapp-cli) - A tool that allows you to export an HTML doc to Sketch from the command line. Distributed as a NodeJS package.
* [html-to-sketch-electron](https://github.com/KimDal-hyeong/html-to-sketch-electron) - An electron app that allows you to provides a graphical interface versus the command-line tool option provided by html-sketchapp-cli.

![html-to-sketch-electron](/img/posts/html-sketchapp/html-to-sketch-electron.gif)
<p class="small text-muted">html-to-sketch-electron in action.</p>

Note that both of these utilities don't have a lot of recent commits (and I haven't tested them). This post focuses more on how developers can use html-sketchapp within their own internal tools to help push coded components to design tools.

## How does a developer leverage html-sketchapp?

If you are a developer creating a tool that uses html-sketchapp, your flow might look a bit like this:

1. Install the NodeJS package within your project: `npm i @brainly/html-sketchapp`.
2. Use [puppeteer](https://www.npmjs.com/package/puppeteer) to launch a headless browser session opening a URL of your choice.
3. Feed the loaded `document.body` to html-sketchapp's `nodeTreeToSketchPage` function.
4. Save the output file with an `*.asketch.json` extension.
5. Within Sketch, install the [Almost *Sketch* to Sketch Plugin](https://github.com/html-sketchapp/html-sketchapp#import-asketch-files-to-sketch).
6. Use the Almost *Sketch* to Sketch Plugin to import the `*.asketch.json` file and create a new Sketch document of the HTML export.

See [html-sketchapp-example](https://github.com/html-sketchapp/html-sketchapp-example/blob/master/src/inject.js) for a complete example of steps 1-4.

## Why can't html-sketchapp just export a Sketch file?

You may have noticed that in the steps above, we're able to export a JSON file from html-sketchapp but not an actual Sketch file. We take that file and use a Sketch plugin to take the export over the finish line, converting JSON to the final Sketch file format.

Why is there an extra moving part? At the time html-sketchapp was written, some parts of the Sketch file format stored data as a binary blob (like text styling information). This is not easy to generate from Javascript. Additionally, as the html-sketchapp functions run in the browser, it is limited by [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) and may not be able to access all images on an HTML page. While text information is no longer [stored as a binary blob](https://github.com/html-sketchapp/html-sketchapp/issues/99), sadly a Mac is still required in order to use [NSAttributedString](https://github.com/html-sketchapp/html-sketchapp/issues/99#issuecomment-695957754).

So, the almost-sketch file format remains as cocoascript is still required for the final conversion step. This final step is a bit of a pain: most recently, an issue with an Almost *Sketch* to Sketch Plugin dependency [broke exports](https://github.com/html-sketchapp/html-sketchapp/issues/196#issuecomment-696052103). That said, it doesn't appear there's anything html-sketchapp can do about the plugin requirement given the need to call `NSAttributedString`.

## How does html-sketchapp generate the *.asketch.json file?

When `nodeTreeToSketchPage` is called, it creates a Sketch group representation of the node and its child nodes. This group is added to a Sketch page with a width and height set to the same dimensions as the root node.

The meat of the HTML to Sketch translation is in the `nodeToSketchLayers` function. This function is responsible for taking the style properties of an HTML element and mapping those to Sketch styles. The flow works a bit like this:

1. Call [Window.getComputedStyle()](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle) to get an object that contains all of the CSS properties of the HTML element. Remember, html-sketchapp is run within a browser session so we're able to call functions within the Javascript [Web API](https://developer.mozilla.org/en-US/docs/Web/API).
2. Run a series of checks to determine if any layers should be created for this HTML node:
  * Is the HTML node a descendent of a parent [SVGElement](https://developer.mozilla.org/en-US/docs/Web/API/SVGElement)? If so, don't create any layers (more on this later).
  * Is the node is visible? It's actually pretty complex to detect this and the logic for determining visibility is in the `isNodeVisible` function. If the node isn't visible don't create any layers.
3. Create a rectangle [Sketch shape](https://www.sketch.com/docs/shapes/) to represent this HTML node.
4. If the node is an image:
  * Set the shape background color to the HTML node's background color
  * If the node is an HTML `IMG` element, apply an [image fill](https://www.sketch.com/docs/styling/#how-to-add-an-image-fill) using the url of the image. This image will need to be downloaded and can't be loaded from the url dynamically (more later).
5. If the node has a `box-shadow` generate appropriate Sketch inner and outer shadows.
6. If the HTML node has borders, apply these using Sketch inner shadows as Sketch does not support side-specific borders.
7. Apply the opacity.
8. Create a new Sketch Rectangle shape, applying the HTML node border-radius to each corner. Note that only % values are supported.
9. If the HTML node has a background image, applies an image fill similar to the earlier step where the node is an actual HTML `IMG` element. If the background image doesn't fit entirely within the HTML element, create a Sketch rectangle shape and use an image fill to correctly position the background image.
10. If the HTML has a background image that is a linear gradient, apply a Sketch gradient fill.
11. If the node is an `SVG` element, creates a Sketch SVG layer and generates the SVG path string by walking through the child [SVGElement](https://developer.mozilla.org/en-US/docs/Web/API/SVGElement) HTML nodes.
12. If the HTML element text is visible, iterates over text nodes (see [nodeType](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType)) and creates a Sketch layer for each text node.

You can see what HTML properties are supported in the Sketch conversion on the [html-sketchapp wiki](https://github.com/html-sketchapp/html-sketchapp/wiki/What's-supported%3F).

## How does html-sketchapp perform on real-world examples?


The easiest way to try html-sketchapp is to clone [html-sketchapp-example](https://github.com/html-sketchapp/html-sketchapp-example), follow the setup instructions, and run the following:

```
npm run inject YOUR_URL
```

On right I'm comparing a screenshot of npr.org versus the html-sketchapp output. Click for a full-scale version. I'm impressed with the output:

<a href="/img/posts/html-sketchapp/npr_example.png">
  <img src="/img/posts/html-sketchapp/npr_example_small.png"/>
</a>
<p class="small text-muted">
  Comparing the output of npr.org using html-sketchapp.
</p>

 The most significant differences:

* Fonts (I don't have the NPR fonts installed)
* Missing images - there are several of missing images that are replaced with a red rectangle.

## Are there hosted tools that can just do this HTML to Sketch export for me?

In [_Sketching in the Browser_](https://medium.com/seek-blog/sketching-in-the-browser-33a7b7aa0526) from 2018, Mark Dalgleish mentions a number of tools that are trying to bridge the code-to-design gap. At the end of 2021, only one of those tools appears to actually let you export some form of HTML (React components) into your editor: [UXPin](https://www.uxpin.com/merge)*.

## Is there a sweet spot for HTML-Sketchapp?

Rather than converting entire HTML documents into Sketch pages, I think the sweet spot for HTML-Sketchapp is turning coded components into Sketch symbols. This eliminates the need for designers to maintain their own design libraries.

There are two relevant examples of this:

* [html-sketchapp-style-guide](https://github.com/brainly/html-sketchapp-style-guide) - Brainly's tool for converting their styleguide into `*.asketch.json` files.
* [story2sketch](https://github.com/chrisvxd/story2sketch) - Convert [Storybook](https://storybook.js.org/) stories into Sketch symbols.

## Summary

[HTML-Sketchapp](github.com/html-sketchapp/html-sketchapp) is a library that developers can use to help automate the conversion of HTML into the Sketch file format. It's a great way to remove the tedious manual process designers need to apply today of maintaining their own design library. HTML-Sketchapp works great for converting coded components into Sketch symbols.

<p class="small text-muted">
* - disclaimer: <a href="https://www.uxpin.com/studio/blog/meet-uxpin-merge/">I'm working at UXPin to help with Merge</a>.
</p>
