---
layout: post
title: "RestoreBundledWithを使ってBundlerのBUNDLED WITHをうまく取り扱う"
excerpt: >-
  tl;dr Bundler v1.10.0からBUNDLED WITHのsectionが導入された。
  RestoreBundledWithを使うことで、Bundler開発チームの精神を逸脱して(!)、
  異なるversionのBundlerとBUNDLED WITHをうまく取り扱う。
categories: articles
tags: [restore-bundled-with, bundler, ja]
author: sanemat
comments: true
share: true
image:
  thumb: 2015-06-21-bundled-with.png
---

## tl;dr

* Bundler v1.10.0からBUNDLED WITHのsectionが導入された。
* RestoreBundledWithを使うことで、Bundler開発チームの精神を逸脱して(!)、異なるversionのBundlerとBUNDLED WITHをうまく取り扱う。

<figure>
  <img src="/images/2015-06-21-bundled-with.png" alt="BUNDLED WITH section">
  <figcaption>BUNDLED WITHのsectionで差分が出る</figcaption>
</figure>

## Bundler v1.10 とBUNDLED WITH

Bundler v1.10.0から`BUNDLED WITH`というsectionに`bundle update`を実行したBundlerのversionを
記録するようになった。
日本語詳細はこちらが詳しい。
[Ruby - BUNDLED WITH で Gemfile.lock が更新されてしまう件 - Qiita](http://qiita.com/suu_g/items/2b1630b8015d51c5292e)

## 挙動の確認

Bundler v1.10.2で`bundle update`して、BUNDLED WITHにversionを記録しておく。

```text
$ cat Gemfile.lock
GEM                                                                                                                              
  remote: https://rubygems.org/                                                                                                  
  specs:                                                                                                                         
    actionmailer (4.1.11)                                                                                                        
      actionpack (= 4.1.11)
(snip)
  unicorn-worker-killer
  webmock

BUNDLED WITH
   1.10.2
```

### Bundler v1.9.9で bundle update

BUNDLED WITHのsectionごと消える。悲しい。

```text
$ bundle update
(update)

$ git diff
(snip)
@@ -289,6 +290,3 @@ DEPENDENCIES
   unicorn
   unicorn-worker-killer
   webmock
-
-BUNDLED WITH
-   1.10.2
```

### Bundler v1.10.4(新しい)で bundle update

BUNDLED WITHの記録を更新する。

```text
$ bundle update
(update)

$ git diff
(snip)
@@ -291,4 +292,4 @@ DEPENDENCIES
     webmock
  
   BUNDLED WITH
  -   1.10.2
  +   1.10.4
```

### Bundler v1.10.1(古い)で bundle update

Warning出しつつ、BUNDLED WITHは更新しない。

```text
$ bundle update
Warning: the running version of Bundler is older than the version that created the lockfile.
We suggest you upgrade to the latest version of Bundler by running `gem install bundler`.
(snip)

$ git diff
(no diff)
```

## 現実解?

## まとめ

[詰まったら聞いて](https://github.com/packsaddle/ruby-restore_bundled_with/issues/new)ください。このサイトやREADMEなどに反映します。
[ruby-restore_bundled_withにスター](https://github.com/packsaddle/ruby-restore_bundled_with)ください!

----

## 参照

* [ruby-restore_bundled_withにスター](https://github.com/packsaddle/ruby-restore_bundled_with)
* [Ruby - BUNDLED WITH で Gemfile.lock が更新されてしまう件 - Qiita](http://qiita.com/suu_g/items/2b1630b8015d51c5292e)
* [Track Bundler version in lockfile by smlance #3485 bundler/bundler](https://github.com/bundler/bundler/pull/3485)
* [Do not update BUNDLED WITH if no other changes to the lock happened #3697 bundler/bundler](https://github.com/bundler/bundler/issues/3697)
