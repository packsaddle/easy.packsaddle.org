---
layout: post
title: "GitHubとHerokuの組み合わせでCron for GitHub"
excerpt: >-
  tl;dr GitHubとHeroku Schedulerの組み合わせで、Cron for GitHubを実現する。
  Heroku Buttonで一発。
categories: articles
tags: [cron, github, heroku, travisci, circleci, ja]
author: sanemat
comments: true
share: true
image:
  thumb: 2015-03-03-cron-for-github-app-branches.png
---

## tl;dr

* GitHubとHeroku Schedulerの組み合わせでCron for GitHubを実現する
* [![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/packsaddle/ruby-cron_for_github-app)

<figure class="half">
  <img src="/images/2015-03-03-cron-for-github-app-readme.png" alt="GitHub readme">
  <img src="/images/2015-03-03-cron-for-github-app-branches.png" alt="GitHub branches">
  <figcaption>cron-for-githubの実行結果イメージ</figcaption>
</figure>

## 使い道

1. たとえば、毎日`ping/` のbranchを生成しておく。
2. CI用のスクリプトには、`ping/` 以下のbranchのビルドだったらこれこれする、と条件を書く。
3. 完成!

## はじめ方

1. [cron_for_github-app](https://github.com/packsaddle/ruby-cron_for_github-app)
   の[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/packsaddle/ruby-cron_for_github-app)
   からherokuにデプロイ。
2. cronを使いたいリポジトリがprivateなら`repo`, publicなら`public_repo`の[github access tokenを発行](https://github.com/settings/tokens/new)。
3. GITHUB_ACCESS_TOKEN にそれを設定。
4. このコマンドを実行できるようになる。

{% highlight bash %}

cron-for-github ping --slug=YOU/YOUR_REPO

{% endhighlight %}

5. Heroku Schedulerで定期実行。

<figure>
  <img src="/images/2015-03-03-cron-for-github-app-heroku-scheduler.png" alt="heroku scheduler">
  <figcaption>heroku schedulerの設定イメージ</figcaption>
</figure>

コマンドのオプション詳細は [cron_for_github gemのreadme](https://github.com/packsaddle/ruby-cron_for_github#command)参照。

### オススメ設定

このままだと、いっぱいブランチが作られていくママなので、

{% highlight bash %}

bundle exec cron-for-github clear --slug=YOU/YOUR_REPO --verbose && bundle exec cron-for-github ping --slug=YOU/YOUR_REPO --verbose

{% endhighlight %}

こんな感じに実行して、pingする前にclearするとよい。
verboseオプションつけるとそれなりにログが出る。
そして、それをnewrelicやlogentriesなど、ふつうのherokuアプリのように監視する。

## demo

* CircleCI
    * [Branches packsaddle/example-circle_ci-pull_request](https://github.com/packsaddle/example-circle_ci-pull_request/branches/all)

CircleCIで確認してます。
10分おきに起動したらたくさんbranchできたので、今は1日1回起動中。
TravisCIでも動くはず。

`.travis.yml`で

{% highlight yaml %}

branches:
  only:
    - master

{% endhighlight %}

している場合は

{% highlight yaml %}

branches:
  only:
    - master
    - /^cron-for-github\/.*$/

{% endhighlight %}

など、自分の設定に合わせて要修正。

## まとめ

* GitHubとHeroku Schedulerの組み合わせでCron for GitHubを実現する
* [![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/packsaddle/ruby-cron_for_github-app)

使ってみたをブログやqiitaに書いてもらえると助かる！フィードバックください！！
多分他にも使いみちがありそう。
[詰まったら聞いて](https://github.com/packsaddle/ruby-cron_for_github-app/issues/new)ください。このサイトやREADMEなどに反映します。
[ruby-cron_for_github-app](https://github.com/packsaddle/ruby-cron_for_github-app)と
[ruby-cron_for_githubにスター](https://github.com/packsaddle/ruby-cron_for_github)ください!

----

## 参照

* [ruby-cron_for_github-appにスター](https://github.com/packsaddle/ruby-cron_for_github-app)
* [ruby-cron_for_githubにスター](https://github.com/packsaddle/ruby-cron_for_github)
* [Git Refs | GitHub API](https://developer.github.com/v3/git/refs/)
* [Creating a 'Deploy to Heroku' Button | Heroku Dev Center](https://devcenter.heroku.com/articles/heroku-button)
* [app.json Schema | Heroku Dev Center](https://devcenter.heroku.com/articles/app-json-schema)
