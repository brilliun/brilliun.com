---
title: "How Browsers Work"
date: 2017-01-14T22:36:48+09:00
lastmod: 2017-01-14T22:36:48+09:00
draft: false
keywords: []
description: ""
tags: ['browsers', 'frontend', 'javascript']
categories: ['Learning']

isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

<!-- toc -->

# Prior Knowledge

There are 3 main tasks for a browser to execute a web application:

* Fetching resources
* Page layout and rendering
* Javascript execution

## Browsers And Browser Kernels

[Comparison of web browser engines](https://en.wikipedia.org/wiki/Comparison_of_web_browser_engines)

### (WebKit)

* Web layout engine: WebCore
* Javascript engine: JavascriptCore

### Chrome

* Web layout engine: Blink
    * Originally forked from WebCore, a component of WebKit
* Javascript engine: V8

### Safari

* Web layout engine: WebCore
* Javascript engine: JavascriptCore
    * later evolved to **Nitro**, which compiles JavaScript into native machine code

### Firefox

* Web layout engine: Gecko
* Javascript engine: SpiderMonkey

### Microsoft Edge

* Web layout engine: EdgeHTML
    * forked from Trident, the engine of IE
* Javascript engine: Chakra


# Fetching Resources

## Prior Knowledge
### URL And URI

### Socket

### IP Protocol

### MAC Address


## Fetching from Cache
Given the URL of a resource on the web, the browser starts by checking its local and application caches. If you have previously fetched the resource and the appropriate cache headers were provided (see below), then it is possible that we are allowed to use the local copy to fulfill the request.

### Cache Headers

* Expires
* ETag
* Last-Modified
* Cache-Control

## Socket Reuse
Given a hostname and resource path, Chrome first checks for existing open connections it is allowed to reuse–sockets are pooled by {scheme, host, port}. 
Finally, if neither of the above conditions is matched, then the request must begin by resolving the hostname to its IP address–also known as a DNS lookup.

## DNS Lookup
If we are lucky, the hostname will already be cached in which case the response is usually just one quick system call away. If not, then a DNS query must be dispatched before any other work can happen.

## TCP Connection
New TCP connection -> Three-way handshake -> One round-trip

And possibly, once the TCP handshake is complete, and if we are connecting to a secure destination (HTTPS), then the SSL handshake must take place. This can add up to two additional round-trips of latency delay between client and server. If the SSL session is cached, then we can “escape” with just one additional round-trip.

## Dispatching Request
Finally, the browser is able to dispatch the HTTP request. Once received, the server can process the request and then stream the response data back to the client. This incurs a minimum of one network round-trip, plus the processing time on the server. Following that, we are done–unless the actual response is an HTTP redirect, in which case we may have to repeat the entire cycle once over. Have a few gratuitous redirects on your pages? You may want to revisit that decision.

## Extra Topic 1: Possible Latency
* If the server response does not fit into the initial TCP congestion window (4-15 KB), then one or more additional round-trips of latency is introduced
* SSL delays could get even worse if we need to fetch a missing certificate or perform an online certificate status check (OCSP), both of which will require an entirely new TCP connection, which can add hundreds and even thousands of milliseconds of additional latency.

## Extra Topic 2: Predictor

```
chrome://predictors/
```

A few example signals:

* mouse hover link
* typing in address bar
* user histories

Network optimization techniques used by Chrome

* DNS pre-resolve
    * Resolve hostnames ahead of time, to avoid DNS latency
* TCP pre-connect
    * Connect to destination server ahead of time, to avoid TCP handshake latency
* Resource prefetching
    * Fetch critical resources on the page ahead of time, to accelerate rendering of the page
* Page pre-rendering
    * Fetch the entire page with all of its resources ahead of time, to enable instant navigation when triggered by the user

# Page Layout and Rendering

## The Main Flow

![webkitflow](/images/how-browsers-work/webkitflow.png)

### Step1 - Parsing
* Parse HTML and generate DOM tree
* Parse CSS and generate style rules

### Step2 - Construction of Render Tree

DOM tree and style rules are combined to generate a render tree

> **Render tree**
> Render tree contains rectangles with visual attributes like color and dimensions. They are in the right order to be displayed on the screen

### Step3 - Layout

Give each node of render tree the exact coordinates where it should appear on the screen

### Step4 - Painting

The render tree will be traversed and each node will be painted using the UI backend layer.

 > **Note:** 
 > For better user experience, the rendering engine will try to display contents on the screen as soon as possible. It will not wait until all HTML is parsed before starting to build and layout the render tree. Parts of the content will be parsed and displayed, while the process continues with the rest of the contents that keeps coming from the network.
 

## The Order of Processing Scripts and Style Sheets

### scripts 
The model of the web is synchronous.

Authors expect scripts to be parsed and executed immediately when the parser reaches a ```<script>``` tag. The parsing of the document halts until the script has been executed. 

If the script is external then the resource must first be fetched from the network–this is also done synchronously, and parsing halts until the resource is fetched. 

There are two attributes of ```<script>``` tag which address this problem:

> **defer**
> I will not halt document parsing and will execute after the document is parsed. 

>**async**
> It will be parsed and executed by a different thread.

> *Note: The async attribute is only for external scripts (and should only be used if the src attribute is present)*.

### style sheets

Scripts might ask for style information during the document parsing stage. If the style is not loaded and parsed yet, the script will get wrong answers.

Therefore, Firefox blocks all scripts when there is a style sheet that is still being loaded and parsed. WebKit blocks scripts only when they try to access certain style properties that may be affected by unloaded style sheets.


## Render Tree Construction

Elements(renderer) on render tree know how to lay out and paint itself and its children. 

The renderers correspond to DOM elements, but the relation is not one to one. For example, non-visual DOM elements will not be inserted in the render tree. 

There are also DOM elements which correspond to several visual objects. These are usually elements with complex structure that cannot be described by a single rectangle.

Some render objects correspond to a DOM node but not in the same place in the tree. Floats and absolutely positioned elements are out of flow, placed in a different part of the tree, and mapped to the real frame.

## Layout

HTML uses a flow based layout model, meaning that most of the time it is possible to compute the geometry in a single pass. Elements later in the flow typically do not affect the geometry of elements that are earlier in the flow, so layout can proceed left-to-right, top-to-bottom through the document.

### Dirty Bit System
In order not to do a full layout for every small change, browsers use a "dirty bit" system. A renderer that is changed or added marks itself and its children as "dirty": needing layout.


## Painting



# Javascript Execution


# Further Readings

* [Javascript Startup Performance](https://medium.com/@addyosmani/javascript-start-up-performance-69200f43b201)
* [Chrome Design Docs](https://www.chromium.org/developers/design-documents)
* [Chrome Graphics](https://www.chromium.org/developers/design-documents/chromium-graphics)
* [The V8 Parser(s)](https://docs.google.com/presentation/d/1214p4CFjsF-NY4z9in0GEcJtjbyVQgU0A-UqEvovzCs/mobilepresent?slide=id.p)
* [GPU Animation: Doing It Right](https://www.smashingmagazine.com/2016/12/gpu-animation-doing-it-right/)
* [Slides by myself](https://drive.google.com/open?id=15kiEfYAGCtRPUJb9mGq-HOcvQg_5hj7fHpsrjDIeIPM)
