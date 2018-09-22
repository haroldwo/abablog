---
title: "Write a Simple Crawler."
date: 2018-09-20T12:00:00+08:00
weight: 10
keywords: ["crawler","goquery"]
description: "Write a Simple Crawler."
tags: ["crawler","goquery"]
categories: ["Dev"]
author: "Fred"
banner: ""
---

## 1. Basic process.

#### HTTP request -> Web -> HTTP response -> Parse data -> Handle data

This is the basic process of a crawler program. I will not talk about concurrency or distributed framework here. This article aim to give a simple introduction of crawler.

A crawler actually is a script which execute simulation of our action very fast when we browse a web page and it follows some rules we designed for catching our data.

## 2. Describe a crawler.

Let's describe the crawler as a object. What attribute can a crawler own? These attributes will be used as parameter in our program. Let me see. Well, First of all, I need a target URL for visit. Then, I need set my HTTP head when I send request to the website. Also, there could be more than one link when we search something like pictures so I need an array to store these links. Additionally, don't forget that the crawler will be blocked by a web server if it sends requests too frequently. We can set a optional interval mode for it. Ok, Let's code. Please follow this mind to create a stronger crawler if you want.

```
type Crawler struct {
	targetUrl   string
	headHost    string
	headReferer string
	dataDir     string
	Contents    []string
	Mode
}

type Mode struct {
	name       string
	interval   *time.Ticker
	startPoint time.Time
	end        time.Timer
	duration   time.Duration
}
```
In most of cases, we only need to change host and referer of HTTP head. So I just involved these two.

## 3. HTTP request -> Web

I use lib "github.com/PuerkitoBio/goquery" for convenience. You can use regexp instead of it for a more original way. Here, I wrote two methods.

First one, I just catch data from a page.
```
func (c Crawler) getDoc() (*goquery.Document, error) {
	res, err := http.Get(c.targetUrl)
	if err != nil {
		log.Fatal(err)
		return nil, err
	}
	defer res.Body.Close()
	if res.StatusCode != 200 {
		log.Fatalf("status code error: %d %s", res.StatusCode, res.Status)
		return nil, errors.New(res.Status)
	}
	doc, err := goquery.NewDocumentFromReader(res.Body)
	if err != nil {
		log.Fatal(err)
		return nil, err
	}
	return doc, nil
}
```

Second one, I send different requests for further links.
```
func (c Crawler) reqDoc() (*goquery.Document, error) {
	req, err := http.NewRequest("GET", c.targetUrl, nil)
	if err != nil {
		log.Fatal(err)
		return nil, err
	}
	header := map[string]string{
		"Host":                      c.headHost,
		"Connection":                "keep-alive",
		"Cache-Control":             "max-age=0",
		"Upgrade-Insecure-Requests": "1",
		"User-Agent":                "Mozilla/5.0 (X11; Ubuntu; Linu…) Gecko/20100101 Firefox/62.0",
		"Accept":                    "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
		"Referer":                   c.headReferer,
	}
	for key, value := range header {
		req.Header.Add(key, value)
	}
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
		return nil, err
	}
	defer resp.Body.Close()
	doc, err := goquery.NewDocumentFromReader(resp.Body)
	if err != nil {
		log.Fatal(err)
		return nil, err
	}
	return doc, nil
}
```

## 4. HTTP response -> Parse data

First, This is a convenient function you may use for remove duplicate data from a array such as urls. Just FYI.
```
func Duplicate(dirty interface{}) (clean []interface{}) {
	va := reflect.ValueOf(dirty)
	for i := 0; i < va.Len(); i++ {
		if i > 0 && reflect.DeepEqual(va.Index(i-1).Interface(), va.Index(i).Interface()) {
			continue
		}
		clean = append(clean, va.Index(i).Interface())
	}
	return clean
}
```

Then, Let's write a method that filter some info from the response.
```
func (c *Crawler) getContent() error {
	doc, err := c.reqDoc()
	if err != nil {
		log.Fatal(err)
		return err
	}
	doc.Find(".content").Each(func(i int, s *goquery.Selection) {
		content := s.Find("a").Text()
		c.Contents = append(c.Contents, content)
	})
	return nil
}
```

This is another example for filtering picture info.
```
func (c *Crawler) getPic() error {
	doc, err := c.reqDoc()
	if err != nil {
		log.Fatal(err)
		return err
	}
	doc.Find("img").Each(func(i int, s *goquery.Selection) {
		link, ok := s.Attr("src")
		if ok {
			c.Contents = append(c.Contents, link)
		} else {
			fmt.Println("Address not found.")
		}
	})
	return nil
}
```

## 5. Parse data -> Handle data

Now, it's the turn we dealing with data.

In this example, I create a file and then write the data I got from the website.
```
func (c Crawler) handleContent() error {
	fileName := filepath.Base(c.targetUrl)
	fullPath := filepath.Join(c.dataDir, fileName)
	file, err := os.Create(fullPath)
	if err != nil {
		log.Panic("Can not create file.")
		return err
	}
	for _, content := range Duplicate(c.contents) {
		file.WriteString(content.(string))
	}
	return nil
}
```

In this one, I download the pictures which I got info of from the website.
```
func (c Crawler) handlePic() {
	for _, link := range c.contents {
		resp, err := http.Get(link)
		if err != nil {
			log.Fatal(err)
		}
		defer resp.Body.Close()
		if resp.StatusCode != http.StatusOK {
			log.Fatalf("status code error: %d %s", resp.StatusCode, resp.Status)
		}
		FileName := filepath.Base(link)
		FullName := filepath.Join(c.dataDir, FileName)
		file, err := os.Create(FullName)
		if err != nil {
			log.Panic("Can not create file.")
		}
		io.Copy(file, resp.Body)
	}
}
```

## 6. Continuation.

Now, we can write down our main function. Please try and remember to use timer to limit request. Crawler is very interesting and always used as important part in our application such as Big Data and AI. If you are a gopher, Let me introduce some projects for you.

1. [goquery](https://github.com/PuerkitoBio/goquery) -- a helpful lib for crawler.
2. [gocrawl](https://github.com/PuerkitoBio/gocrawl) -- a light concurrent web crawler.
3. [pholcus](https://github.com/henrylee2cn/pholcus) -- a heavy concurrent web crawler with complete distributed architecture.
4. [colly](http://go-colly.org/docs/introduction/start/) -- the most popular crawler framework in Go.
