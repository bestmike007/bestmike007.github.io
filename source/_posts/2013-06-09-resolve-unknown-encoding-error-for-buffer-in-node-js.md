---
title: 'Resolve &#8216;Unknown Encoding&#8217; Error for Buffer in Node.js'
author: mike
layout: post
permalink: /2013/06/resolve-unknown-encoding-error-for-buffer-in-node-js/
categories:
  - IT
tags:
  - Encoding
  - Nodejs
---
According to the official document for Buffer implementation, the Buffer class only supports a few encoding charsets including &#8216;ascii&#8217;, &#8216;utf8&#8242;, &#8216;utf16le&#8217;, &#8216;ucs2&#8242;, &#8216;base64&#8242;, &#8216;binary&#8217;, and &#8216;hex&#8217; [<a title="Node.js Buffer class" href="http://nodejs.org/api/buffer.html#buffer_buffer" target="_blank">http://nodejs.org/api/buffer.html#buffer_buffer</a>]. So if you&#8217;re calling &#8216;toString&#8217; method from a Buffer instance you should probably get an &#8216;Unknown Encoding&#8217; error , e.g. reading a text file encoded by &#8216;latin1&#8242; or &#8216;ISO-8859-1&#8242;. This post provides a quick fix to this problem.

<!--more-->

This solution requires iconv. You can install it with npm.

<pre>npm install iconv</pre>

And then before using Buffer.toString method, override it with the following code.

<pre>var Iconv = require('iconv').Iconv;

Buffer.prototype._$_toString = Buffer.prototype.toString;
Buffer.prototype.toString = function(charset) {
    if (typeof charset == 'undefined' || charset == 'utf8' || charset == 'utf16le' || charset == 'ascii' || charset == 'ucs2' || charset == 'binary' || charset == 'base64' || charset == 'hex') {
        return this._$_toString.apply(this, arguments);
    }
    var iconv = new Iconv(charset, 'UTF-8');
    var buffer = iconv.convert(this);
    var args = arguments;
    args[0] = 'utf8';
    return buffer.toString.apply(buffer, args);
}</pre>

And that&#8217;s all.

P.S. This solution is originated from <a href="http://stackoverflow.com/questions/14551608/cant-find-encodings-for-node-js" target="_blank">http://stackoverflow.com/questions/14551608/cant-find-encodings-for-node-js</a>