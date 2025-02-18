---
title: 'Patches of the Week: AllRecipes.com and eBay Whitespace'
authors:
- hallvord-steen
tags:
- sitepatching
license: cc-by-3.0
---

A new browser.js file is about to go out, the changes are as follows:

## Added patches

### allrecipes.com

<a href="http://allrecipes.com" target="_blank">allrecipes.com search failure fixed</a>, as reported and analysed <a href="http://my.opera.com/community/forums/topic.dml?id=341151" target="_blank">in this forum thread</a>.

This issue has a deeper history than you&#39;d tell from that discussion though. Yes, we fail to ignore byte order marks in the external script file - but the underlying reason is that <i>Opera doesn&#39;t decode the file as UTF-8, so there is not a byte order mark there - there are some junk characters instead</i>.

So why doesn&#39;t Opera understand that the <a href="http://images.media-allrecipes.com/js/bundles/main-site.min.js?v=114" target="_blank">external script</a> is UTF-8 encoded? The main site <i>is</i> served as UTF-8 after all..

Well, one obvious reason is that their script isn&#39;t labelled as UTF-8. Just sending the correct content-type and including the charset - <strong>Content-Type: text/javascript;charset=UTF-8</strong> for example, and we&#39;d be happy to ignore as many BOMs as required. But yet, shouldn&#39;t we be able to guess that a UTF-8 page has encoded its external scripts as UTF-8 too?

No, for security reasons Opera prior to 10.50 did not let the encoding of the main page affect the interpretation of included content served from a different server. We thought that was the <s>more paranoid</s>safer thing to do, since if something is interpreted with a different encoding than intended, chances are it could turn out to be readable by the wrong site or contribute to maliciousness somehow. And let me tell you: we&#39;re breaking <strong>millions</strong> of sites right now with this policy, usually in minor ways like an ad appearing as <a href="http://en.wikipedia.org/wiki/Mojibake" target="_blank">gobbledygok</a>.

(One example of a potential exploit - though perhaps not a good one, as how encodings work is not my strongest point - is how Google tends to put <strong>while(1);</strong> as the first line of data when it sends plain JS with private data in it. This line makes the data useless as an included script and is stripped away before GMail and other apps eval() the script. However, if a malicious page managed to load that script into an UTF-7 page and make Opera interpret it as UTF-7, it would be decoded into an entirely different stream of characters and there is a minuscule chance that somehow, that stream of characters might be readable when it shouldn&#39;t be, and that the evil site could decode it back to the actual characters and reveal your private data.)

I don&#39;t think we&#39;ve ever seen such an attack demonstrated in practise, and now we&#39;ve decided to drop support for UTF-7 (being the most dangerous alternate encoding and virtually unused on the web), and allow external scripts to inherit encoding information even when it comes from a different server. One of 10.5&#39;s most important compat fixes, and the reason why allrecipes.com doesn&#39;t need patching on 10.5 :)

### eBay

<a href="http://www.ebay.com" target="_blank">eBay</a> auctions should <i>hopefully</i> no longer suffer from huge empty areas in the middle of the page (as <a href="http://my.opera.com/community/forums/topic.dml?id=316241" target="_blank">discussed here</a> and in many other threads over the years.

This issue was caused by eBay&#39;s script having problems figuring out how large an IFRAME should be. They wanted to make it exactly as tall as the document inside. Initially, in the markup it is actually hardcoded to be 25000 pixels tall (!) and they would use scripts to read the actual size of the document and shrink the IFRAME accordingly.

The problem is that it&#39;s not all that simple to figure out across browsers and rendering modes what property gives the document&#39;s and what the window&#39;s height. The best reference I know is the table on <a href="http://www.howtocreate.co.uk/tutorials/javascript/browserwindow" target="_blank">howtocreate.co.uk&#39;s article on window dimensions</a>. To complicate matters further, Opera has actually changed the implementation - 9.2x and less tried to align with IE, 9.5x and above tries to align with Firefox/Safari.

eBay included a quirks mode document in an IFRAME inside a strict mode parent document, and tried to read the &quot;offsetHeight&quot; property from document.body to see how tall the document was. For reasons lost in the annals of history (not really - but I&#39;ll spare you the details) that one returns how tall the <strong>window</strong> is when Opera is in quirks mode. The script would naturally always conclude that the IFRAME was already perfectly sized for its contents, since it was as tall as itself, and no shrinking would happen.

## Removed patches

<span class='imgright'><img alt='' src='http://files.myopera.com/hallvors/blog/buggy.jpg' /></span>

Two patches are also removed this week, one for an animated menu on <a href="http://www.athome.co.jp" target="_blank">athome.co.jp</a> which is now handled in a generic way for all sites using this script, and one for overlapping item in <a href="http://mail.yahoo.com" target="_blank">Yahoo mail</a> search results (as in the image on the right), where we now handle vertical-align styling combined with TABLE tags like Firefox does, and like Yahoo mail expects.
