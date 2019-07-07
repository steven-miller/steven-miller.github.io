---
layout: post
title: "Getting Response Headers with the Javascript Fetch API"
comments: true
description: "It's not crystal clear how to access response headers using the fetch API, but it is doable with the entries() method and some looping"
keywords: "javascript, fetch, json-server, response"
---

I've Googled-skimmed-copied plenty of answers in my day, so I'm going to start this post with the tl;dr - if you're here for the fix, just read the first part. If you're curious as to more of what's going on (use cases, methods, etc), read further!

## Getting the headers via method `entries()`

The headers are hidden in an `entries()` method that doesn't return an object, but an iterator, that then gives access to the headers in a key / value format through a `for...of` loop. ([More on this at MDN](https://developer.mozilla.org/en-US/docs/Web/API/Headers/entries))

For example, if you `fetch` my site and log out the loop, this is what you'll get:

```javascript
// code
fetch('https://stevemiller.dev')
  .then((response) => {
    for (var pair of response.headers.entries()) {
      console.log(pair[0]+ ': '+ pair[1]);
    }
  });
// logs
cache-control: max-age=600
content-type: text/html; charset=utf-8
expires: Sun, 07 Jul 2019 03:54:43 GMT
last-modified: Sun, 07 Jul 2019 03:43:47 GMT
```

***

## More info: Response Header Uses

A number of RESTful APIs use response headers to indicate important information after a call. For example, some third-party APIs will include current rate limit information in custom headers.

In my case, I was using the handy [json-server](https://www.npmjs.com/package/json-server) package for some test data.

When running a query on the json-server dataset, it returns a header with the total number of results for that query to help with paginating (`x-total-count`).

## Javascript Fetch API

Using a React front-end, I was calling the json-server API using the built-in [fetch functionality](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) with Javascript:

```javascript
fetch(`/events?q=${query}&_page=${page}`)
  .then((response) => {
    return response.json();
  })
  // then setting state, etc
```

(I love `fetch` because it's built into Javascript, so no need for an extra package)

But, to help with pagination, I needed to know exactly how many "pages" worth of content this query would provide. Accessing the headers like this logged an empty object:

```javascript
console.log(reponse.headers);
// ouptu: Headers {}
```

This is because the headers are _not_ an object, they're an `iterator`

## Accessing the headers using `for...of` and `entries()`

You can't directly access the headers on the response to a `fetch` call -- you have to iterate through after using the `entries()` method on the headers. Code sample (using react and looking for a query count) below:

```javascript
fetch(`/events?q=${query}&_page=${page}`)
  .then((response) => {
    for (var pair of response.headers.entries()) { // accessing the entries
      if (pair[0] === 'x-total-count') { // key I'm looking for in this instance
        this.setState({
          total: pair[1] // saving that value where I can use it
        })
      }
    }
    return response.json();
  });
```

From here, you could construct your own object via the loop, or filter for specific values (like I did at the top of the post).

***

_Looking for a Javascript developer? Check out my [portfolio](/portfolio) if you're intrigued or start the conversation by reaching out to steve \[at\] stevemiller.dev_