---
layout: post
title: "Httpsで簡単に、ログ安全に、git pushする Git HTTPSable Push"
excerpt: >-
  tl;dr Git HTTPSable Pushを使うことで、Httpsで簡単に、ログ安全に、git push出来る。
  git:// や git@ でcloneしていてもhttpsで、設定なしにgit push origin branchできる。
  logやerrorのmessageでは、tokenやパスワードの出てくる部分を確実にマスクして、ログに出ても安全。
categories: articles
tags: [git, https, push, git_httpsable-push, overview, ja]
author: sanemat
comments: true
share: true
image:
  thumb: 2015-03-06-git-httpsable-push-overview.png
---

## tl;dr

* Git HTTPSable Pushを使うことで、Httpsで簡単に、ログ安全に、git pushする。

<figure>
  <img src="/images/2015-03-06-git-httpsable-push-overview.png" alt="Git HTTPSable Push">
  <figcaption>git-httpsable-pushの実行結果イメージ</figcaption>
</figure>

## 使い道

1. TravisCIや、CircleCIからGitHubにPushしたい
4. Git HTTPSable Push を使い`git httpsable-push` で簡単に、ログ安全に、httpsでのgit pushが出来る

## ログ安全とは?

{% highlight bash %}

$ git push --quiet "https://${GITHUB_ACCESS_TOKEN}@github.com/<repos_name>.git" gh-pages 2> /dev/null

{% endhighlight %}

と書けば、TravisCIやCircleCIから安全にpushできる。
でも、`--quiet`や`2> /dev/null`のオプションを書き忘れるまたは書き損じると、tokenがログの中に漏れてしまう。
いちいち書くの大変。
さらに、TravisCIやCircleCIはどうしてもログ経由のデバッグになるので、ログが全く出せなくなると苦しい。

## 使い方

{% highlight bash %}

gem install git_httpsable-push
git httpsable-push origin YOUR_BRANCH

{% endhighlight %}

gitリポジトリの設定から、`https://${GITHUB_ACCESS_TOKEN}@github.com/<repos_name>.git` をコマンド内部でいい感じに作る。
このため、リポジトリが`git://...`でも`git@...`でも、GITHUB_ACCESS_TOKEN以外の設定は不要。
たとえば、`git httpsable-push origin gh-pages`と書けばOK。
ログとエラーの中の該当部分をマスクしているので、ログに出ても安全。
エラーが出てもエラーを見ながら直せる。
安心して使える。

## まとめ

* Httpsで簡単に、ログ安全に、git push出来る。HTTPSable Push。

使ってみたをブログやqiitaに書いてもらえると助かる！フィードバックください！！
masterにmergeしたらgh-pagesに自動でpush、が典型的に使える例。
そのほか、bundle updateしてgit pushしてpull requestなど、TravisCI, CircleCIで何か処理をしたい時に便利。

[詰まったら聞いて](https://github.com/packsaddle/ruby-git_httpsable-push/issues/new)ください。このサイトやREADMEなどに反映します。
[ruby-git_httpsable-pushにスター](https://github.com/packsaddle/ruby-git_httpsable-push)ください!

----

## 参照

* [ruby-git_httpsable-pushにスター](https://github.com/packsaddle/ruby-git_httpsable-push)
* [Middleman で作った web サイトを Travis + GitHub pages でお手軽に運用する - tricknotesのぼうけんのしょ](http://tricknotes.hateblo.jp/entry/2013/06/17/020229)
* [class Logger](http://docs.ruby-lang.org/ja/2.2.0/class/Logger.html)
* [class Exception](http://docs.ruby-lang.org/ja/2.2.0/class/Exception.html)
