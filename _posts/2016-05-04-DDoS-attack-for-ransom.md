---
layout:     post
title:      My first DDoS attack for a $200 ransom
date:       2016-04-04
summary:    Last week-end could have been a really nice week-end with some ex-colleagues and friends, in Bali. Instead, some jerk choose to DDoS us a Saturday afternoon at 4pm. Hackers, please respect my days-off!
categories: symfony2
tags: assets security DDoS cache
---

**2016-05-06 Edit:** Thank you [Hacker News](https://news.ycombinator.com/item?id=11637602) for the visits and comments! Here are some missing informations in my story:

- I'm the only full time developer working on [Villa-Finder](http://www.villa-finder.com) websites : [Villa-Bali.com](https://www.villa-bali.com), [Villa-Phuket.com](https://www.villa-phuket.com), [SriLanka-Villa.com](https://www.srilanka-villa.com)... 🏖
- It's the first time I'm in charge, I had to recode lots of old pieces of code, to add features to please my colleagues. That's my explanation for missing cache and poor performances. (I just feel it's the real life, not the fancy Valley startups who will pretend they're always state of the art on all levels.)
- It's a small DDoS attack but it was my first one, and I previously only read about huge ones mitigated by large companies.
- We're on AWS, it's a Symfony2 stack, with a MySQL database on RDS.
- I'm French and try to write in English, sorry for this 🙈
- I'm just a noob, but I'm working on it 🤘 

![6k visits, and only 33 in karma, are you kidding?)](/images/what-a-nice-surprise.png)

-------------

So while we were visiting the really nice [Uluwatu temple](https://en.wikipedia.org/wiki/Uluwatu_Temple) with two friends/ex-colleagues, I learned, thanks to [Uptime Robot](http://uptimerobot.com) and some Slack messages from my colleagues that our website had started to be sluggish/buggy/inaccessible.

After a lazy-shameless `sudo service php5-fpm restart` and seeing with `htop` that our EC2 m4.xlarge instance was still 100% on all 4 cores, it started to really look like an attack. Thanks to Fail2Ban, `iptables` and our Symfony2 refreshed code, I was feeling quite secured against SSH and SQL injections. But downtime is nether good for business, it was time to find my Mac, as [Serverauditor](https://serverauditor.com) iOS app is quite light to really investigate (but could have been enough as [spoiler] our quick-fix were quite easy to install).

# Let’s investigate

## Make the Apache2 logs readable behind an AWS ELB

We first discovered (with [@remiii](http://github.com/remiii) who was visiting me) that my Apache2 logs where always giving us the same IP address : the ones of the AWS ELB (Elastic Load Balancer). To fix this, I had to update my `vHost` configuration to change the IP to the one sent by the ELB in the `X-Forwarded-For` header :

```ApacheConf
LogFormat "\"%{X-Forwarded-For}i\" %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"” combined_new
...
ErrorLog ${APACHE_LOG_DIR}/admin_error.log
CustomLog ${APACHE_LOG_DIR}/admin_access.log combined_new
```

We were now happy to have these nice and readable logs (this is just one sec 😥):

```
"185.28.193.95" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78630 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)"
"115.111.179.133" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78629 "-" "Mozilla/5.0 (Windows NT 5.1; rv:13.0) Gecko/20100101 Firefox/13.0.1"
"186.203.134.5" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78638 "-" "Mozilla/5.0 (Windows NT 6.1; rv:12.0) Gecko/20100101 Firefox/12.0"
"79.120.72.222" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78666 "-" "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322)"
"52.53.237.11" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78629 "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)"
"52.53.237.11" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78638 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.47 Safari/536.11"
"201.245.205.229" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78453 "-" "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322)"
"195.229.210.162" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78453 "-" "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)"
"187.44.1.167" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78666 "-" "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:13.0) Gecko/20100101 Firefox/13.0.1"
"79.120.72.222" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78629 "-" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.47 Safari/536.11"
"177.22.143.0" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78630 "-" "Mozilla/5.0 (iPad; CPU OS 5_1_1 like Mac OS X) AppleWebKit/534.46 (KHTML, like Gecko) Version/5.1 Mobile/9B206 Safari/7534.48.3"
"202.21.116.13" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78630 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_4) AppleWebKit/536.5 (KHTML, like Gecko) Chrome/19.0.1084.56 Safari/536.5"
```

And we can see that the fancy tool our hacker is using is able, for a same IP, to spoof its `user-agent` between each hit (scroll on the right):

```
"185.28.193.95" - - [30/Apr/2016:21:59:56 +0800] "GET / HTTP/1.1" 200 78630 "-" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)"
"185.28.193.95" - - [30/Apr/2016:21:59:57 +0800] "GET / HTTP/1.1" 200 78453 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.47 Safari/536.11"
"185.28.193.95" - - [30/Apr/2016:21:59:57 +0800] "GET / HTTP/1.1" 200 78453 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.6; rv:13.0) Gecko/20100101 Firefox/13.0.1"
"185.28.193.95" - - [30/Apr/2016:21:59:57 +0800] "GET / HTTP/1.1" 200 78638 "-" "Mozilla/5.0 (Windows NT 5.1; rv:12.0) Gecko/20100101 Firefox/12.0"
"185.28.193.95" - - [30/Apr/2016:21:59:58 +0800] "GET / HTTP/1.1" 200 78638 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) ; .NET CLR 3.5.30729)"
"185.28.193.95" - - [30/Apr/2016:21:59:58 +0800] "GET / HTTP/1.1" 200 78651 "-" "Mozilla/5.0 (Windows NT 6.1; rv:13.0) Gecko/20100101 Firefox/13.0.1"
"185.28.193.95" - - [30/Apr/2016:21:59:58 +0800] "GET / HTTP/1.1" 200 78666 "-" "Mozilla/5.0 (Windows NT 6.1; rv:5.0) Gecko/20100101 Firefox/5.02"
"185.28.193.95" - - [30/Apr/2016:21:59:58 +0800] "GET / HTTP/1.1" 200 78630 "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; MRA 5.8 (build 4157); .NET CLR 2.0.50727; AskTbPTV/5.11.3.15590)"
```

## Scale of the attack, or is-it a DDoS?

**If the attack is not distributed, a single rule on `iptables` could shut down the attack in 2 sec...**

We were now able to look at the logs and check the distribution of the IPs. I took the last 10000 lines of the access log, put them in a Google Sheets, kept only the IPs, used the `UNIQUE()` fonction to get distincts ones, and then a `COUNTIF()` function to get hits/IP.

![It seemed to me easier than some grep pipe sed in command line #noob)](/images/Thank-you-Excel.png)

## What are the attacked pages?

By looking at Apache2 logs, we easily found that all requests were GET on the homepage.

# Let's fix this

## `iptables`, catch 'em all!

![When I'm adding iptables rule)](/images/catch-them-all.gif)

Based on our Google Sheets of cleaned IP, we first thought the attack might not be distributed enough and we could just block the most harmful IPs. We added some `iptables -A INPUT -s xx.xx.xx.xx -j DROP` on what seemed to be the most harmful IPs, but it did not lower the attack. And it also broke the website as some of these IPs where AWS internal ones 🙃

**2016-05-06 Edit:** this might have been fully useless, as we did not specified that the real IP to block was in `X-Forwarded-For`!!

**First conclusion: it's a distributed attack, we cannot just block IPs.**

## Look how big is my instance

We first tried to switch from my EC2 m4.xlarge to a m4.10xlarge. As our websites were already down, it was quite easy to shutdown the instance, switch to the largest available, and start again. Sadly, it was 100% useless 😁

![40 cores, but still unable to process 10 requests/sec)](/images/M4-10xlarge.png)

## Smarter (also, working) solution : caching

After this first failure, we spent a few minutes in the villa pool, and thought that as the attack was only on the homepage, we could just serve a cached version of this one page.

In my `HomeController.php`, I just had to add this annotation before my action :

```php
/**
 * @Cache(expires="tomorrow", public=true)
 */
```

And to modify the `web/app.php` as explained in [The Book](http://symfony.com/doc/current/book/http_cache.html).

🏅🏆

Bingo! Huge drop in CPU usage, but huge increase in bandwidth: we went from 30Mb/sec to nearly 130Mb/sec, as the instance was now able to serve the homepage to everyone.
And we can now see that the attack ratio is quite huge. As the ~10 req/sec were spotted at 1.6Mb/sec incoming, **we were sending 80 time** what we were receiving. The bandwidth bill might be expensive this month.

![Before the first orange bar, it's the taxi back to reach my laptop.)](/images/Attack-graph.png)

## Future solution

Hopefully, our opponent did not really tried for long to hack us, so the cache solution was enough. We could have played hide and seek on all the website pages : him starting to attack, me adding some cache annotation.

A more viable solution will be to progressively add HTTP cache on all our pages, and also to add CloudFlare so that requests are filtered by them before hitting our servers.

# After words

Our attacker ransom message was stuck in our inaccessible CRM, so we only found it after we "fixed" his attack 🙃 We got its IP, but it's a Tor node.

![Sending your ransom message in the system you're currently hacking is not logic.)](/images/Hello-Hacker.png)

The day after, Sunday, our friend retried, but this time I had my laptop with me. CPU peak was the time for me to re-add the caching lines (that I had removed because of weird side effects)

![Guess who'd back?)](/images/Second-attack.jpg)
