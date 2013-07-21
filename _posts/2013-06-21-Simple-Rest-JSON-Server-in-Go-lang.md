---
layout: post
tags : [golang] [json] [rest]
---

These days I'm experimenting with Go lang, I love the language. I like specially the simplicity, you can check out
in the presentation from Rob Pike, [Another Go At Language Design](http://www.stanford.edu/class/ee380/Abstracts/100428-pike-stanford.pdf).

So, my first try in Go is to do a small Rest API for serving JSON documents. Also try a little TDD in Go to see how easy it could be.

On the test it's easy to mock whatever you need, because function are first class citizens, 
so you can have implementations on fly. And some of the clasess of the standard library they have test utilities like 'httptest'

Here the test.

https://gist.github.com/dahernan/6047770

And now the implementation, with a little functional style.

https://gist.github.com/dahernan/6047777
