---
layout: post
tags : [docker, elasticsearch]
---

In this post I'm going to explain how to create and use a docker image, for easy access to [elasticsearch](http://www.elasticsearch.org) for development purposes.
If you don't know [docker](http://docker.io/), just check it out. In my opinion is the way that everybody is going to distribuite webapps soon.

In the same way that the project has library dependencies, your app usually depends on more things. Let's say the database or in this case the search engine. 
Why not distribute the dependency with the code? Well even if you have a VM could be to much.

With containers that is over, now you can distribute with your source a Dockerfile, or even [have yor own repository of docker images](http://blog.docker.io/2013/07/how-to-use-your-own-registry/).

For my dependency [elasticsearch](http://www.elasticsearch.org) a need a new Dockerfile, so the next developer could reproduce my environment very easy.

So this is the content of my *Dockerfile*.

{% gist 6149626 %}

Now I can build the image with:

```
$ docker build -t="elasticsearch" .
```

And after a while if I do:

```
$ docker images
```

{% gist 6149703 %}

I can see my new image *elasticsearch* there, and finally I can run elasticsearch in that container.

```
$ docker run -d elasticsearch
```

{% gist 6149724 %}

Now my elasticsearch instance is running in its own container, forwarding the port 9200. So if I want to try to run a elasticsearch cluster, I'm only have to run more containers.




