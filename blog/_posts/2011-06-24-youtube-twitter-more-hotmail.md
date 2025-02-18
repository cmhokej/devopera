---
title: YouTube, Twitter, More Hotmail
authors:
- ola-kleiven
tags:
- sitepatching
license: cc-by-3.0
---

## Added patches



PATCH-447, YouTube html5 fullscreen videos not rendering well. Core issue with stacking contexts of position:fixed elements. Patch is simply to add a z-index to the element. Proper fix in progress.

PATCH-446, Strip whitespace from script type attribute on americanexpress.com. Core issue, will be fixed.

PATCH-445, Fix Ctrl-G shortcut in Maconomy portal.

PATCH-444, Make Twitter hashtags visible. Twitter does some script magic and ends up with broken element nesting: <pre>&lt;span&gt;&lt;strong&gt;foo&lt;/span&gt;&lt;/strong&gt;</pre> in turn causing Opera&#39;s layout engine to be upset. This will be fixed with the new parser so we&#39;ll just patch it meanwhile.

PATCH-441, Script sets window.event manually for other events but not for (all) click events, breaks attachment download in Hotmail. This fixes clicking sometimes not working and the error messages in console. Be aware that attachment download is still somewhat non-obvious as attachments end up in an invisible frame if you use &quot;Open&quot; in the download dialog and the resource should open in Opera. We&#39;re looking into that as well, but for now use &quot;Save&quot; or select a different application to open in directly (like Adobe or Foxit Reader for PDFs)

PATCH-440, Avoid png-fix script hiding images on afexsuns. They&#39;re using attachEvent to apply some IE filter magic, effectively hiding them in Opera.

## Changed patches



PATCH-387, apply Apple menu fix to /support/ as well.

## Removed patches



PATCH-428, brand new Hotmail not working. Removed for 11.50 only as a Core fix was patched in. Load event for object-issue.

PATCH-425, Hide document.attachEvent to get W3C-event.button-value-compatible code. QQ.com, site changed.

USAirways is not compatible with WF2 spec required attribute. Site changed.
