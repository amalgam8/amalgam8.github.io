# Amalgam8 Website

This repository contains the entire source for the [Amalgam8 Website](https://amalgam8.github.io). This is a [jekyll](https://jekyllrb.com) project, which builds the site from these source files.

Content
-------

The website manages collections of blog posts, documentation, and API definitions. Docs and API definitions have their own sidebars.

Documents are stored in the `_docs` directory and are written in markdown with frontmatter.
You can order the list of documents using the `order` attribute in the document's front matter.
The list of documents is sorted ascending by numerical `order`.

You can organize documents into folders (under `_docs`) without impacting the generator.
This does not impact the document list. To group the list of documents by logical category use the
`category` attribute in the document's front matter. The sidebar will be sorted by `category` and then by `order`.

You can also have a `subcategory` attribute as well. This will sort the documents in a given category by `subcategory` and then by `order`.

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

If you do not have (or don't want) `jekyll`, we also provide a `Vagrantfile` and a `Dockerfile` (for [vagrant](https://www.vagrantup.com/) and
[docker](https://www.docker.com/) respectively), that you can use that includes `jekyll` and all the needed dependencies.

### Vagrant

```
vagrant up
vagrant ssh
cd /vagrant
jekyll serve --host 0.0.0.0 --port 4000 --force_polling --incremental
```

From your host machine you can access the website at `http://192.168.33.100:4000`

### Docker

```
docker build -t a8/pages .
docker run -p 4000:4000 -v "$(pwd)":/srv/jekyll a8/pages
```

For users who do not need the `docker-machine`, from your host machine you can access the website at `http://localhost:4000`. For those who do, you
can access the website from `http://<your-docker-machine-ip>:4000`.

You can use the `-d` option on the `docker run`, but it will take a few moments for dependencies to load and for the page to be generated. During this time you will not be able to access the website.
