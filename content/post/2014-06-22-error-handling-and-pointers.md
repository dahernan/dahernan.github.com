---
title: "Error handling and pointers in Go lang"
date: "2014-06-22"
categories:
tags:
  - "golang"
---

A couple of weeks ago I went to the [Go London User Group](http://www.meetup.com/Go-London-User-Group/) and one of the talks was about [error handling in Go](https://www.youtube.com/watch?v=Ph4eYD7Bgek) by Daniel Morsing (really good talk).

After thinking, I rembembered that there is another antipattern when using pointers to error, for example here.

```go
// http://play.golang.org/p/Xktu2Jz-Zc

package main

import "fmt"

type MyError struct {
	path string
	msg  string
}

func (e MyError) Error() string {
	return e.path + ":" + e.msg
}

func uglyBug(blowup bool) error {
	var err *MyError = nil
	if blowup {
		err = &MyError{"path1", "msg1"}
	}
	return err
}

func main() {
	e1 := uglyBug(false)
	if e1 != nil {
		fmt.Println("Pointer -> ", e1)
	} else {
		fmt.Println("It's ok")
	}

}
```

Output:
```
Pointer ->  <nil>
```

The value of *e1* is going to be _not nil_, because the interface is not nil (it needs to have the value and the type nil, to return nil).

To fix this, you have to explicit return nil.

```
package main

import "fmt"

type MyError struct {
	path string
	msg  string
}

func (e MyError) Error() string {
	return e.path + ":" + e.msg
}

func fixed(blowup bool) error {
	if blowup {
		return &MyError{"path1", "msg1"}
	}
	return nil
}

func main() {
	e1 := fixed(false)
	if e1 != nil {
		fmt.Println("Pointer -> ", e1)
	} else {
		fmt.Println("It's ok")
	}

}
```
Output:
```
It's ok
```

So always do a explicit return nil, when you need an interface to be nil.






