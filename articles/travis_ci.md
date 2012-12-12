---
title: "buster-cljs | Setup Travis CI"
layout: article
---

## About this guide

This guide explains how to setup your project to automatically run
your clojure and javascript tests via [Travis CI][travis_ci]. The guide will cover:

* Setup a package.json file for managing busterjs dependency

* Setup a .travis.yml file for running all different tests

In this guide we asume you are using [github][github] and want to enable [Travis CI][travis_ci]
continious testing.

## Automate installation of busterjs dependency

In order to run busterjs tests on Travis CI, you will require a `package.json` file in the
root of your project. This file is used by nodejs' npm to track the package meta-data and
dependencies. Follow the sample given next:

    // This file is only here for busterjs dependency management in Travis CI
    {
        "name": "<your-project-name>",
        "description": "<your-project-description>",
        "version": "<your-project-version>",
        "author": "<your-project-author>",
        "dependencies" : { "buster" : "{{site.busterjs_version}}" },
        "scripts": { "test": "buster-test -e node && buster-test -e browser" }
    }

With this files you will be able to install buster using `npm
install`, and to run your testsuite using `npm test`.

__Important:__ you need to follow the instructions on how to [setup
buster.js configuration
file]({{site.baseurl}}/articles/getting_started.html#setup_cljsbuild_and_busterjs),
otherwise the test suite won't be executed correctly with busterjs.

In case you don't want your Clojurescript library to support nodejs,
you may remove the `buster-test -e node` bit on the code above.


### Add a .travis.yml file to your project

Next step is to add a `.travis.yml` file to your project, follow the
sample given next:

    language:
      - node_js
      - clojure
    lein: lein2
    jdk:
      - oraclejdk7
    node_js:
      - "0.8"
    before_script:
      - "export DISPLAY=:99:0"
      - "echo 'Starting buster server'"
      - "./node_modules/buster/bin/buster-server &"
      - "sleep 3"
      - "echo 'Done.'"
      - "echo 'Starting phantomjs server'"
      - "phantomjs ./node_modules/buster/script/phantom.js &"
      - "sleep 3"
      - "echo 'Done.'"
    script:
      - lein2 test && npm test

This script will run the `buster-server` and start a phantomjs
headless browser that will slave automatically to it, then it will
execute both Clojure JVM testsuite (using `lein2 test`) and Clojurescript
testsuite (using `npm test`).

### Setup your github project to run Travis CI

Please follow instructions
[here](http://about.travis-ci.org/docs/user/getting-started/). You may
skip the setup of the `.travis.yml` file specified there, given we
covered that in this guide.

### Wrapping Up

Congratulations! You've learned how to setup your project to execute
tests automatically after your push to [Github][github].

[travis_ci]:http://travis-ci.org/docs/
[github]:http://github.com/