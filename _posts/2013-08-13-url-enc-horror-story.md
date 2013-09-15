---
layout: post
title:  "A URL Encoding Horror Story"
date:   2013-08-13 09:10:00
categories: characterencoding unicode
---

There once was a web application that needed to create a link that included
the name of a journal. Its name was "Ûžno-Rossijskij muzykal’nyj
al’manah". But the link the application created didn't work. In fact, it
failed spectacularly to create a valid link. This is what it made:

> `Ûžno%2DRossijskij%20muzykal%2019nyj%20al%2019manah`

(Insert music from the shower scene in Psycho)

There are a two things wrong.

The first is that the initial two characters in the name are not encoded
correctly. According to [RFC 3986][rfc3986] (and the definitions in
[RFC 2234][rfc2234]) it's illegal for those characters to appear in a URI —
they _must_ be percent encoded in UTF-8. The first part of the name should
look like

>`%C3%9B%C5%BEno-Rossijskij…`

The second is related to the quotation marks in the second and third
words. This is U+2019 RIGHT SINGLE QUOTATION MARK. Notice how _al’manah_
appears in the generated link:

> `al%2019manah`

Whatever created this URL appears to have put the actual hexadecimal
code-point for the quote into the encoded string! But what you end up with
after decoding is "al 19manah" which is obviously wrong. What could lead to
this?

The ECMAScript standard, [ECMA-262][ecma-262], provides a function
**escape** that generates a percent encoded version of its argument. This
function diverges from RFC 3986 in two ways:


- A character whose code-point is `0xFF` or less is encoded as a two digit escape
  sequence: `0xDB` becomes `%DB`.
- A character whose code-point is greater than `0xFF` is encoded using a four digit
  escape sequence of the form `%uXXXX`. For example, U+2019 becomes `%u2019`.

Encoded with **escape** the title is

> %DB%u017Eno-Rossijskij%20muzykal%u2019nyj%20al%u2019manah

Unfortunately the W3C has rejected this syntax, and it isn't portable.

Perhaps whoever wrote the code to URL encode the title was thinking about the
ECMAScript function but misremembered the syntax? If that was the case I would
have expected the ž to appear as `%017E`, but it wasn't.

I really don't know what happened to this poor title. Here is what it should
look like:

> `%C3%9B%C5%BEno-Rossijskij%20muzykal%E2%80%99nyj%20al%E2%80%99manah`

These problems are endemic: any Unicode string that contains characters
outside of US-ASCII gets similar treatment by the application. It is very sad.


[rfc3986]: http://tools.ietf.org/html/rfc3986#section-2.1
[rfc2234]: http://tools.ietf.org/html/rfc2234#page-11
[ecma-262]: http://www.ecma-international.org/publications/standards/Ecma-262.htm
