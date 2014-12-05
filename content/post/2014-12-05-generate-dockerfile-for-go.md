---
title: "Generate a Dockerfile for Go Microservice"
date: "2014-12-05"
categories:
tags:
  - "golang"
  - "docker"
---

As part of doing a few Go REST Services, for every project I usually put a Dockerfile to compile an run the project in a Docker container.
The Dockerfile it's always the same, just change it the project name, and I'm using [godep](https://github.com/tools/godep).

So in DRY spirit a did a Minimal Viable Script to generate the Dockerfile for me, the project is in Github and I called [godockerize](https://github.com/dahernan/godockerize).

The Dockerfile is based in the golang image, it follows these steps.

* Based in golang:stable
* Use [godep](https://github.com/tools/godep) for vendoring the dependencies. (if you are not using godep, It will break).
* Uses the project name and the root directory as a ENTRYPOINT
* Restores the dependencies via 'godep restore'
* Compiles the project
* Exposes the port

Is inspired by this two blog post.

* http://blog.charmes.net/2014/11/release-go-code-and-others-via-docker.html
* https://blog.golang.org/docker

### Install godockerize
```
$ go get github.com/dahernan/godockerize

```

### Usage Example
```
# Go to the root directory of your project, for example
$ cd $GOPATH/src/github.com/dahernan/gopherscraper


# Run godockerize exposing the port 3001
$ godockerize -expose 3001

Dockerfile generated, you can build the image with:
$ docker build -t gopherscraper .

```

### Run my microservice

```
$ docker run --rm  -p 3001:3001 -e ES=192.168.1.101 gopherscraper

```

