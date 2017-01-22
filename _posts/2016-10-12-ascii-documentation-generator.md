---
layout: post
title: Documenting Your Knowledge in Large Projects
---

Everyone of us knows the pain of bad or missing documentation. It's a critical element for onboarding, knowledge transfer and knowledge conversation within a software project. If you're in a five person team, tasks are done asynchronously and there are quite a few repositories, modules, services or however you call it, there is *no way without documentation*. You must fill the gap of missing knowledge of your colleagues to craft good software.

To solve this problem we require three things:

- Our common and distributed knowledge about our modules, domain and tech
- Commitment to our Definition of Done: We are only "done" when things are documented
- Good structure and tooling for the resulting documentation

The first two requirements are given if you are willing to do this. For the last one, I started to create and develop a tool: [moddoc](https://github.com/jverhoelen/moddoc "Moddoc - Simple documentation PDF builder from AsciiDoc"), standing for modular documentation.

## Approach

Moddoc makes some assumptions:

- You work for some project and organize everything in the folder `company`
- The project consists of multiple modules (or whatever) belonging together in subfolders, e.g. (`company/module`)
- Your modules have a doc-directory (`company/module/doc`) containing an `index.adoc` file as entry point
- There is module-specific knowledge (e.g. knowledge about a REST-service)
- There is common knowledge which relates to more than one module

If you fulfill them, it rewards you with the ability to build a single PDF file containing everything in a well structured way.

## Implementation and Usage

To try it out, download or fork [moddoc](https://github.com/jverhoelen/moddoc "Moddoc - Simple documentation PDF builder from AsciiDoc") as another module of your project hierarchy. It will contain your common documentation (`common/`), configuration (`config.json` and `common/modules.adoc`) and brings you the needed build tool.

To generate the resulting documentation, install the dependencies with `npm install`, make sure to have the [asciidoctor-pdf ruby gem](http://asciidoctor.org/docs/convert-asciidoc-to-pdf/#install-the-published-gem "Asciidoctor-pdf ruby gem") installed and hit `npm run build` to build it.

There you go. Feel free to comment your opinion and ideas or insert issues!
