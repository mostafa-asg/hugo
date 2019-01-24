---
title: "Load testing using K6"
date: 2019-01-24T10:41:16+03:30
draft: false
---
# Introduction
Understanding the performance of mission critical API is crucial. Before you deploy your API, probably you want to know, how your API behaves under heavy loads. Most API nowadays are HTTP-based, and that's why there are numerous **HTTP load testing** tools available. Here is just a few:

* [wrk](https://github.com/wg/wrk)
* [wrk2](https://github.com/giltene/wrk2)
* [vegeta](https://github.com/tsenart/vegeta) 
* [locust](https://github.com/locustio/locust)
* [hey](https://github.com/rakyll/hey)
* [Apache Benchmark](https://httpd.apache.org/docs/2.4/programs/ab.html)

## Problem
Most of them designed to generate load on the static url, like:
```
wrk -t12 -c400 -d30s http://test.loadimpact.com
```
But my API is some kind of file storage. Users can call the API to store their file and if they send the same request again, they will get **File already exist** error. I solely wanted to test the performance of **write path** so I needed a mechanism to change the path each time. Although some of the tools that I mentioned above, has some scripting capabilities, I found all of them very inconvenient or very hard to implement.

## Solution
Luky enough, I found [K6](https://k6.io). [K6](https://k6.io) is a developer centric open source load testing tool for testing the performance of your backend infrastructure. It’s built with Go and JavaScript to integrate well into your development workflow, so you can stay on top of performance without fuzz.
The nice thing about [K6](https://k6.io) is that, you define the logic in Javascript and [K6](https://k6.io) will handle the rest of the work. Here how it works:
```
import http from "k6/http";

export default function() {
  http.get("http://test.loadimpact.com");
};
```
Save the above script (for example `script.js`), then run using K6:
```
k6 run --vus 2 --duration 5s script.js 
```
So far so good. Now how can I change the path? I used **Execution context variables** inside
Javascript to construct a unique filename in each iteration:
```
import http from "k6/http";

export default function() {
  const filename = `file_${__VU}_${__ITER}`;
  http.put("http://localhost:8585/folderA/" + filename);
};

```
For understanding the meaning of **__VU** and **__ITER**,
please read the [documentation](https://docs.k6.io/docs/execution-context-variables).

## Sending file content
Now that I uniquely created the url path, I needed a way to send the actual file content. Fortunately, [K6](https://k6.io) has provided a way. In [init context](https://docs.k6.io/docs/init-context) we can open and read a file's content, and then we can send the file's content in each iteration. Note that reading file's content will be happen only once:
```
import http from "k6/http";

const fileContent = open("./my-file")

export default function() {
  const filename = `file_${__VU}_${__ITER}`;
  http.put("http://localhost:8585/folderA/" + filename, fileContent);
};
``` 
Thanks to [K6](https://k6.io), I easily managed to benchmark my HTTP API without hassle.
