# Amalgam8 Website

This repository contains the entire source for the [Amalgam8 Website](https://amalgam8.github.io). This is a [jekyll](https://jekyll.io) project, which builds the site from these source files.

Content
-------

The website manages collections of blog posts, documentation, and API definitions. Docs and API definitions have their own sidebars.

Documents are stored in the `_docs` directory and are written in markdown with frontmatter.
You can order the list of documents using the `order` attribute in the document's front matter. 
The list of documents is sorted ascending by numerical `order`.

You can organize documents into folders (under `_docs`) without impacting the generator. 
This does not impact the document list. To group the list of documents by logical category use the 
`category` attribute in the document's front matter. The sidebar will be sorted by `category` and then by `order`.

Swagger API documents are stored in the `_api` directory and must reference the a URL for the actual swagger
JSON file in the frontmatter parameter `spec`. For example:

```
---
layout: swaggerui
title: Route Controller
description: The controller provides APIs to the developer for configuring rules for request routing, fault injection, etc.
spec: https://raw.githubusercontent.com/amalgam8/amalgam8/master/api/swagger-spec/controller.json
---
```

You can use the [Swagger UI](http://editor.swagger.io/#/) to convert Swagger JSON documents to YAML.

Contributions Welcome!
----------------------

If you find a typo or you feel like you can improve the HTML, CSS, or JavaScript, we welcome contributions. Feel free to open issues or pull requests like any normal GitHub project, and we'll merge it in.

Running the Site Locally
------------------------

Running the site locally is simple. If you have `jekyll` installed with the `github-pages` plugin you can clone this repo 
and run `jekyll serve`. Your website will be visible at `http://localhost:4000`.

If you do not have (or don't want) `jekyll` installed there is a `Vagrantfile` you can use to run a VM with `jekyll` and all the needed dependencies.

```
vagrant up
vagrant ssh
cd /vagrant
jekyll serve --host 0.0.0.0 --port 4000 --force_polling --incremental
```

From your host machine you can access the website at http://192.168.33.100:4000
