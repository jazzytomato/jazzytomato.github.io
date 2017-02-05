---
layout: post
title: Hacker News Lookup. A chrome extension built with ClojureScript
---
![HNLookup screenshot]({{ site.baseurl }}/images/hnlookup/hnlookup-screenshot.png)

## Problem

[Hacker News](https://news.ycombinator.com/ "Hacker News website") is a social news website focusing on computer science and entrepreneurship, and has become the front page of the web for many people (me included). ["I have not found a public internet forum with better technical commentary"](http://danluu.com/hn-comments/) and I would usually prefer reading comments on HN rather than on the original website. I would often find myself searching for the HN thread on the pages I visit, or when someone sent me a link to an article or resource and because this was happening so often, I had to do something about it; I had to ease this painful process that takes more than 3 clicks.

## Solution

Hacker News Lookup is a minimal and non-intrusive Chrome extension that allows you to lookup on Hacker News the page that you are currently viewing, and browse for the related topics. It is simple, fast and beautiful :smile:.

It only runs when you click on the extension button and queries the [algolia api](https://hn.algolia.com). It does not track any of your browsing history.

The extension is available on the Chrome store here and the source code here: [https://github.com/jazzytomato/hnlookup](https://github.com/jazzytomato/hnlookup)


## Vision

The extension is doing the bare minimum; however, I believe there could be more to it than just showing hits on Hacker News. I would like to allow people to browse the web as part of a community (adding adapters for other sources, like Reddit), but I am not sure yet if I will structure the codebase in that direction or if that will be part of another project.

## Building a chrome extension with ClojureScript

I had already developed a Chrome extension in the past, but the development process was cumbersome.

Luckily, I was in the course of learning Clojure (with [this awesome book](http://www.braveclojure.com/clojure-for-the-brave-and-true/)), and watching some random Clojure tech talks when I heard about [figwheel](https://github.com/bhauman/lein-figwheel). As I learned more about ClojureScript, I realised that it was a game changer. I do a lot of back-end programming at work, and I am having a hard time keeping up with the pace at which the Javascript ecosystem evolves. What a relief when you realise that *you don't have to*: [There is no better way of developing front-end application than with ClojureScript](https://www.youtube.com/watch?v=gsffg5xxFQI).

Below is a 20 seconds video example of what the development workflow looks like (look at the extension on the right as I change and evaluate the code) 

<iframe width="720" height="400" src="http://www.youtube.com/embed/a0K2Cfnp7q8" frameborder="0" allowfullscreen="allowfullscreen"> </iframe>

 Here is what happens in the video:

* Evaluation of the code directly in the editor (the selected code is sent in the live REPL at the bottom of the editor)
* The code evaluated is connected to the chrome extension, and changes the current state of the app
* Modifications to the loading component are hot reloaded
* Errors and warnings are intercepted and sent to the page

Sweet.


<img style="display: block; margin: auto;" src="http://i.memeful.com/media/post/3M2aPw2_700wa_0.gif" alt="This will display an animated GIF" />
<br>
