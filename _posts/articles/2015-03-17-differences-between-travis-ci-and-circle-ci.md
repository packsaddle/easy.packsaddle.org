---
layout: post
title: "TravisCIとCircleCIでちょっとずつ違うビルド環境の考え方の違い"
excerpt: >-
  tl;dr TravisCIとCircleCIの根底にある違いの考え方を理解すると早い。
  そして、違いを考慮しているproductを使うと便利。
categories: articles
tags: [travis-ci, circle-ci, ci, git, pull-request, github, ja]
author: sanemat
comments: true
share: true
image:
  thumb: 2015-03-17-travis-result.png
---

## tl;dr

* TravisCIとCircleCIの根底にある違いの考え方を理解すると早い。
* そして、違いを考慮しているproductを使うと便利。
    * e.g. [checkstyle形式の結果をpull request review commentするsaddler](https://github.com/packsaddle/ruby-saddler)

<figure class="half">
  <img src="/images/2015-03-17-travis-result.png" alt="TravisCI result">
  <img src="/images/2015-03-17-circle-result.png" alt="CircleCI result">
  <figcaption>CI result</figcaption>
</figure>

## 考え方

### CircleCI

* pushされたbranchをビルドする。

### TravisCI

* pushされたbranchをビルドする。
* pull requestのbranchを、仮にmasterにmergeしてみて、ビルドする。

## 仮にmasterにmergeしてみて??

TravisCIの二つ目のがわかりにくくて、混乱する。
やっていることは正しいんだけど、挙動が直感から外れるので、
TravisCIヘビーに使っているユーザーにもわりと意味不明挙動扱いされやすい(要出典)

* pull requestするとtravisのビルドがなぜか2個走る
* (GitHub statusが最新結果しか取れなかった頃は)一度passしたはずのテストが、
なぜかあとからやってきたfailにかき消される passしたはずのテストはどこへ
* .travis.yml でよく見る branches: only: - master ってなんだよ
おれはmaster以外のテストもしたいぞ
あれ、よくわからないが意図通りに動いてるぞ
何だこの設定

たとえばbranchを作った時のmasterと今のmasterが乖離していて、
pull requestのbranchのテストは全部通るけど、mergeしたらmasterが落ちる、
というのはよくあることだと思う。
そこを人間任せにするのか「ちゃんとmasterに追い付かせておいてね」、
仮にmergeしてみてtestするのか、は考え方による。
後者は便利に見えるが、なんで落ちたかよくわからない状況になることもありがち。

### branchのpushでbuildされるとジャマ

となると単にbranchをpushしただけでbuildされるとジャマなので、

{% highlight yaml %}
# .travis.yml
branches:
  only:
  - master
{% endhighlight %}

としたい。
これで、master branchへの変更、およびmasterに対してのpull requestに仮にmergeしてみて、が走る。
大きめのfeatureブランチを作っていて、
そのfeatureブランチに対するpull requestも仮にmergeしてみて、をしたい場合

{% highlight yaml %}
# .travis.yml
branches:
  only:
  - master
  - feature/your-big-change
{% endhighlight %}

とすればよい。

なにかこのブランチが作られた時には、hookで何かしたい、などの場合は

{% highlight yaml %}
# .travis.yml
branches:
  only:
  - master
  - /^cron_for_tachikoma\/.*/
{% endhighlight %}

とする。正規表現が使える。

## pull request時の挙動

### TravisCI

pull request作成時にhookで起動する。
条件を満たしていれば。

pull request済みのbranchに追加のcommitでも起動する。

### CircleCI

pull request作成時にはhookで**起動しない**。
branchのpushを検知する側だけなので。

だから、pull requestに対して何かをしたい、例えばlintしてコメントをつけるとか、の場合、
さっさとpull requestをつくらないと、コメントをpostする先のpull requestがない状態で、
処理が終わってしまう。
branchをpushしてしばらくしてからpull request作成、
そのpull requestに追加でcommitしてpush、
この後者のpushのタイミングで初めてコメントが付くことになる。
ちょっと注意。

## git操作での挙動

### TravisCI

仮にmergeしてみる TravisCI式がいいことづくめか?
というとそうでもない。

{% highlight text %}
# TravisCI checkoutの挙動(logより)
$ git clone --depth=50 git://github.com/sanemat/ruby-example-rails-banana.git sanemat/ruby-example-rails-banana
$ cd sanemat/ruby-example-rails-banana
$ git fetch origin +refs/pull/2/merge:
$ git checkout -qf FETCH_HEAD

$ tig
13fac8f 2015-03-16 23:35 Murahashi Sanemat Kenichi M─┐ [HEAD] Merge c47af08fe6c62f6c95fa9dea440866d6348ac38
c47af08 2015-03-16 23:35 sanemat                   │ o Bundle update 20150316_23-34-12
37f6de7 2015-03-17 08:07 Murahashi Sanemat Kenichi o─┘ [master] {origin/HEAD} {origin/master} chore(tachiko
8b7d629 2015-03-17 07:58 Murahashi Sanemat Kenichi o chore(travis): add debug

$ git branch
* (detached from FETCH_HEAD)
  master
{% endhighlight %}

git 操作でなにかするアプリの場合、
あまりこれを想定した挙動になっていないことが多い。
それからたとえば、GitHubのapiと連携してpull requestにreview commentをつけたいので、
上のtigで言う`c47af08`を取りたい場合に、うってなる。
[saddler-reporter-support-gitでは結構頑張って解決](https://github.com/packsaddle/ruby-saddler-reporter-support-git/blob/cc8ce5f819e4d137b94f71b3f3ca4f3f7208ba00/lib/saddler/reporter/support/git/repository.rb#L39-L50)
している。
feature branchにmasterをmergeして追い付かせた時に正しく検出しなそう、とかある。

注) TravisCIに限れば、環境変数 `TRAVIS_COMMIT`で該当コミットが取れる。

この「仮にmergeしてみる」は色々流派がありそうで、実際対TravisCIでは動いているけど、
他のパターンが有るときに全部解決してるかは自信ない。

#### pushされたbranchのビルド

Travisは必要最低限にチューンしてあるので、masterとかorigin/masterとかいない。
ちょっとだけ注意。

{% highlight bash %}
git checkout -b YOU_WANT_TO_CREATE origin/master
{% endhighlight %}

と手癖で打つと`origin/master`無いので落ちる。

{% highlight text %}
# TravisCI checkoutの挙動(logより)
$ git clone --depth=50 --branch=cron_for_tachikoma/56e24ecc-46bd-4d15-92fb-8d27fde41c8c git://github.com/sanemat/ruby-example-rails-banana.git sanemat/ruby-example-rails-banana
$ cd sanemat/ruby-example-rails-banana
$ git checkout -qf 8b7d6291495e947c7961e98dd9d89042eed2fd05

$ git branch
* (detached from 8b7d629)
  cron_for_tachikoma/56e24ecc-46bd-4d15-92fb-8d27fde41c8c

$ git branch -r
  origin/cron_for_tachikoma/56e24ecc-46bd-4d15-92fb-8d27fde41c8c
{% endhighlight %}

##### 余談

checkout -b したいときは
{% highlight bash %}
git checkout -b YOU_WANT_TO_CREATE "${TRAVIS_BRANCH}"
{% endhighlight %}
か単に
{% highlight bash %}
git checkout -b YOU_WANT_TO_CREATE
{% endhighlight %}
とする。

### CircleCI

CircleCIはおそらく事故防止のためにgit関連のログをwebからは表示しないので、
推測だけど

{% highlight text %}
# CircleCI checkoutの挙動(推測)
$ git clone git://github.com/sanemat/ruby-example-rails-banana.git ruby-example-rails-banana
$ cd ruby-example-rails-banana
$ git checkout YOUR_TARGET_BRANCH
{% endhighlight %}

してるだけだと思う。
オーソドックス。
そして多分cloneは初回だけで、cacheしつつあとはfetchとcheckoutっぽい。
sshで入るとこんな感じなので。

{% highlight text %}
$ git branch
  chore/use-tachikoma
  cron_for_github/530f0756-3664-4631-b5f4-eb53a842677a
* cron_for_tachikoma/bar-boo
  master
  pull/9
  tachikoma/update-20150222035312
  tachikoma/update-20150301035325
{% endhighlight %}

remote側も綺麗に全部見える(必要最低限ではなく富豪的)

{% highlight text %}
$ git branch -r
  origin/HEAD -> origin/master
  origin/chore/use-tachikoma
  origin/cron_for_github/0e560ce0-6ae0-49b4-a8f9-26b335a752b1
  origin/cron_for_github/44f3c4e7-af74-4e29-bbbf-6c04e1b41048
  origin/cron_for_github/530f0756-3664-4631-b5f4-eb53a842677a
  origin/cron_for_github/5b9d9688-3844-42a4-b34a-3d519177e7b3
  origin/cron_for_github/foo-bar-baz
  origin/cron_for_tachikoma/148fbb23-dc21-461b-b207-5be874ea864f
  origin/cron_for_tachikoma/2212ad06-cdd2-4e77-8071-8552f1268d9e
  origin/cron_for_tachikoma/6344f6a9-2989-4352-9e5e-78ca1f71cc41
  origin/cron_for_tachikoma/bar-boo
  origin/master
  origin/pull/9
  origin/tachikoma/update-20150215035300
  origin/tachikoma/update-20150222035312
  origin/tachikoma/update-20150301035325
  origin/tachikoma/update-20150307_13-26-17
  origin/tachikoma/update-20150307_13-34-52
  origin/tachikoma/update-20150308_15-01-02
  origin/tachikoma/update-20150315035345
  origin/tachikoma/update-20150315_15-03-42
{% endhighlight %}

#### 余談

これたまに `git fetch origin --prune` しないと膨れ上がりそう。
ログとかの都合上あったほうがいいのかもだから詳しくは調べてない。

## まとめ

普通にやったらこうなる、を高速に解決しているのがCircleCI。
多分Jenkinsで自分でやったらこうなりそう、ただし低速。
ちゃんとやったらこうなる、を正しく解決しているのがTravisCI。
ただし、正しく理解しないと、適当に、じゃあこうかな? とやっても正解にたどり着けない。

drone.io, wercker, teamcity, codeship, あたりのメジャーどころは、
だいたいどっちかなんじゃないかな(未調査)、
カバーできてるんじゃないかな、カバーできてるといいなあ。
詳しい人に補足おねがいしたい!!

ローカルやCI上で動くはずのツールが、CIサービス上では動かないとき、
このへんの詰めが漏れてることが多いので e.g. pronto なぜか動かないときは、
ココらへんに気を配ってcontributeするとよいです。
なお、
[checkstyle形式の結果をpull request review commentするsaddler](https://github.com/packsaddle/ruby-saddler)
などのpacksaddleプロダクトはその辺のCI間の差異を吸収して、使うときは考えなくていいように、作ってます。

----

## 参照

* [CircleCI example](https://circleci.com/gh/metarubygems/carrion_crow)
* [TravisCI example](https://travis-ci.org/sanemat/ruby-example-rails-banana)
* [Configuring CircleCI - CircleCI](https://circleci.com/docs/configuration)
* [Travis CI: Configuring your build](http://docs.travis-ci.com/user/build-configuration/)
* [Travis CI: The Build Environment](http://docs.travis-ci.com/user/ci-environment/)
* [Travis CI: Environment Variables](http://docs.travis-ci.com/user/environment-variables/)
* [Continuous Integration - StackShare](http://stackshare.io/continuous-integration)
* [CI SaaS / OSSをまとめてみたら25個もあったヨ - SideCI Blog](http://sideci.hatenablog.com/entry/2015/03/13/144948)
