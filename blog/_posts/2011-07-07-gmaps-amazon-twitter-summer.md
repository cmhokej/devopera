---
title: GMaps, Amazon, Twitter, Summer
authors:
- ola-kleiven
tags:
- sitepatching
license: cc-by-3.0
---

## Added patches

PATCH-454, No closure for eval&#39;ed function expression means no way to close info box on Google Maps. Sometimes after logging in there is a pop-up that promotes new features of My Places. Due to a bug in Carakan it is impossible to close this pop-up, making it hard to use the site. Temporary fix while we wait for a Core fix.

PATCH-453, Avoid plugin-triggered document.cookie setting script deadlock bug on slideshare.net. Same issue as the TED.com patch from last time.

PATCH-452, Validate result from document.all.item. Earlier this year we made a change to what undefined means when used e.g to access something in a NodeList (should it throw or return the 0th element). Turns out it is not straightforward when dealing with standards/quirks mode and document.all collections. Who could imagine :p In short, it broke a few sites. <a href="http://my.opera.com/m25686/about/" target="_blank">Makoto Mizukami</a> wrote a nice patch to emulate a more complete document.all.item method.

PATCH-451, Add jQuery to Amazon pages. Many of you have probably noticed, some Amazon pages served to Opera have less functionality than in other browsers and throw various script errors. This is mainly because for some reason Amazon do not link in jQuery when serving pages to Opera. This patch just appends jQuery to Amazon pages.

## Changed patches



PATCH-444 - make sure @-tags are shown on Twitter too.

## Removed patches



356736 - add &quot;KHTML&quot; to navigator.userAgent on virtualearth.net map pages. Microsoft fixed the underlying issue long ago, but for some reason patch was never removed from browser.js. We only noticed when it broke goudengids.be listing pages. Sorry.

This will be the last update from here for a couple of weeks (unless something very bad happens). Enjoy your summer!
