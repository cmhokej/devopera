---
title: New Java Plug-in Support and Object Parsing
authors:
- ola-kleiven
tags:
- sitepatching
- java
- object
- type
- classid
- plugins
- embed
license: cc-by-3.0
---

<p>10.50 introduces support for <a href="https://jdk6.dev.java.net/plugin2/" rel="nofollow" target="_blank">Java plugin2</a> and at the same time changes parsing of object tags to be in line with <a href="http://www.whatwg.org/specs/web-apps/current-work/multipage/the-iframe-element.html#the-object-element" rel="nofollow" target="_blank">HTML5</a>

This topic isn&#39;t strictly about sitepatching but will change site compatibility. Mostly for the better but there could be unexpected side effects on sites that expect certain behavior - thus becoming targets for site patching. More on that later.

Let&#39;s take the last first, object parsing. Assume the following quite typical markup (simplified):

	<object classid="clsid:CAFEEFAC-0016-0000-FFFF-ABCDEFFEDCBA">
		<param name="code" value="foo.class">
		<embed type="application/x-java-applet" code="foo.class">
	</object>

The object is here meant for IE. IE recognizes the classid and invokes the Java ActiveX control based on it. Other browsers don&#39;t understand the object and fall back to using the embed where there is a type attribute they can understand. ... </p><!--more-->Now let&#39;s add a type attribute to the object to make it more interesting:

	[html:object
		classid="clsid:CAFEEFAC-0016-0000-FFFF-ABCDEFFEDCBA"
		type="application/x-java-applet"]
	<param name="code" value="foo.class">
		<embed type="application/x-java-applet" code="foo.class">
	</object>

All of a sudden the object means something to non-IE browsers as well. So let&#39;s go through what they do with it.

<ul class="bullets"><li>IE: still acts on the classid</li><li>Chrome: looks at the content, discovers that there is an embed there and uses it. If there was no embed, it would have used the object (this is based on tests, not actual code reading, so it could be inaccurate)</li><li>Opera10: understands the type on the object and uses that.</li><li>Firefox: ignores objects with a non-empty classid attribute and uses the embed</li><li>Opera10.50: ignores objects with a non-empty classid attribute and uses the embed</li></ul>

Now this is messy, 4 browsers, 4 different selection algorithms. Imagine the author wanted to serve different content to IE vs. the rest - impossible with the above markup.

Notice that the last line says that Opera10.50 does the same as Firefox. This is the important bit. Opera 10.50 will now do like step 3 of the HTML5 object content selection algorithm allows:

<blockquote class="bbquote"><p>If the classid attribute is present, <strong>and has a value that isn&#39;t the empty string</strong>, then: if the user agent can find a plugin suitable according to the value of the classid attribute, and plugins aren&#39;t being sandboxed, then that plugin should be used, and the value of the data attribute, if any, should be passed to the plugin. <strong>If no suitable plugin can be found</strong>, or if the plugin reports an error, <strong>jump to the last step in the overall set of steps (fallback)</strong></p></blockquote>

and be compatible with Gecko browsers.

What are the implications? So far we know of one thing breaking because of this. It is unfortunately the most common <a href="http://www.bankid.no/" target="_blank">log-in solution</a> for Norwegian banks (BankID) so many will be upset if it breaks. We are working with the banks to resolve this though. The reason for this breaking is that they have markup similar to this:

	<object classid="clsid:CAFEEFAC-0016-0000-FFFF-ABCDEFFEDCBA">
		<param name="type" value="application/x-java-applet">
		<param name="code" value="foo.class">
	</object>

Previously Opera would scan the param list for a suitable type value so it would work. WebKit still does this, while Opera10.50 ignores the object since it has a non-empty classid value. As you can see there is no fallback, so nothing will happen. The above doesn&#39;t work in Gecko either but they have server side sniffing in place to serve an embed to Gecko-browsers.

I&#39;ve used Java applets in the above examples but the new parsing also applies to any other object type like Flash, Silverlight etc.

<strong>Java plugin2</strong>

For many years Opera has used JNI (Java Native Interface) to connect to the JRE (Java Runtime Environment). This meant that Opera had to implement certain features on its own, like network access and certificate handling. Two different implementations of the same non-trivial feature will rarely be completely identical, so also the case here. Opera has had some compatibility issues with Java. One example is the inability to log into the online bank Nordea in Denmark.

In 2008 Sun launched a major update to its Java plugin, named plugin2. This was a complete rewrite that improved both speed and stability as well as introduced some new features. One of the most interesting improvements is the usage of the NPAPI <a href="https://developer.mozilla.org/en/NPN_PluginThreadAsyncCall" target="_blank">NPN_PluginThreadAsyncCall</a> that in effect enables a Java applet to run in its own thread. This means that Java can crash without bringing the browser down and also that the browser can stay responsive even if the applet is working hard on something CPU intensive (best effect on multi-core CPUs).

From 10.50 Opera has also implemented the bits required to get this working. This may have some implications for users and developers.

First of all, you need at least JRE1.6.0_u14 (u18 is the latest for Windows when this is written) for this to work.

Opera will now run Java like any other browser, so there should be less compatibility issues.

There will no longer be a need to use the com.opera.* classes, like com.opera.HttpURLConnection. In fact they will no longer exist, so Java code that depends on this may start failing. So far we know of one example of this, the speed test applet on <a href="http://www.speedcheck.arcor.de/vodafone/start.jsp?k=1" target="_blank">Vodafone Germany</a> that detects Opera and gives an empty result for download speed in 10.50.

Javascript code like this (calling Java without any applet present):

	<script>
		java.lang.System.err.println("Hello");
	</script>

will no longer work. This has only worked in Opera and Gecko-browsers and is not widely used. It will continue to work in Gecko, but Sun strongly discourages this and it is on the list of <a href="https://jdk6.dev.java.net/plugin2/liveconnect/#DEPRECATED_FUNCTIONALITY" target="_blank">deprecated functionality</a>

Some refer to this specific feature as LiveConnect, but officially LiveConnect covers all script&lt;-&gt;applet communication. An applet is of course still scriptable like any other NPAPI plug-in.

If sites don&#39;t work after these changes, please file bugs if Opera is wrong or even contact the site yourself if you know the site is wrong or outdated.
