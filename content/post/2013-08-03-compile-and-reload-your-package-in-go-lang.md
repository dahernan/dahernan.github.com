---
title: "Compile and reload your package in Go lang"
date: "2013-08-03"
categories:
tags:
  - "golang"
---

One of the Go lang goodness is the blazing fast compiler (can you hear me Scala? :P), that can give you the productivity of a interpreted language.
But to be, productive in the same way, at least building Web services, you need get rid of the recompile and kill/reload cycle for your application.

I tried a couple of utils to do that, and the most simple that it works is [*rerun*](https://github.com/skelterjohn/rerun)
You can just clone the repository, do a 'go install' and add the binary to your PATH variable. 

And now with a simple:

```
rerun --test github.com/dahernan/goangular/server
```

I have my webserver running, and every time that I make a change in the code, the test are passed again, and the server is reload. I know, right now my server It's very light and It does that in milliseconds, but I'm sure it's not going to reach the start time of any Java application :)  

