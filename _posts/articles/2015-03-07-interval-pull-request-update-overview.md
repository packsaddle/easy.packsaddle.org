---
layout: post
title: "定期的にライブラリの依存関係をアップデートしてPull Requestする"
excerpt: >-
  tl;dr ライブラリを組み合わせて、CIサーバーから依存関係アップデートしたPull Requestを送る。
categories: articles
tags: [github, dependency-update, tachikoma, overview, ja]
author: sanemat
comments: true
share: true
image:
  thumb: 2015-03-07-interval-pull-request.png
---

## tl;dr

* Cron for GitHubで定期的にCIサーバーのフックを起動する
* CI側では、特定のブランチの場合のみ起動するスクリプトを用意する
* 自分のやりたい操作をして、git_httpsable-push, pull_request_create でPull Requestを送る

<figure>
  <img src="/images/2015-03-07-interval-pull-request.png" alt="Interval Dependency Update">
  <figcaption>定期的にDependency UpdateのPull Requestを送る</figcaption>
</figure>

## フックの起動

Cron for GitHubのschedulerはこのように設定した。

{% highlight bash %}

bundle exec cron-for-github clear --slug=metarubygems/carrion_crow --namespace=cron_for_tachikoma --verbose && bundle exec cron-for-github ping --slug=metarubygems/carrion_crow --namespace=cron_for_tachikoma --verbose

{% endhighlight %}

Cron for GitHubの使い方はリンク先を参照する。

* [GitHubとHerokuの組み合わせでCron for GitHub – Saddler - checkstyle to anywhere](http://packsaddle.org/articles/cron-for-github-app-overview/)

キモはnamespaceの指定で、`cron_for_tachikoma/XXXXX`というブランチを定期的に作るようにする。
CI側ではこのブランチを監視する。

## CI側スクリプト

CI側では、テスト後にスクリプトを実行。なお、例はCircleCIの場合。

{% highlight yaml %}

test:
  post:
    - bin/run-tachikoma.sh

{% endhighlight %}

実行するスクリプト。
特定のブランチを監視して、あとは曜日で起動するしないの頻度を決めている。
この場合、日曜のみ起動。
環境変数 `GITHUB_ACCESS_TOKEN`に[token](https://github.com/settings/tokens/new) を入れている。

{% highlight bash %}

#!/usr/bin/env bash
set -ev

# only sunday
if [[ "${CIRCLE_BRANCH}" != "master" && "${CIRCLE_BRANCH}" =~ ^cron_for_tachikoma/.* && $(date +%w) -eq 0 ]]; then
  # gem prepare
  gem install --no-document git_httpsable-push pull_request-create

  # git prepare
  git config user.name sanemat
  git config user.email foo@example.com
  HEAD_DATE=$(date +%Y%m%d_%H-%M-%S)
  HEAD="tachikoma/update-${HEAD_DATE}"

  # checkout
  git checkout -b "${HEAD}" origin/master

  # bundle install
  bundle --no-deployment --without nothing --jobs 4

  # bundle update
  bundle update

  git add Gemfile.lock
  git commit -m "Bundle update ${HEAD_DATE}"

  # git push via https
  git httpsable-push origin "${HEAD}"

  # pull request
  pull-request-create
fi

exit 0

{% endhighlight %}

Git HTTPSable Pushや PullRequest Create はリンク先を参照する。

* [Httpsで簡単に、ログ安全に、git pushする Git HTTPSable Push – Saddler - checkstyle to anywhere](http://packsaddle.org/articles/git-httpsable-push-overview/)
* [packsaddle/ruby-pull_request-create](https://github.com/packsaddle/ruby-pull_request-create)

以上により、
[定期的にライブラリの依存関係をアップデートしてPull Request](https://github.com/metarubygems/carrion_crow/pull/18)
出来るようになった。

## 備考

反対に、webhook ping用のブランチである `cron_for_tachikoma/XXXXX` ではテストをスキップする、というのを入れておかないと、
テストのキューが多く詰まるような環境では悪い影響出る可能性があるので注意。

## 他言語でも

実行するスクリプトの中身はシェルスクリプトでコマンドが並んでいるだけなので、もちろん言語非依存。
`composer update`でも`pod update`でもOK。
どんなコマンドオプションだとCIサーバー上で動かしやすいのかは、
[Tachikoma gem](https://github.com/sanemat/tachikoma/blob/master/lib/tachikoma/application.rb)
見るといけそう。

## まとめ

* ライブラリを組み合わせて、CIサーバーから依存関係アップデートしたPull Requestを送る事ができるようになった

使ってみたをブログやqiitaに書いてもらえると助かる！フィードバックください！！
他にも使い道がありそう。

----

## 参照

* [GitHubとHerokuの組み合わせでCron for GitHub – Saddler - checkstyle to anywhere](http://packsaddle.org/articles/cron-for-github-app-overview/)
* [Httpsで簡単に、ログ安全に、git pushする Git HTTPSable Push – Saddler - checkstyle to anywhere](http://packsaddle.org/articles/git-httpsable-push-overview/)
* [Tachikoma gem](https://github.com/sanemat/tachikoma/blob/master/lib/tachikoma/application.rb)
