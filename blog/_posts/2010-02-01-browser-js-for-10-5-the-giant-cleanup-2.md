---
title: 'Browser.js for 10.5: The Giant Cleanup'
authors:
- hallvord-steen
tags:
- sitepatching
license: cc-by-3.0
---

During the last two weeks, browser.js has been tested against Opera 10.5, and the updates both on bug fixing and on handling broken code means an unprecedented number of patches are now due for removal. :D

WARNING: This is probably the longest browser.js update post ever!

<strong>This post is evidence of remarkable progress on the bug fixing and site compatibility front</strong>. Whether we were talking plain bugs or errors due to Opera&#39;s handling of unexpected code or spec violations, Opera 10.5 contains the results of man-years of analysis and bug fixing, hopefully taking a great leap forward for site compatibility. (Unless Carakan, Vega and the rest of the new and exciting stuff brings a heap of NEW bugs that break websites, that is.. Time will show ;))

## New patches

<strong>Added - <a href="http://www.pchome.net" target="_blank">pchome.net</a> layout fix</strong>

Layout issue caused by a margin pushing floats out of their expected position. This is already fixed in core for 10.5.

<strong>Added - <a href="http://mail.yahoo.com">Y!Mail</a> chat enter fix</strong>

This issue was <a href="http://my.opera.com/community/forums/topic.dml?id=288637&amp;t=1265045705&amp;page=1#comment3106971">reported in the fora</a> and prompted by <a href="http://my.opera.com/sitepatching/blog/2010/01/11/patches-of-the-week-allrecipes-com-and-ebay-whitespace#comment15651521">the first comment</a> on the previous blog post I had a quick look. Turns out it was caused by a familiar problem..

Back when we implemented the much-requested rich text editing support, we weren&#39;t quite sure what scripts should be allowed to run inside a document with designMode enabled. (Before you ask - no, there was no spec for designMode back then. It&#39;s something browser vendors invented, and it&#39;s still plagued with crippling interoperability problems. HTML5 is going to handle at least some of it but I assume something as complex as rich text editing will remain underspecified for a while..)

Our testing back then indicated that other browsers stopped running scripts inside a document when designMode was enabled. Depending on the context, running scripts when a site doesn&#39;t expect you to can cause cross-site scripting exploits and similar security problems, so we made really sure we blocked script execution in editing mode. So far, so good... except that&#39;s <i>not</i> what the others are doing. Several broken sites later we realised that they still allow event listeners (like those added with <strong>document.addEventListener(&#39;keyup&#39;, ..)</strong> to run.

So because of this misunderstanding we don&#39;t send the event to Yahoo, and thus enter doesn&#39;t send your chat message. The patch works around that by making designMode sort of an alias for contentEditable, which does not disable scripts.

<strong>Added - <a href="http://www.toshiba.co.jp">Toshiba </a>script hang fix</strong>

A while ago, I noticed a site setting innerHTML to a markup string with an OBJECT loading Flash content and immediately trying to access the Flash from the same script thread. At the time, this worked in Firefox but not Opera - we didn&#39;t initialize the Flash plugin until after the script finished.

So, we&#39;ve had this bug hanging around saying we should initialize plugin objects immediately when innerHTML is set, and pause the script until the plugin is ready for scripting. Sounds simple? So I thought. Of course it wasn&#39;t.

At the Toshiba site, they have an OBJECT for IE and an EMBED fallback to load Flash. Both have the same ID. Further they also have a peculiar <strong>#topmovie:after { content: &quot;.&quot;; }</strong> style instruction that affects the object. When JavaScript tries to interact with their Flash content, it hooks up with the wrong object - one that Opera is not going to initialize - and starts waiting until it&#39;s ready for scripting. And while that script is waiting, no other scripts may run - breaking most interaction with the site.

<h2>Updated patches</h2>

<strong>Changed - <a href="http://www.ebay.com">eBay </a>white space to apply on more domain than .com</strong>

According to suggestions in comments on previous post - thanks, guys!

<strong>Changed <a href="http://www.walla.co.il">Walla.co.il</a> styling</strong>

Walla.co.il is improving their standards compliance, making patches obsolete.

<h2>Patches removed for 10.5</h2>

<strong>Removed - <a href="https://www.youtube.com" target="_blank">YouTube</a> menu painting</strong> - site changed

<strong>Removed CORE-17333 - The constructor property of DOM nodes should not be object</strong> - <a href="http://my.opera.com/hallvors/blog/2009/01/08/google-maps-vs-dom2-specification-1-0" target="_blank">this problem</a> now fixed in core

<strong>Removed CORE-17460 - Constructor property of event should be Event interface</strong> - same as above

<strong>Removed CORE-3776 - generic patch for window.scrollX/Y</strong> - implemented these <a href="https://developer.mozilla.org/En/DOM/Window.scrollX" target="_blank">non-standard properties</a>

<strong>Removed - <a href="http://www.expedia.com" target="_blank">expedia.com</a> car rental list</strong>

The issue here was when to encode angle brackets to entities when a site reads innerHTML. Apparently browsers should not do so for certain attributes if the value starts with javascript:. If you want a complex demo of this issue, compare the output from <a href="http://files.myopera.com/hallvors/testcases/CORE-26201.htm" target="_blank">this</a> script in your preferred browsers. Tell me if you find two different browser engines doing exactly the same thing, because I haven&#39;t!

<strong>Removed CORE-25405 - unclickable <a href="http://www.tianya.cn" target="_blank">tianya.cn</a> links</strong> - due to missing OBJECT closing tag (this fix is not actually in 10.5 builds quite yet, but should get there soon)

<strong>Removed - <a href="http://ebay.fr" target="_blank">ebay.fr</a> hangs, select.remove() options</strong> - core fix now landing for the spec violation discussed <a href="http://my.opera.com/hallvors/blog/2009/08/12/browser-js-update-ebay-sun-webmail-salesforce-2" target="_blank">in an earlier browser.js update post</a>

<strong>Removed - <a href="http://www.em.com.cn" target="_blank">em.com.cn</a> menu disappears</strong> - because we didn&#39;t know what order other browsers fired mousemove and mouseout events in when you moved the mouse from one element to another..

<strong>Removed - <a href="http://google.com/reader" target="_blank">Google Reader</a> blank page on &#39;v&#39;</strong> - a peculiar timing problem, I think Reader would try to navigate a popup before we were done loading about:blank.

<strong>Removed - <a href="http://www.picsearch.com" target="_blank">picsearch.com</a>/<a href="http://www.wimbledon.org" target="_blank">wimbledon.org</a>/<a href="http://www.adf.ly" target="_blank">adf.ly</a> loads blank</strong>

Ah. I&#39;ve been <a href="https://dev.opera.com/articles/view/using-capability-detection/" target="_blank">going on</a> about using feature detection rather than browser sniffing and <a href="https://dev.opera.com/articles/view/a-browser-sniffing-warning-the-trouble/" target="_blank">demonstrated</a> how sniffing can cause serious problems. All of a sudden, jQuery <a href="http://docs.jquery.com/Release:jQuery_1.3" target="_blank">releases an update without browser sniffing</a> and it turns out Opera has an unflattering core bug that breaks their feature detection and makes sites that use both FRAMESET and the new jQuery render nothing at all!

This rather unpleasant discovery depends on the fact that a form element is supposed to be associated with a FORM. A form, on the other hand, is supposed to live inside a document&#39;s BODY. A FRAMESET, on the other hand, will only be rendered if the browser hasn&#39;t seen any BODY at all..

So, when jQuery runs this somewhat opinionated snippet:

	// Check to see if the browser returns elements by name when
	// querying by getElementById (and provide a workaround)
	var form = document.createElement("form"),
	id = "why_is_this_needed";
	form.innerHTML = '<input name="foo">';
	var root = document.documentElement;
	root.insertBefore( form, root.firstChild );
	root.removeChild( form );

There was some logic inside Opera thinking: &quot;oops, there&#39;s a form being created here, and we don&#39;t have a BODY, so let&#39;s make one up&quot;. Bizarrely, this code would trigger even if all you did was <strong>document.createElement(&#39;select&#39;)</strong> without inserting <i>anything</i> in the document! A &quot;why_is_this_needed&quot; moment indeed..

Of course, when we reached the FRAMESET tag Opera ignored it because it had created a BODY for that document already.

The patch has a certain odd beauty, overwriting createElement() to make sure elements are created inside a fake document until we&#39;ve seen a real BODY or FRAMESET:

	document.createElement=(function(createElement){
		var tmpDoc = document.implementation.createDocument('http://www.w3.org/1999/xhtml/', 'html', null);
		return function(tagName){
			if( ! document.body ){
				return createElement.call( tmpDoc, tagName );
			}else{
				document.createElement=createElement; // this hack is not necessary anymore
				return createElement.call(this, tagName);
			}
		}
	})(document.createElement);

Surprisingly, it worked.

<strong>Removed - <a href="http://mail.yahoo.com" target="_blank">Y!Mail</a> chat enter fix</strong> - yes, this patch is being added for 9.x and 10.00/.10 and obsoleted for 10.5 at the same time. We intend to get the core fix that allows certain event listeners in designMode into 10.5, though it&#39;s not in just yet.

<strong>Removed - <a href="http://mail.yahoo.com" target="_blank">Y!Mail</a> XMLDocument usage</strong> - I&#39;m not even sure if we fixed it or they did

<strong>Removed - <a href="http://mail.yahoo.com" target="_blank">Y!Mail</a> send button opens attach dialog</strong> - Y!Mail had a huge, invisible INPUT type=file and used CSS clip() styling to &quot;cut&quot; away the parts that were above other buttons than &quot;attach&quot;. Unfortunately, Opera sent mouse events even to content that was clipped away.

<strong>Removed - <a href="http://www.allrecipes.com" target="_blank">allrecipes.com</a> byte order marks in JS files</strong> - as <a href="http://my.opera.com/sitepatching/blog/2010/01/11/patches-of-the-week-allrecipes-com-and-ebay-whitespace" target="_blank">promised</a> the <a href="http://en.wikipedia.org/wiki/Mojibake" target="_blank">Mojibake</a> problem is being fixed in core.

<strong>Removed - <a href="http://www.amazon.com.cn" target="_blank">amazon.com.cn</a> menus disappear</strong> - same problem with mouse event order as em.com.cn. The patch was rather tricky to get right..

<strong>Removed - <a href="http://www.asahi.com" target="_blank">asahi.com</a> never stops loading</strong> - one of Japan&#39;s most important news papers was inserting scripts both from timeouts and with document.write() and the mix confused Opera into a never-stops-loading state. We&#39;ve had similar issues elsewhere - <a href="http://www.digg.com" target="_blank">digg.com</a> among other high-profile sites, so this core fix is much appreciated.

<strong>Removed - <a href="http://bbs.pcpop.com" target="_blank">bbs.pcpop.com</a> shows blank</strong> - caused by styling of nested forms (one of these things HTML4 didn&#39;t want to deal with)

<strong>Removed - <a href="http://www.cdon.com" target="_blank">cdon.com</a> shopping cart empty</strong> - a convoluted layout issue with relative position and nested floats.

<strong>Removed CORE-21773 - <a href="http://www.china-pub.com" target="_blank">china-pub.com</a> window.event getter function&#39;s caller property is wrong</strong> - site used an &quot;IE features emulation&quot; script for non-IE browser, defining window.event among other things. Of course we support window.event already, but it was redefined with a getter function anyway, and within that function they tried to read the function&#39;s caller property and exposed a bug.

<strong>Removed DSK-246430 - <a href="http://digg.com" target="_blank">digg.com</a> NSL/no content</strong> - another of these &quot;never stops loading due to peculiar script self-mutations&quot; bugs

<strong>Removed - <a href="http://grainger.com" target="_blank">grainger.com</a> document load</strong> - we <a href="http://my.opera.com/hallvors/blog/2009/05/20/the-day-supporting-document-onload-became-a-bug" target="_blank">realised</a> we should stop supporting document.onload, and now the fix hits the road.

<strong>Removed CORE-10896 - <a href="http://groups.google.com" target="_blank">google groups</a> not 100% tall</strong> - long standing annoyance regarding nested tables with 100% height

<strong>Removed - <a href="http://www.ironmaiden.com" target="_blank">ironmaiden.com</a> disappearing menu</strong> - another mouse event order problem, exacerbated by the especially weird markup and scripting their menu uses.

<strong>Removed - <a href="http://www.hotmail.com" target="_blank">Hotmail</a> oncontextmenu</strong> - :yes:

<strong>Removed - <a href="http://maps.google.com" target="_blank">Google maps</a> oncontextmenu</strong> :yes:

<strong>Removed - <a href="http://www.nasdaq.com" target="_blank">nasdaq.com</a> overlapping content</strong> - another layout problem regarding CSS positioning

<strong>Removed - <a href="http://news.sina.com.cn" target="_blank">news.sina.com.cn</a> CDATA parsing</strong> - odd bug where the sheer size of a script element caused it to be split into two text chunks, and the CDATA escape stuff caused a syntax error.

<strong>Removed - <a href="http://photobucket.com" target="_blank">photobucket</a> disappearing menu</strong> - probably also a mouse event order issue

<strong>Removed - <a href="http://picasaweb.google.com" target="_blank">Picasaweb</a> upload button too narrow</strong> - bug in calculations of the width of an element with overflow:hidden, now fixed.

<strong>Removed - <a href="http://www.portalaz.com.br" target="_blank">portalaz.com.br</a> disappearing menus</strong> - mouse event order issue

<strong>Removed - <a href="http://rent.toyota.co.jp" target="_blank">rent.toyota.co.jp</a> window.open issue</strong> - similar to the Reader issue but different. I&#39;m not sure if I ever understood this.

<strong>Removed - <a href="http://www.sfc.jp" target="_blank">sfc.jp</a> noscript content shows</strong> - styling NOSCRIPT elements with display:block turned out to be a bad idea when Opera defaulted to hiding them with a display:none :)

<strong>Removed - <a href="http://www.shutterfly.com" target="_blank">shutterfly.com</a> Array splice on 0-length arrays</strong> - splicing an empty array didn&#39;t work very well in Opera.

<strong>Removed - <a href="http://status.renren.com" target="_blank">status.renren.com</a> missing comments</strong> - same as the CDOn bug

<strong>Removed - <a href="http://www.towerrecords.co.jp" target="_blank">towerrecords.co.jp</a> disappearing menu</strong> - mouse event order problem

<strong>Removed - <a href="http://www.weather.com" target="_blank">weather.com</a> disappearing menu</strong> - mouse event order problem

<strong>Removed - <a href="http://www.wikimapia.org" target="_blank">wikimapia.org</a> oncontextmenu</strong> -  :yes:

<strong>Removed - <a href="http://google.com/finance" target="_blank">Google Finance</a> missing stock details</strong> - a convoluted issue involving Google&#39;s script working around an Opera bug, and the workaround failing for the specific CSS Finance was using.

<strong>Removed - <a href="http://www.pchome.net" target="_blank">pchome.net</a> broke margin</strong> - as stated at the top of the post, already fixed for 10.5

<strong>Removed - <a href="http://zhangmen.baidu.com" target="_blank">zhangmen.baidu.com</a> textarea typing</strong> - we supported IE&#39;s `document.selection` object without full support for the IE selection APIs. This site (and some others) would detect document.selection and assume the entire API existed. In 10.5 we fix this by removing document.selection support.

Wow!
