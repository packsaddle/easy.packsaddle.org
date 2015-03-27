---
layout: post
title: "Ruby version policy with Travis-CI"
excerpt: >-
  How to treat Ruby version policy in Travis-CI.
categories: articles
tags: [travis-ci, ruby, en]
author: sanemat
comments: true
share: true
---

Ruby uses semver after `2.1.0`.

{% highlight yaml %}
# .travis.yml
- 1.9.2
- 1.9.3
- 2.0.0
- 2.1
- 2.2
{% endhighlight %}

So this means `1.9.2-p330`(latest), `1.9.3-p551`(latest), `2.0.0-p643`(latest), `2.1.5`(latest), `2.2.1`(latest).
But `2.1.0` means *exactly* `2.1.0`.

They follows Travis-CI's latest precompiled one.

* [Ruby version policy changes starting with Ruby 2.1.0](https://www.ruby-lang.org/en/news/2013/12/21/ruby-version-policy-changes-with-2-1-0/)
* [ruby-build's targets](https://github.com/sstephenson/ruby-build/tree/a55247a8c5eacc485ed1ce8bc0129376d6b00298/share/ruby-build)
* [Travis CI: Precompiled Ruby Versions](http://rubies.travis-ci.org/)
