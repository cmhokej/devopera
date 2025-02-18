---
title: Facebook, Orkut++
authors:
- ola-kleiven
tags:
- sitepatching
license: cc-by-3.0
---

## Added patches

PATCH-237, <strong>Delay script execution on <a href="http://di.jal.co.jp/jalmap/" target="_blank">JAL</a> map to avoid deadlock in plug-in scripting.</strong> Bug in Opera.

PATCH-303, <strong>Reporting different clientHeight and scrollHeight for TEXTAREA breaks commenting on fantasy.nfl.com</strong>. Opera doesn&#39;t take textarea padding into account when calculating scrollHeight, causing while(clientHeight&gt;scrollHeight) to loop forever. Turns out no browser does this the same way, but WebKit appears to be most sane and consistent, so we&#39;ll do the same.

PATCH-269, <strong>Reposition invisible file input on rediff.com</strong>. Like last week&#39;s <a href="http://www.freemail.hu" target="_blank">Freemail.hu</a> fix, <a href="http://www.rediff.com/" target="_blank">Rediff</a> is a site that is important in a specific region. The problem we&#39;re patching is that attaching a file while writing an E-mail does nothing. As most webmail sites these days, they put an invisible <pre>&lt;input type=&quot;file&quot; onchange=&quot;this.form.submit()&quot;&gt;</pre> above some stylish &quot;Attach file&quot; text and graphic. The idea is that when you think you click their &quot;Attach file&quot; text, you should actually click the invisible &quot;choose&quot; button, get the file dialog, and once you&#39;ve made a selection the file uploads automatically.

The problem is sort of both Opera&#39;s and Rediff&#39;s fault - Rediff fails to position the invisible file input so that users click the invisible &quot;choose&quot; button due to their browser-specific CSS, and Opera fails to make the &quot;text&quot; part of the file input clickable. Opera requires click on <i>choose</i> button, not text box part of the input.  The result is that when you click the &quot;Attach file&quot; text you&#39;re clicking the invisible &quot;text&quot; part of the file input, and nothing happens.

For Rediffmail we also ship a patch to <strong>make sure window.focus() focuses the BODY element</strong> inside the IFRAME. When you call window.focus() on an IFRAME with designMode, Opera focuses the document and the cursor appears. However, if the page inside the IFRAME uses

	<body contentEditable="true">

instead of designMode, Opera fails to put the cursor inside the BODY element when the page calls focus(). Worse: it actually removes the cursor from BODY if it&#39;s already there. This makes it hard or impossible to type a letter when the site keeps trying to focus the window! The fix is to overwrite window.focus() with a method that calls document.body.focus(). It&#39;s fixed in core now, and several patches for this bug across many sites will be obsolete when that fix ships. :)

PATCH-304, <strong>avoid extra linebreaks in Orkut chat box</strong>. Opera requires cancelling keypress, not keydown - the key event issues strike again. Issue was mentioned in the shout box to the right of this blog by <a href="http://my.opera.com/rafaelluik/" target="_blank">Rafael Luik</a> - thanks ;)

## Changed patches



PATCH-264, <strong>Enable Facebook @mentions</strong>. The original patch broke the FB music player (unable to change track). Normally, if you through the DOM add an inline script to the page it will run immediately. However, if there is an active BeforeScript event listener this changes - the current thread will finish, then BeforeScript runs, and then the new script.

The solution is to make sure we remove the BeforeScript listener when it&#39;s fixed the sniffing that prevents @mentions from working.

PATCH-272, <strong>Remove non-functional addEventListener from XHR objects</strong> was changed to use the delete keyword. Thanks to Andrea Giammarchi for <a href="http://webreflection.blogspot.com/2010/09/opera-inevitably-unexpected.html" target="_blank">blogging about the problem</a> with the old patch.

## Removed patches



PATCH-30, <strong>MSNBC sniffing hides Flash content</strong>. Site fixed their sniffing to include Opera, so the patch we <a href="http://my.opera.com/sitepatching/blog/2010/09/22/hungary-here-we-come" target="_blank">fixed last week</a> wasn&#39;t required for much longer. :D
