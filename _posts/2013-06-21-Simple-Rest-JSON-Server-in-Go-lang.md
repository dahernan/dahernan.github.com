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
```go
package main

import (
	"github.com/bmizerany/assert"
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestJsonServerReturnsJsonDocumentWithRightHeaders(t *testing.T) {

	req := new(http.Request)

	builder := func(req *http.Request) interface{} {
		return map[string]interface{}{
			"test": "test_value",
		}
	}

	jsonServer := JsonServer(builder)
	responseWriter := httptest.NewRecorder()

	// call to test
	jsonServer(responseWriter, req)

	contentType := responseWriter.Header().Get("Content-Type")
	assert.Equal(t, contentType, "application/json")

	assert.Equal(t, "{\"test\":\"test_value\"}\n", responseWriter.Body.String())

}
```
And now the implementation, with a little functional style.

```go
package main

import (
  "encoding/json"
  "github.com/bmizerany/pat"
	"log"
	"net/http"
)

type httpHandlerFunc func(w http.ResponseWriter, req *http.Request)
type jsonHttpBuilderFunc func(req *http.Request) interface{}

func JsonBuilder(req *http.Request) interface{} {
	jsonMap := make(map[string]interface{})
	name := req.URL.Query().Get(":name")
	jsonMap["message"] = "hello " + name
	return jsonMap
}

func JsonServer(builderFunc jsonHttpBuilderFunc) (hanlderFunc httpHandlerFunc) {
	hanlderFunc = func(w http.ResponseWriter, req *http.Request) {
		enc := json.NewEncoder(w)
		jsonMap := builderFunc(req)
		w.Header().Set("Content-Type", "application/json")
		if err := enc.Encode(&jsonMap); err != nil {
			log.Fatal(err)
		}
	}
	return
}

func main() {
	m := pat.New()
	m.Get("/api/hello/:name", http.HandlerFunc(JsonServer(JsonBuilder)))

	http.Handle("/", m)
	err := http.ListenAndServe(":3000", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```
