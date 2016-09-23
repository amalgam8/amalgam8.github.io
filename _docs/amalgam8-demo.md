---
layout: page
title: Amalgam8 Demo
permalink: /docs/amalgam8-demo/
order: 1
---

This guide illustrates how to use Amalgam8 for version and content-based
routing using two demo applications. The guide is divided into two parts:

1. [Setup](/docs/amalgam8-demo/setup/). This section provides step-by-step
   instructions on how to get the Amalgam8 control plane and the CLI
   installed and configured for different container runtimes locally
   ([Docker](https://www.docker.com), [Kubernetes](https://kubernetes.io))
   as well as on PaaS environments like [IBM Bluemix](https://bluemix.net),
   [Google Cloud Platform](https://cloud.google.com), etc.


2. **Example Apps:** Two example applications are provided to illustrate
   various features of Amalgam8.

    1. [Helloworld](/docs/amalgam8-demo/helloworld/). A simple application
       with one microservice. We will use this example app to illustrate
       how Amalgam8 provides _version-based routing with weighted load
       balancing between two different versions of the same microservice_.

    2. [Bookinfo](/docs/amalgam8-demo/bookinfo/). A polyglot application
       with 4 microservices written using Python, Ruby and Java. We 
       will use this app to illustrate several features of Amalgam8 such as
       _content-based routing across different microservice versions,
       internal release for QA testing, systematic resilience testing, and
       gradual release of new version using weighted load balancing_.
