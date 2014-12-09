---
title: "Docker image for Go service from the Scratch"
date: "2014-12-09"
categories:
tags:
  - "golang"
  - "docker"
---

Today I changed the script to dockerize go projects [godockerize](https://github.com/dahernan/godockerize), to support the 'scratch' image.

The way I do it, is cross compile for Linux, and after that add the binary to the image, that's all. Ready to build and run.

### To enable cross compile on Mac to build Linux executable

Follow this steps

```
$ cd /usr/local/go/src

$ sudo GOOS=linux CGO_ENABLED=0 ./make.bash --no-clean

```

You can try it doing, for example:

```
$ GOOS=linux CGO_ENABLED=0 go build -o gopherscraper

# Checking that is linux binary
$ file gopherscraper
gopherscraper: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped

```

## Use the new flag 'scratch'

Once you activate cross compilation you can use the new flag to generate and build the Docker image from the 'scratch'

For example:

```
$ cd $GOPATH/src/github.com/dahernan/gopherscraper

# Run godockerize using -scratch
$ godockerize -expose 3001 -scratch

Dockerfile generated, you can build the image with:
$ docker build -t gopherscraper .


# Run my microservice
$ docker run --rm  -p 3001:3001 -e ES=192.168.1.101 gopherscraper

```

You can get a really light image, using this method :)

```
REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
gopherscraper                 latest              8dad2e0ca7f9        55 minutes ago      10.61 MB
```
