---
layout: post
title: "CI環境からGitHubのPull Requestのstatusを簡単に通知するコマンド"
excerpt: >-
  tl;dr github-status-notifierコマンドでCI環境からGitHubのPull Requestのstatusを
  簡単に通知する。TravisCIがやってるみたいなのを作れる。
categories: articles
tags: [github, github_status_notifier, overview, status, travisci, circleci, ja]
author: sanemat
comments: true
share: true
image:
  thumb: 2015-02-28-github-status-notifier.png
---

## tl;dr

* [github_status_notifier](https://github.com/packsaddle/ruby-github_status_notifier)コマンドでCI環境からGitHubのPull Requestのstatusを
  簡単に通知する。
* TravisCIがやってるみたいなテスト中は黄色、テスト終わったら緑/赤のを簡単に作れる。

<figure class="half">
  <img src="/images/2015-02-28-github-status-notifier-pending.png" alt="github status pending">
  <img src="/images/2015-02-28-github-status-notifier.png" alt="github status success">
  <figcaption>github-status-notifierの実行結果イメージ</figcaption>
</figure>

## 使い方

単体で使ってもよし、典型的には、他のコマンドの前後を `state pending`と `exit-status $?`で囲んで使う。
こうするとこのスクリプトが始まったところでpending の黄色が始まり、成功/失敗で success/failureのstatusが表示できる。

{% highlight bash %}

gem install github_status_notifier

github-status-notifier notify --state pending --context saddler/rubocop

SOME_YOUR_COMMAND

github-status-notifier notify --exit-status $? --context saddler/rubocop

{% endhighlight %}

## 使い道

GitHub関連サービス作るような人には便利に使えそう。Saddlerとの連携などに便利。

チェックがまだ始まってないのか、チェック中なのか、チェックが終わって何も問題がないから無言なのか、途中で終わってるのか、
Pull Requestの画面上から把握できるようになる。

これから用にJenkinsなりで自前CIを作ってて、status通知連携だけで来てない人(いるのか?)は多少便利になるかも。
でもその場合はこっち見てその通りに作るほうがよい。 [Building a CI server | GitHub API](https://developer.github.com/guides/building-a-ci-server/)

[ruby-github_status_notifierにスター](https://github.com/packsaddle/ruby-github_status_notifier)ください!

----

## 参照

* [ruby-github_status_notifierにスター](https://github.com/packsaddle/ruby-github_status_notifier)
* [See results from all pull request status checks](https://github.com/blog/1935-see-results-from-all-pull-request-status-checks)
* [Statuses GitHub API](https://developer.github.com/v3/repos/statuses/)
* [Building a CI server GitHub API](https://developer.github.com/guides/building-a-ci-server/)
* [Commit Status API](https://github.com/blog/1227-commit-status-api)
