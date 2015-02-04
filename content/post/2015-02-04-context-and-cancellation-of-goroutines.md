---
title: "Context and Cancellation of goroutines"
date: "2015-02-04"
categories:
tags:
  - "golang"
---

Yesterday I went to the event London Go Gathering, where all the talks had a great level, but particulary [Peter Bourgon](https://twitter.com/peterbourgon) gave me idea to write about the excelent package [context](http://blog.golang.org/context).

Context is used to pass request scoped variables, but in this case I'm only going to focus in cancelation signals.

Lets say that I have a program that execute a long running function, in this case `work` and we run it in a separate go routine.

```go
package main

import (
	"fmt"
	"sync"
	"time"

)

var (
	wg sync.WaitGroup
)

func work() error {
	defer wg.Done()

	for i := 0; i < 1000; i++ {
		select {
		case <-time.After(2 * time.Second):
			fmt.Println("Doing some work ", i)
		}
	}
	return nil
}


func main() {
	fmt.Println("Hey, I'm going to do some work")

	wg.Add(1)
	go work()
	wg.Wait()

	fmt.Println("Finished. I'm going home")
}
```
```
$ go run work.go
Hey, I'm going to do some work
Doing some work  0
Doing some work  1
Doing some work  2
Doing some work  3
...
Doing some work  999
Finished. I'm going home
```

Now imagine that we have to call that `work` function from a user interaction or a http request, we probably don't want to wait forever for that goroutine to finish, so a common pattern is to set a timeout, using a buffered channel, like this:

```go
package main

import (
	"fmt"
	"log"
	"time"
)

func work() error {
	for i := 0; i < 1000; i++ {
		select {
		case <-time.After(2 * time.Second):
			fmt.Println("Doing some work ", i)
		}
	}
	return nil
}

func main() {
	fmt.Println("Hey, I'm going to do some work")

	ch := make(chan error, 1)
	go func() {
		ch <- work()
	}()

	select {
	case err := <-ch:
		if err != nil {
			log.Fatal("Something went wrong :(", err)
		}
	case <-time.After(4 * time.Second):
		fmt.Println("Life is to short to wait that long")
	}

	fmt.Println("Finished. I'm going home")
}

```
```
$ go run work.go
Hey, I'm going to do some work
Doing some work  0
Doing some work  1
Life is to short to wait that long
Finished. I'm going home
```

Now, is a little bit better because, the main execution doesn't have to wait for `work` if it's timing out. 

But it has a problem, if my program is still running like for example a web server, even if I don't wait for the function `work` to finish, the goroutine it would be running and consuming resources. So I need a way to cancel that goroutine.

For cancelation of the goroutine we can use the context package. We have to change the function to accept an argument of type `context.Context`, by convention it's usuallly the first argument.
 

```go
package main

import (
	"fmt"
	"sync"
	"time"

	"golang.org/x/net/context"
)

var (
	wg sync.WaitGroup
)

func work(ctx context.Context) error {
	defer wg.Done()

	for i := 0; i < 1000; i++ {
		select {
		case <-time.After(2 * time.Second):
			fmt.Println("Doing some work ", i)

		// we received the signal of cancelation in this channel	
		case <-ctx.Done():
			fmt.Println("Cancel the context ", i)
			return ctx.Err()
		}
	}
	return nil
}

func main() {	
	ctx, cancel := context.WithTimeout(context.Background(), 4*time.Second)
	defer cancel()

	fmt.Println("Hey, I'm going to do some work")

	wg.Add(1)
	go work(ctx)
	wg.Wait()

	fmt.Println("Finished. I'm going home")
}

```
```
$ go run work.go
Hey, I'm going to do some work
Doing some work  0
Cancel the context  1
Finished. I'm going home
```

This is pretty good!, apart that the code looks more simple to manage the timeout, now we are making sure that the function `work` doesn't waste any resource.

These examples are good to learn the basics, but let's try to make it more real.
Now the `work` function is going to do an http request to a server and the server is going to be this other program:

```go
package main

// Lazy and Very Random Server 
import (
	"fmt"
	"math/rand"
	"net/http"
	"time"
)

func main() {
	http.HandleFunc("/", LazyServer)
	http.ListenAndServe(":1111", nil)
}

// sometimes really fast server, sometimes really slow server
func LazyServer(w http.ResponseWriter, req *http.Request) {
	headOrTails := rand.Intn(2)

	if headOrTails == 0 {
		time.Sleep(6 * time.Second)
		fmt.Fprintf(w, "Go! slow %v", headOrTails)
		fmt.Printf("Go! slow %v", headOrTails)
		return
	}

	fmt.Fprintf(w, "Go! quick %v", headOrTails)
	fmt.Printf("Go! quick %v", headOrTails)
	return
}
```
Randomly is going to be very quick or very slow, we can check that with `curl`

```
$ curl http://localhost:1111/
Go! quick 1
$ curl http://localhost:1111/
Go! quick 1
$ curl http://localhost:1111/
*some seconds later*
Go! slow 0
```

So we are going to make an http request to this server, in a goroutine, but if the server is slow we are going to Cancel the request and return quickly, so we can manage the cancellation and free the connection.

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"sync"
	"time"

	"golang.org/x/net/context"
)

var (
	wg sync.WaitGroup
)

// main is not changed
func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	fmt.Println("Hey, I'm going to do some work")

	wg.Add(1)
	go work(ctx)
	wg.Wait()

	fmt.Println("Finished. I'm going home")

}

func work(ctx context.Context) error {
	defer wg.Done()

	tr := &http.Transport{}
	client := &http.Client{Transport: tr}

	// anonymous struct to pack and unpack data in the channel
	c := make(chan struct {
		r   *http.Response
		err error
	}, 1)

	req, _ := http.NewRequest("GET", "http://localhost:1111", nil)
	go func() {
		resp, err := client.Do(req)
		fmt.Println("Doing http request is a hard job")
		pack := struct {
			r   *http.Response
			err error
		}{resp, err}
		c <- pack
	}()

	select {
	case <-ctx.Done():
		tr.CancelRequest(req)
		<-c // Wait for client.Do
		fmt.Println("Cancel the context")
		return ctx.Err()
	case ok := <-c:
		err := ok.err
		resp := ok.r
		if err != nil {
			fmt.Println("Error ", err)
			return err
		}

		defer resp.Body.Close()
		out, _ := ioutil.ReadAll(resp.Body)
		fmt.Printf("Server Response: %s\n", out)

	}
	return nil
}
```

```
$ go run work.go
Hey, I'm going to do some work
Doing http request is a hard job
Server Response: Go! quick 1
Finished. I'm going home

$ go run work.go
Hey, I'm going to do some work
Doing http request is a hard job
Cancel the context
Finished. I'm going home
```

As you can see in the output, we avoid the slow responses from the server. 

In the client the tcp connection is canceled so is not going to be busy waiting for a slow response, so we don't waste resources.

Happy coding gophers!.








