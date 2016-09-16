# Amalgam8 Website

This repository contains the entire source for the [Amalgam8 Website](https://amalgam8.github.io). This is a [jekyll](https://jekyll.io) project, which builds the site from these source files.

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
jekyll serve --host 0.0.0.0 --port 4000
```

From your host machine you can access the website at http://192.168.33.100:4000
