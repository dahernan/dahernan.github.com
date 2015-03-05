---
title: "Authentication library for Go net/http and Negroni"
date: "2015-03-05"
categories:
tags:
  - "golang"
---

Some weeks ago in the [Gophers Slack Community](http://blog.gopheracademy.com/gophers-slack-community/) somebody asked about how to do authentication, for a http server in Go.
I could make up a gist, based in a web application that I had from some months ago. But, I decided to put everything together and realease a simple library to do Authentication based in a configurable store.

I named [Auth](https://github.com/dahernan/auth), not very original name, but at least it shows the purpose.

Some of the features are:

* Authentication of the users via configurable store
* `net/http` compatible
* [Negroni](https://github.com/codegangsta/negroni) middleware
* Use of JSON Web Tokens (JWT) 
* JSON interface

The following example is using `net/http` to secure the path `/secure`

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"

	"github.com/boltdb/bolt"
	"github.com/dahernan/auth"
	"github.com/dahernan/auth/jwt"
	"github.com/dahernan/auth/store"
)

func main() {
	// Using BoltDB to store the users
	db, err := bolt.Open("usersdb", 0600, &bolt.Options{})
	if err != nil {
		log.Fatalln("Can not open the database", err)
	}
	defer db.Close()

	boltStore, err := store.NewBoltStore(db, "users")
	if err != nil {
		log.Fatalln("Can not create bolt store", err)
	}

	// check github.com/dgrijalva/jwt-go for the JWT options
	options := jwt.Options{
		SigningMethod: "RS256",
		PrivateKey:    Private, // $ openssl genrsa -out app.rsa keysize
		PublicKey:     Public,  // $ openssl rsa -in app.rsa -pubout > app.rsa.pub
		Expiration:    60 * time.Minute,
	}

	// creates the route with Bolt and JWT options
	authRoute := auth.NewAuthRoute(boltStore, options)

	http.HandleFunc("/login", authRoute.Login)
	http.HandleFunc("/signin", authRoute.Signin)

	// protects this handle 
	http.Handle("/secure", authRoute.AuthHandlerFunc(SecurePlace))
	
	http.ListenAndServe(":1212", nil)

}

func SecurePlace(w http.ResponseWriter, req *http.Request) {
	userId := auth.GetUserId(req)
	token := auth.GetToken(req)
	fmt.Fprintf(w, "Hey %v you have a token %v", userId, token)
}
```

To use it, you can do some http request to try it, for example:

### Signin

```
$ curl -XPOST "http://localhost:1212/signin" -d'
> {
>    "email": "dahernan@dahernan.com",
>    "password": "justTest123"
> }'

{"id":"dahernan@dahernan.com"}

```

### Login 
```
$ curl -XPOST "http://localhost:1212/login" -d'
> {
>    "email": "dahernan@dahernan.com",
>    "password": "justTest123"
> }'

{"token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0MjU0MDcyMTksImlhdCI6MTQyNTQwMzYxOSwic3ViIjoiZGFoZXJuYW5AZGFoZXJuYW4uY29tIn0.inRfTdlPZ4dKbHY2dHzTqmyBzLIW9_52oc8NFh_yfnrVEdCzfIAIHIVo_7cksUdUdQ4yMciy-JbuQc8hECx31a3RDNkH2iUSDaueZqM0nJaHsWdLAtO_8nz_zX6AqxPPP-cTb5Qjw3R8cmsrFTwpqTn7agDxgypn7NxFW67WIq_XcjH9Ev9VKFqC7AoV9wo2noX6l42JL_338UoNib3K--cUKTeJdCjygj-LH2TBobG9t3Wn55rFmr1oVfKMOTVx1eJpRl376tUemD-IXxCN0ZG788TLihLhXolbumnHzJ13AzriQEHTOc2GKtwT5M-DqEpj9lhV6uu6clPRFRs9O2PB82t4LkuhWra62-h9_R7fBaCN1-ni03RI-tXKUpWz1XFfcbHzCXzFOhl0fb_h_xpx-xEDKFbEE6JDQowxIIFNTIElv7kz0wke_QUddK1Lz--StMr2BS9Q3h3Xk1XC0dgHsSfZDvF-qbud-asUbaoaokNlsAm0kwUSMTJ_oBi1"}
```

### GET Secure url without Token

```
$ curl -XGET "http://localhost:1212/secure"

Error no token is provided
```

### GET Secure url with Token

```
$ curl -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0MjU0MDcyMTksImlhdCI6MTQyNTQwMzYxOSwic3ViIjoiZGFoZXJuYW5AZGFoZXJuYW4uY29tIn0.inRfTdlPZ4dKbHY2dHzTqmyBzLIW9_52oc8NFh_yfnrVEdCzfIAIHIVo_7cksUdUdQ4yMciy-JbuQc8hECx31a3RDNkH2iUSDaueZqM0nJaHsWdLAtO_8nz_zX6AqxPPP-cTb5Qjw3R8cmsrFTwpqTn7agDxgypn7NxFW67WIq_XcjH9Ev9VKFqC7AoV9wo2noX6l42JL_338UoNib3K--cUKTeJdCjygj-LH2TBobG9t3Wn55rFmr1oVfKMOTVx1eJpRl376tUemD-IXxCN0ZG788TLihLhXolbumnHzJ13AzriQEHTOc2GKtwT5M-DqEpj9lhV6uu6clPRFRs9O2PB82t4LkuhWra62-h9_R7fBaCN1-ni03RI-tXKUpWz1XFfcbHzCXzFOhl0fb_h_xpx-xEDKFbEE6JDQowxIIFNTIElv7kz0wke_QUddK1Lz--StMr2BS9Q3h3Xk1XC0dgHsSfZDvF-qbud-asUbaoaokNlsAm0kwUSMTJ_oBi1" -XGET "http://localhost:1212/secure"

Hey dahernan@dahernan.com you have a token
```

You can implement your own store, you only have to implement this interface:

```
type UserRepository interface {
	Signin(email, pass string) (string, error)
	Login(email, pass string) (string, error)
	UserByEmail(email string) (User, error)
}
```

I think the library is ideal to implement a little Authentication Microservice.
