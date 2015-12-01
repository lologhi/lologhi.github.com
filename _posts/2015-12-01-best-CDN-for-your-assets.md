---
layout:     post
title:      What is the best CDN for your assets?
date:       2015-12-01
summary:    Based on the idea that loading assets that are already cached on user browser is the fastest, let's have a look on switching from your localy-Grunt-generated-minified-uglified-jQuery-Bootstrap file to multiple separated request to popular CDN.
categories: performance
tags: assets
---

Based on the idea that loading assets that are already cached on user browser is the fastest, we, at [Villa-Finder.com](http://www.villa-finder.com), switched from our localy-Grunt-generated-minified-uglified-jQuery-Bootstrap ("lGgmujB"?) file to multiple separated request to popular CDN.

TL;DR: Just **don't use MaxCDN** (the default one for jQuery and Bootstrap), they're super slow compared to other one. As you want to only use one CDN, CloudFlare is the one that offer more choice for the best performances.
Is it better to keep it's own compressed version of assets on CloudFront ? How could I check if my visitors already know the files I tell them to GET from CDN ? I don't know !

--------

After looking around, I've decided the largest CDN available would be:

- [Google Hosted Libraries](https://developers.google.com/speed/libraries/)
- [Microsoft Ajax Content Delivery Network](http://www.asp.net/ajax/cdn)
- [CloudFlare/cdnjs](https://cdnjs.com/)
- [MaxCDN](https://www.maxcdn.com/blog/free-open-source-cdns/)

And that I should test them.

# Huge differences on local tests

## Let's start with web inspector

After a `Shift +  + R`, here are the results (tested with 4G in a Singaporean bus) for some libraries: (Transfered and Size are in KB, Tests are in ms, tests 1 & 2 are fresh load)

| jQuery                                                                         | Transfered | Size  | Test 1 | Test 2 | Test 3 | Test 4 |
|---------------------------------------------------------------------------------|:----------:|:-----:|:------:|:------:|:------:|:------:|
| [Microsoft](https://ajax.aspnetcdn.com/ajax/jQuery/jquery-2.1.4.min.js)         | 36.87      | 82.40 | 99     | 67     | 55     | 33     |
| [Cloudflare](https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.min.js) | 29.11      | 82.40 | 45     | 188    | 28     | 27     |
| [Google](https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js)      | 28.86      | 82.40 | 235    | 243    | 30     | 85     |
| [MaxCDN](https://code.jquery.com/jquery-2.1.4.min.js)                           | 33.61      | 82.37 | **630**|**708** | **216**| **201**|


| Bootstrap css                                                                                      | Transfered | Size   | Test 1 | Test 2 | Test 3 | Test 4 |
|----------------------------------------------------------------------------------------------------|:----------:|:------:|:------:|:------:|:------:|:------:|
| [Microsoft](https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.5/css/bootstrap.min.css)                 | 19.74      | 119.67 | 181    | 190    | 123    | 119    |
| [Cloudflare](https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.5/css/bootstrap.min.css) | 27.17      | 119.67 | 132    | 68     | 11     | 33     |
| [MaxCDN](https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css)                    | 24.74      | 119.67 | **694**| **750**| **657**| **683**|

It's really bad for MaxCDN! In fact a simple `ping` show us the problem.

## Something's wrong even on a ping

	ping maxcdn.bootstrapcdn.com
	--- bootstrapcdn.jdorfman.netdna-cdn.com ping statistics ---
	6 packets transmitted, 6 packets received, 0.0% packet loss
	round-trip min/avg/max/stddev = 236.934/266.647/338.548/35.002 ms

	ping cdnjs.cloudflare.com
	--- cdnjs.cloudflare.com ping statistics ---
	6 packets transmitted, 6 packets received, 0.0% packet loss
	round-trip min/avg/max/stddev = 8.054/13.970/17.638/3.753 ms

	ping ajax.aspnetcdn.com
	--- cs1.wpc.v0cdn.net ping statistics ---
	6 packets transmitted, 6 packets received, 0.0% packet loss
	round-trip min/avg/max/stddev = 48.221/56.626/64.773/5.831 ms


## More hits with ApacheBench

And I've been nice with only 20 hits in one minute, *i.e.* `ab -S -n 20 -c 1` :

| Server                          | [MaxCDN](maxcdn.bootstrapcdn.com) | [Cloudflare](cdnjs.cloudflare.com) | [Microsoft](ajax.aspnetcdn.com) |
|---------------------------------|----------------------------------:|-----------------------------------:|--------------------------------:|
| Server Software                 | NetDNA-cache/2.2                  | cloudflare-nginx                   | ECAcc                           |
| Time taken for tests in seconds | **33.6**                          | 2.2                                | 5.6                             |
| Requests per second             | **0.59**                          | 8.91                               | 3.58                            |
| Time per request in ms          | **1 682**                         | 112                                | 279                             |
| Transfer rate [Kbytes/sec]      | **71**                            | 1 068                              | 430                             |


# Using [WebPageTest](http://webpagetest.org) to check elsewhere

And we discover here that CloudFlare is the only one that makes only one SSL Negotiation.

Sorry, I didn't found an easy way to present these data ¯\\_(ツ)_/¯

At least, I reduced the filenames :

  - Microsoft: 
    - MS1: `jquery-2.1.4.min.js`
    - MS2: `bootstrap.min.js`
    - MS3: `bootstrap.min.css`
  - MaxCDN
    - M1: `jquery-2.1.4.min.js`
    - M2: `bootstrap.min.js`
    - M3: `bootstrap.min.css`
    - M4: `font-awesome.min.css`
  - Cloudflare
    - C1: `jquery.min.js`
    - C2: `bootstrap.min.js`
    - C3: `bootstrap.min.css`
    - C4: `font-awesome.min.css`
  - Google
    - G1: `jquery.min.js`

| Paris                 | Duration | DNS Lookup | Connection | SSL Negotiation (ms) | TTFB     | Download | Weight |
|-----------------------| -------- |------------|------------|----------------------| -------- |----------|--------|
| MS1                   | 545      | 190        | 34         | 136                  | 124      | 61       | 45.2   |
| MS2                   | 628      | 249        | 31         | 139                  | 188      | 21       | 17.1   |
| MS3                   | 491      | 199        | 32         | 137                  | 78       | 45       | 83.4   |
| M1                    | 482      | 0          | 172        | 98                   | 179      | 33       | 52.5   |
| M2                    | **5561** | 305        | 37         | 93                   | **5125** | 1        | 12     |
| M3                    | 813      | 431        | 35         | 90                   | 226      | 31       | 69     |
| M4                    | 713      | 416        | 37         | 88                   | 170      | 2        | 7.2    |
| C1                    | 643      | -          | -          | -                    | **444**  | 199      | 60.1   |
| C2                    | 642      | -          | -          | -                    | **640**  | 2        | 14     |
| C3                    | 1261     | 584        | 88         | 147                  | 132      | 310      | 20.2   |
| C4                    | 445      | -          | -          | -                    | **444**  | 1        | 6.5    |
| G1                    | **1917** | 1470       | 109        | 173                  | 90       | 75       | 45.8   |

| San Francisco         | Duration | DNS Lookup | Connection | SSL Negotiation (ms) | TTFB | Download | Weight |
|-----------------------| -------- |------------|------------|----------------------|------| -------- |--------|
| MS1                   | 729      | 32         | 29         | 66                   | 122  | 480      | 28     |
| MS2                   | 352      | -          | 52         | 155                  | 44   | 101      | 37.9   |
| MS3                   | **1192** | -          | -          | -                    | 101  | **1091** | 36.6   |
| M1                    | **1163** | 57         | 141        | 63                   | 133  | **769**  | 33.6   |
| M2                    | 435      | -          | -          | -                    | 434  | 1        | 12.1   |
| M3                    | 467      | -          | 31         | 68                   | 367  | 1        | 7.5    |
| M4                    | 521      | 37         | 31         | 65                   | 375  | 13       | 25.6   |
| C1                    | 472      | -          | -          | -                    | 469  | 3        | 0.4    |
| C2                    | 653      | -          | -          | -                    | 630  | 23       | 0.4    |
| C3                    | 617      | -          | -          | -                    | 551  | 66       | 0.4    |
| C4                    | 624      | 35         | 29         | 90                   | 251  | 219      | 19.7   |
| G1                    | **1399** | 56         | 164        | 166                  | 120  | **893**  | 28.9   |

| Singapore             | Duration | DNS Lookup | Connection | SSL Negotiation (ms) | TTFB | Download | Weight |
|-----------------------| -------- |------------|------------|----------------------|------|----------|--------|
| MS1                   | 722      | 277        | 32         | 286                  | 86   | 41       | 28     |
| MS2                   | 983      | -          | 32         | 284                  | 127  | 540      | 37.8   |
| MS3                   | 904      | -          | 33         | 284                  | 161  | 426      | 13.5   |
| M1                    | **2725** | 73         | 247        | 334                  | 287  | 1784     | 33.6   |
| M2                    | 826      | -          | 248        | 312                  | 264  | 2        | 12.3   |
| M3                    | 807      | -          | 247        | 297                  | 261  | 2        | 7.5    |
| M4                    | **1048** | 266        | 247        | 292                  | 229  | 14       | 25.6   |
| C1                    | 489      | -          | -          | -                    | 341  | 148      | 0.4    |
| C2                    | 503      | -          | -          | -                    | 495  | 8        | 0.4    |
| C3                    | 297      | -          | -          | -                    | 290  | 7        | 0.4    |
| C4                    | 744      | 279        | 31         | 184                  | 97   | 153      | 19.7   |
| G1                    | 459      | 71         | 35         | 256                  | 50   | 47       | 28.9   |
|                       |          |            |            |                      |      |          |        |