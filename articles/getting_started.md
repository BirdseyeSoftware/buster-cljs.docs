---
title: "Getting Started with buster-cljs"
layout: article
---

## About this guide

This guide explains how to setup buster-cljs in your Clojurescript
project, specificaly:

* How to install and setup buster.js

* How to setup [lein-cljsbuild][lein_cljsbuild] to collaborate with buster.js

* How to keep cross-platform test suites for both Clojure and
  Clojurescript

## Overview

buster-cljs provides a powerful platform to test your Clojurescript
code both in the browser (using many different vendors) and node.

This is done by installing the node package busterjs and then running
a node web server that, when accessed through a browser, you can make
it a slave. By doing this, whenever you want to run your javascript
test from the terminal, you'll use the `buster-test' command to push
the test suite from the node server to the different browser that are
slaves, executing the tests in their environment and then giving you
back the result.

The API for buster-cljs trys to mimic as much as it cans the
clojure.test library, there is a compatibility layer that this library
provides, to make it easy to have test suites both for Clojure and
Clojurescript using the same codebase.

## Supported Clojure/Clojurescript Versions

This library has been tested with Clojure 1.4, and Clojurescript version that
comes bundled with lein-cljsbuild 0.2.9

## Supported buster.js version

This library has been test with busterjs 0.6.3 beta 4

## Adding buster-cljs to your project

With leiningen:

    [com.birdseye/buster-cljs "0.1.0"]

With Maven:

    <dependency>
      <groupId>com.birdseye</groupId>
      <artifactId>buster-cljs</artifactId>
      <version>0.1.0</version>
    </dependency>

## Installing buster.js

In order to start, you'll need to install nodejs and npm, please
follow instructions to install [here][node_install].

Later, you will need to install buster package using npm

    $> npm install -g buster

Once the library is installed, you have to run the `buster-server` command
and slave some browsers

    $> buster-server

## Setting up your cljsbuild and buster.js

In order for your project to work as intended with busterjs, you'll
need to create 3 different builds using the [lein-cljsbuild][lein_cljsbuild] plugin.

Let's assume there are 2 folders in your project, `src/cljs` and
`test/cljs`, the first one contains the actual code, and the second
one contains the test code. You will need to have one build just for
the development code, and multiple ones for the test code, example
follows:

    (defproject my-cljs-project
      :dependencies [[buster-cljs "0.1.0"]]
      :cljsbuild
      {:builds
      [{:id "dev"
        :source-path "src/cljs"
        :compiler
        {:optimizations :simple
         :pretty-print true
         :externs ["externs/buster.js"]
         :output-to "my_cljs_project_dev.js"}}
       {:id "browser-test"
        :source-path "test/cljs"
        :compiler
        {:optimizations :simple
         :pretty-print true
         :externs ["resouces/externs/buster.js"]
         :libraries ["resources/js/my_cljs_project_dev.js"]
         :output-to "resources/js/my_cljs_project_browser_test.js"}}
      {:id "node-test"
        :source-path "test/cljs"
        :compiler
        {:optimizations :simple
         :pretty-print true
         :target :nodejs
         :externs ["resouces/externs/buster.js"]
         :libraries ["resources/js/my_cljs_project_dev.js"]
         :output-to "resources/js/my_cljs_project_node_test.js"}}]})


Notice that we use the `dev` build as a library source in both
`browser-test` and `node-test`, this is to keep your library/app code
separated from the test code. Also is important to mention that the
`dev` build must be compiled always with a __simple__ optimization,
otherwise the test won't be able to compile correctly.

Once your `project.clj` has this builds, you have to create in the
root of your project a buster.js file that specifies the js where the
test suites are.

    var config = exports;

    config["Browser Tests"] = {
        environment: "browser",
        sources: [],
        tests: ["resources/js/my_cljs_project_browser_test.js"]
    };

    config["Server Tests"] = {
        environment: "node",
        sources: [],
        tests: ["resources/js/my_cljs_project_node_test.js"]
    };

__important__: You may be able specify an advanced compilation mode
with buster-cljs, although this code will only work for browser js
development, Clojurescript doesn't support advanced compilation and
node just yet.

## Running your tests in buster.js

Now you are ready to run the tests, once you have some slave browsers
connected to the `buster-server`, you can run:

    $> buster-test -e browser

If you want to execute the tests for node:

    $> buster-test -e node

You may also be able to see the test suite as an static page (qUnit style)
by running a command on the root directory of your project

    $> buster-static

## Crossplatform testing with lein-dalap

buster-cljs works both for Clojure and Clojurescript, by using
[lein-dalap][lein_dalap], you can easily make your code work for both
of them without to much of a hassle.

Lets say there is a test in our project like the following

    ^{:cljs
      '(ns my-cljs-project.test.util-test
         (:require [my-cljs-project.util :as util])
         (:require-macros [buster-cljs.macros :refer [initialize-buster deftest describe it is])))
    (ns my-cljs-project.test.util
      (:require [my-cljs-project.util :as util]
                [buster-cljs.clojure :refer [deftest describe it is]]))

     ;; This line bellow is only necessary if you are going to be testing
     ;; your library against node, and it will only be there in Clojurescript
     #_(:cljs (initialize-buster))

     ;; `describe` and `it` will be equivalent to `testing` on clojure side
     (deftest shout-function
       (it "always caps"
         (is (= (util/shout "hello") "HELLO") "shout should make hello all caps")))



__important__: In order to see your test results via buster, __you
need__ to wrap your assertions inside an `it` clause.

## Wrapping Up

Congratulations, you now know how to do most of the setup and
implementation of simple tests using buster-cljs, by now you should
know:

* Install buster-cljs
* Install buster with node and npm
* Setup your Clojurescript project with [lein-cljsbuild][lein_cljsbuild] and buster.js
* Run your tests both in the browser and in node
* Implement simple test suites

## What to read next

Read how to manage easily cross-platforms Clojure/Clojurescript code
using [lein-dalap][lein_dalap], and more about what [buster.js may offer][busterjs]

[node_install]:http://joyent.com/blog/installing-node-and-npm
[lein_cljsbuild]:https://github.com/emezeske/lein-cljsbuild
[lein_dalap]:https://github.com/BirdseyeSoftware/lein-dalap
[busterjs]:http://docs.busterjs.org/en/latest/