---
title: "Service Discovery with Mangos and Go"
date: "2015-03-17"
categories:
tags:
  - "golang"
  - "nanomsg"
  - "mangos"
---

[Mangos](https://github.com/gdamore/mangos) is a pure Go implementation of [Scalable Protocolos](http://nanomsg.org/index.html), there is a good introduction about it, in this [blog post](http://www.bravenewgeek.com/fast-scalable-networking-in-go-with-mangos/).

Also, that blog gave me the idea to implement a simple way to do service discovery. The common solution for service discovery is using some kind of distribuited system to store the changes, the usual solutions for this problems are [Zookeeper](http://zookeeper.apache.org/), [etcd](https://github.com/coreos/etcd), [consul](https://www.consul.io/), [eureka](https://github.com/Netflix/eureka).

That's usually pretty good, but sometimes you don't need that kind of solution to solve a simple problem. You may not have many servers and using a library could be enough to solve that problem.

So I released a small library to do service discovery based in Mangos, I called:

 * [gopherdiscovery](https://github.com/dahernan/gopherdiscovery)

It uses one socket for emiting a survey heartbeat.
![Survey](https://docs.google.com/drawings/d/1XvKaxDBhWLeOoQHiofWJNAvo1139i_3Z9NCUDZJCJ8c/pub?w=480&h=360)

With the heartbeat from the server you can know the clients that are online, and whenever a client goes down, you'll know it in the next heartbeat slot. 

But you need to notify in some way to the interested parties that new clients are connected or disconnected. And for that I use mangos pub/sub socket to publish the changes.

![Pub/Sub](https://docs.google.com/drawings/d/14vcCmiCFywml5P9ApN76hRbcb7B_PkzSo6XUoOAMcRg/pub?w=480&h=360)

You can have clients that are aware of the changes in other peers, or you can have a pure subscriber that only receives the changes, it doesn't act like a client.

## Use case

A good use case is for example, to update the peers in [groupcache](https://github.com/golang/groupcache)

We can have a very light server for service discovery:

```go

urlServer := "tcp://10.0.0.100:40007"
urlPubSub := "tcp://10.0.0.100:50007"

// on the server
server, err := gopherdiscovery.Server(urlServer, urlPubSub, opts)
server.Wait()

```

And for every single client, use [groupcache](https://github.com/golang/groupcache) in combination with [gopherdiscovery](https://github.com/dahernan/gopherdiscovery) to update the peers that are doing the cache.

```go
var peers chan []string

urlServer := "tcp://10.0.0.100:40007"
urlPubSub := "tcp://10.0.0.100:50007"
me := "http://10.0.0.1:8080"

// any of the peers
poolGroupCache := groupcache.NewHTTPPool(me)

client, err := gopherdiscovery.ClientWithSub(urlServer, urlPubSub, me)

peers, err = client.Peers()	
for nodes := ranges peers {
	// every time there is a change in the peers -> Set in groupcache
	poolGroupCache.Set(nodes...)	
}

```

Another example could be, adding and removing proxies from a loadbalancer.

If you like it, [check the source and the docs](https://github.com/dahernan/gopherdiscovery), and any feedback is welcome!
