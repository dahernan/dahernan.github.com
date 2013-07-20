---
layout: post
tags : [golang] [json] [rest]
---


'''
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
'''
