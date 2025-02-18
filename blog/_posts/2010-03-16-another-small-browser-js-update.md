---
title: Another Small Browser.js Update
authors:
- ola-kleiven
tags:
- sitepatching
license: cc-by-3.0
---

A few more updates to browser.js. This log covers the update from March 1. file to March 16. file.

## New patches



<a href="http://www.imdb.com" target="_blank">IMDB.com</a> - videos not working. Carakan is slightly too strict in security checking here. This patch should be short lived.

<a href="http://blogger.com/" target="_blank">blogger.com</a> - work around side effect of spoofing, improves HTML view.

## Changed patches



<a href="http://cajamadrid.es/" target="_blank">cajamadrid.es</a> - work around site bug that hides login form (and work around the core bug that now prevents the previous patch from doing what it&#39;s supposed to). First reported <a href="http://my.opera.com/community/forums/topic.dml?id=249032" target="_blank">here</a>, thanks .

HVMenu - minor change to fix an old version, spotted in <a href="http://list.opera.com/pipermail/opera-users/2010-March/029264.html" target="_blank">a message to the opera-users mailing list</a>. Here&#39;s a quirk story for you: back in the dark ages of medieval DHTML effects when this script was written, the author wanted to listen to the onload event. Here&#39;s what he did, somewhat snipped:

	var Par=parent.frames[0]&&FirstLineFrame!=SecLineFrame?parent:window;
	var Doc=Par.document;
	var Bod=Doc.body;
	var Trigger=NavYes&&!Opr?Par:Bod;

	if(Trigger.onload)Dummy=Trigger.onload;
	Trigger.onload=Go;

	function Dummy(){return}

	function Go(){
		Dummy();

Look closely at the line setting the &quot;Trigger&quot; variable.. Apparently, the Opera he was testing with back then supported setting document.body.onload. I have absolutely no clue why he goes out of his way to set document.body.onload in Opera instead of window.onload, but that&#39;s what the script does! (Notice also how he is corteous enough to keep a reference to any previously defined onload listener and execute it.)

As <a href="http://my.opera.com/hallvors/blog/2009/05/20/the-day-supporting-document-onload-became-a-bug" target="_blank">mentioned earlier</a>, implementing the humble onload event correctly is a lot harder than it might seem. At some point in history, we noticed that other browsers didn&#39;t really support setting document.body.onload - instead, a &lt;BODY onload=&quot;&quot;&gt; attribute set window.onload directly. When we changed Opera, HVMenu broke because it relied on onload events firing on body. We noticed and I added this to browser.js for HVMenu:

	defineMagicVariable.call(opera, 'Trigger', function(){ return document; }, null);

Of course, once we realised we weren&#39;t supposed to support document.onload either this turned out wrong. <strong>Again</strong>. Isn&#39;t web compat fun?

<a href="http://mail.yahoo.com/" target="_blank">Y!Mail</a> chat enter fix. This was accidentally removed from the 10.50 file. It has been re-introduced and improved, so now enter works and focus stays in the input field. This should solve problems like lost input focus, the about:blank page&#39;s title &quot;Blank page&quot; added to your chat messages :eyes:, truncated long sentences and more - as described <a href="http://my.opera.com/sitepatching/blog/2010/02/01/browser-js-for-10-5-the-giant-cleanup-2#comment17692231" target="_blank">in earlier comments</a> (thanks for complaining, <a href="http://my.opera.com/Galileo/" target="_blank">Galileo</a> ;)).

<a href="http://www.ebay.com" target="_blank">eBay</a> huge <a href="http://my.opera.com/community/forums/topic.dml?id=316241" target="_blank">white space in item listing</a>. Patch improved so it should work more reliably.

## Removed patches

<a href="http://facebook.com/" target="_blank">facebook.com</a> - Arrow navigation of image galleries - fixed by facebook.

We hope it all works for you :)

<i>Ola and Hallvord</i>

(we&#39;re co-writing this post although it appears as posted by Ola because he started it ;))
