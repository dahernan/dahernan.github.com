---
title: "Scrap the Web with Go"
date: "2014-12-03"
categories:
tags:
  - "golang"
---

As a part of a project that I've been involved, I released opensource a Scraper in Go that I called [gopherscraper](https://github.com/dahernan/gopherscraper) is a scraper to extract information of ecommerce sites, but you can extrapolate and extract information of any website with 'items', like news, videos, and so on.

The project comes with a Rest API that you can [check on Github](https://github.com/dahernan/gopherscraper) for more detail, and is based in CSS Selectors.
To store the items, I use Redis and ElasticSearch

Let me share a few insights in the development of the library:

* I'm using [Goquery](https://github.com/PuerkitoBio/goquery), to extract information based on CSS Selectors

* The interface for scrapping is really simple just one function

```go
type ScrapperItems interface {
	Scrap(selector ScrapSelector) (string, chan ItemResult, error)
}
```

* The interface to store the items that you scrap is also very simple, you can store the items in Redis, ElasticSearch or in local disk

```go
type StorageItems interface {
	StoreItem(it ItemResult)
}

// store items as a File
func (sto FileStorage) StoreItem(it ItemResult) {
	if it.Err != nil {
		return
	}
	WriteJsonToDisk(sto.baseDir, it.Item)
}
```

* Was easy to create a recursive Scraper, using composition, based in a normal scraper.

* I started with not concurrency code at all, and after make it work, I put the go routines and waitgroups to syncronize the completion of the scraper.

```go
func (d DefaultScrapper) Scrap(selector ScrapSelector) (string, chan ItemResult, error) {
	wg := &sync.WaitGroup{}
	err := validateSelector(selector)
	if err != nil {
		return "", nil, err
	}

	items := make(chan ItemResult, bufferItemsSize)

	jobId := "D" + GenerateStringKey(selector)
	pages := paginatedUrlSelector(selector)

	wg.Add(len(pages))
	for i, _ := range pages {
		go doScrapFromUrl(jobId, pages[i], items, wg)
	}

	go closeItemsChannel(jobId, items, wg)

	return jobId, items, err
}

func doScrapFromUrl(jobId string, s ScrapSelector, items chan ItemResult, wg *sync.WaitGroup) {
	defer wg.Done()
	doc, err := fromUrl(s)
	if err != nil {
		log.Printf("ERROR [%s] Scrapping %v with message %v", jobId, s.Url, err.Error())
		return
	}
	DocumentScrap(jobId, s, doc, items)
}

func closeItemsChannel(jobId string, items chan ItemResult, wg *sync.WaitGroup) {
	wg.Wait()
	close(items)
}
```

* I limited the number of concurrent connections with a buffered channel.

```go
func fromUrl(selector ScrapSelector) (*goquery.Document, error) {
	lockLimitConnections()
	defer unlockLimitConnections()

	req, err := http.NewRequest("GET", selector.Url, nil)
	if err != nil {
		return nil, err
	}

	req.Header.Add("User-Agent", defaultUserAgent)

	res, err := httpClient().Do(req)
	if err != nil {
		return nil, err
	}
	return goquery.NewDocumentFromResponse(res)
}

func UseMaxConnections(max int) {
	semaphoreMaxConnections = make(chan struct{}, max)
}

func lockLimitConnections() {
	semaphoreMaxConnections <- struct{}{}
}
func unlockLimitConnections() {
	<-semaphoreMaxConnections
} 
```

At the end It was a really fun, doing what it looks like a tedious job, to get a clean JSON document when there is not any API available to use.

