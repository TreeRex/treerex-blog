---
layout: post
title:  "Open Dylan streams and character encodings"
date:   2013-08-15 15:23:56
categories: dylan unicode
---

As I look at extending Open Dylan to support Unicode, it is apparent that I
need to step back from the lexer and compiler and add support for character
set transcoders in the io library. This can be done before we can fully
support 32-bit characters:

- ISO-8859-1 (but not Windows CP1252) can be trivially converted to an (8-bit)
  `<character>`.
- UTF-8 encoded values between U+0000 and U+00FF can also be decoded into an (8-bit)
  `<character>`, since these are identical to ISO-8859-1.
- UTF-16 and UTF-32 values between U+0000 and U+00FF are also readable.

In other words it is possible to create a transcoding protocol and integrate
it into the streams module. This could be done with a `<wrapper-stream>` but
the more I think about it the more I think it should be added to
`<file-stream>` directly. A wrapper stream could then be used around a
`<byte-string-stream>` whenever you wanted to apply a transcoder to a byte
string.

One problem is that `<file-stream>` is a subclass of `<positional-stream>`: in
the presence of a transcoder the position is the logical character in the
stream. For multi-byte encodings like UTF-8 this means changing the position
is not necessarily a constant-time operation.

There is more to think about, especially how this
will tie into a Dylan that defines `<character>` as a 32-bit
Unicode value. For example, `make(<file-stream>)` takes an
`element-type:` parameter which is documented to
take a `byte-character` or `unicode-character`. Later on
this will default to `<character>`.

To be continued...

