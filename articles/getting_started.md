---
title: "buster-cljs | Getting Started"
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

Buster.js is a node.js package that a) provides a test case /
assertions library and b) runs a node web server which pushes tests
out to any browsers you slave to it. When you want to run your
javascript tests, you issue the `buster-test' command in the terminal
to push tests to slaved browsers, in parallel, and collect the
results.

The surface-level API for buster-cljs is clojure.test compatible.

## Supported Clojure/Clojurescript Versions

This library has been tested with Clojure 1.4, and Clojurescript
version that comes bundled with lein-cljsbuild 0.2.9

## Supported buster.js version

This library has been tested with busterjs 0.6.3 beta 4.

## Adding buster-cljs to your project

With leiningen:

    [com.birdseye-sw/buster-cljs "{{site.package_version}}"]

With Maven:

    <dependency>
      <groupId>com.birdseye-sw</groupId>
      <artifactId>buster-cljs</artifactId>
      <version>{{site.package_version}}</version>
    </dependency>

## Installing buster.js

In order to start, you'll need to install node.js and npm. 
Follow the instructions [here][node_install].

Next, install the buster package using npm

    $> npm install -g buster

Once the library is installed, create a `buster.js` config file in your project root directory and run the `buster-server` command.  

    $> buster-server

Buster is now waiting for you to slave some browsers to the server you just started.  Follow the link it gives you.

<a href="#" id="setup_cljsbuild_and_busterjs"></a>
## Setting up your cljsbuild and buster.js

In order for your project to work as intended with busterjs, you'll
need to create 3 different builds using the [lein-cljsbuild][lein_cljsbuild] plugin.

Let's assume there are 2 folders in your project, `src/cljs` and
`test/cljs`.  The first one contains the actual code and the second
one contains the test code. You will need to have one build just for
the development code and multiple ones for the test code. For example:

    (defproject my-cljs-project
      :dependencies [[com.birdseye-sw/buster-cljs {{config.package_version}}]]
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
       {:id "browser-advanced-test"
        :source-path "test/cljs"
        :compiler
        {:optimizations :advanced
         :pretty-print false
         :externs ["resouces/externs/buster.js"]
         :libraries ["resources/js/my_cljs_project_dev.js"]
         :output-to "resources/js/my_cljs_project_browser_optimized_test.js"}}
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
`browser-test` and `node-test`.  This is to keep your library/app code
separated from the test code. Also is important to mention that the
`dev` build must be compiled always with a __simple__ optimization,
otherwise the test won't be able to compile correctly.

Once your `project.clj` has these builds you have to create a
`buster.js` config in the root of your project. It specifies
where the test suites are:

    var config = exports;

    config["Browser Tests"] = {
        environment: "browser",
        sources: [],
        tests: [ "resources/js/my_cljs_project_browser_test.js"
               , "resources/js/my_cljs_project_browser_optimized_test.js"]
    };

    config["Node Tests"] = {
        environment: "node",
        sources: [],
        tests: ["resources/js/my_cljs_project_node_test.js"]
    };

__Important__: Advanced compilation mode is only supported in the
browser. Node.js with advanced compilation is not supported just yet.

## Running your tests in buster.js

Now you are ready to run the tests.  Once you have some slave browsers
connected to `buster-server`, run:

    $> buster-test -e browser

If you want to execute the tests for node:

    $> buster-test -e node

You can also see the test suite as a static page (qUnit style)
by running this command in the root directory of your project:

    $> buster-static

## Crossplatform testing with lein-dalap

buster-cljs works both for Clojure and Clojurescript by using
[lein-dalap][lein_dalap]. 

This example demonstrates use of lein-dalap to translate the JVM
specific forms to JS compatible Clojurescript:

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


__Important__: In order to see your test results via buster, _you'll
need_ to wrap your assertions inside `it` clauses.

## Wrapping Up

Congratulations! You now know how to do most of the setup and
implementation of simple tests using buster-cljs. By now you should be
able to:

* Install buster-cljs
* Install buster with node and npm
* Setup your Clojurescript project with [lein-cljsbuild][lein_cljsbuild] and buster.js
* Run your tests both in the browser and in node
* Implement simple test suites

## What to read next

Read how to manage cross-platforms Clojure/Clojurescript code using
[lein-dalap][lein_dalap] and more about what [buster.js may
offer][busterjs]. 

[node_install]:http://joyent.com/blog/installing-node-and-npm
[lein_cljsbuild]:https://github.com/emezeske/lein-cljsbuild
[lein_dalap]:http://birdseye-sw.com/oss/lein-dalap/
[busterjs]:http://docs.busterjs.org/en/latest/
