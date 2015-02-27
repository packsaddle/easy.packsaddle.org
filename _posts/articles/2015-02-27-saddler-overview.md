---
layout: post
title: "変更したファイルにrubocopやjscsを実行して pull requestに自動でコメントする"
excerpt: >-
  tl;dr コマンドをパイプでつないで、CIからsaddlerコマンドで書き込みする。
categories: articles
tags: [saddler, overview, js, ruby, travisci, circleci, ja]
author: sanemat
comments: true
share: true
image:
  feature: so-simple-sample-image-7.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---

## tl;dr

* 変更したファイルにrubocopやjscsを実行して、pull requestに自動でコメントさせる方法。
* コマンドをパイプでつないで、CIからsaddlerコマンドで書き込みする。
* デモリポジトリに
[rubocop, travis ci](https://github.com/packsaddle/example-travis_ci-pull_request_review),
[jscs, travis ci](https://github.com/packsaddle/example-travis_ci-pull_request_review-jscs)
エラーになるpull requestしてみてね!

<figure class="half">
  <img src="/images/2015-02-27-jscs-comment.png" alt="jscs comment">
  <img src="/images/2015-02-27-rubocop-comment.png" alt="rubocop comment">
  <figcaption>saddlerの実行結果イメージ</figcaption>
</figure>

一番目がjscs, 二番目がrubocop。

## 流れ

### diffのあるファイル名を取り出す

{% highlight bash %}

$ git diff --name-only origin/master
README.md
bin/run-tests.sh
lib/example/travis_ci.rb

{% endhighlight %}

こんなdiffにrubocop や jscsを実行したい。
diffのあるファイル名を取り出す。

### lint実行したいファイルだけに絞り込む

{% highlight bash %}

$ git diff --name-only origin/master \
 | xargs rubocop-select
/Users/sane/work/tmp/packsaddle/example-ruby-travis-ci/lib/example/travis_ci.rb

{% endhighlight %}

lint実行したいファイルだけに絞り込む。
rubocopはrubocop-select という対象絞り込みのコマンドを使う。

ちなみに、jscsの場合はgrepにした。ここ以外は一緒なので割愛。

{% highlight bash %}

$ git diff --name-only origin/master \
 | grep -e '.js$'
/Users/sane/work/tmp/packsaddle/example-ruby-travis-ci/lib/example/some_your.js

{% endhighlight %}

### lint実行して結果をcheckstyle formatで出力する

{% highlight bash %}

$ git diff -z --name-only origin/master \
  | xargs -0 rubocop-select \
  | xargs rubocop \
      --require rubocop/formatter/checkstyle_formatter \
      --format RuboCop::Formatter::CheckstyleFormatter
<?xml version='1.0'?>
<checkstyle>
  <file name='/Users/sane/work/tmp/packsaddle/example-ruby-travis-ci/lib/example/travis_ci.rb'>
    <error line='7' column='120' severity='info' message='Line is too long. [163/120]' source='com.puppycrawl.tools.checkstyle.Metrics/LineLength'/>
  </file>
  <file name='/Users/sane/work/tmp/packsaddle/example-ruby-travis-ci/lib/example/travis_ci.rb'>
    <error line='16' column='120' severity='info' message='Line is too long. [163/120]' source='com.puppycrawl.tools.checkstyle.Metrics/LineLength'/>
  </file>
</checkstyle>

{% endhighlight %}

実行して、結果をcheckstyle formatで出力。
rubocopの場合はcheckstyle formatterをgemで入れる。
jscsの場合はcheckstyle reporterがある。exampleで用意したものはgulp-jscsなので、中から無理やり。
npm runでxargs渡せるんだろうか。

{% highlight bash %}

$ git diff --name-only origin/master \
  | grep -e '\.js$' \
  | xargs ./node_modules/gulp-jscs/node_modules/.bin/jscs \
      --reporter checkstyle

{% endhighlight %}

このステップは、checkstyle formatでどうしても出力できなかったら、そのライブラリのreporterを書くか、結果のconverterを書く。

### checkstyle formatの中から、増えた行のエラーだけ抽出する

{% highlight bash %}

$ git diff -z --name-only origin/master \
  | xargs -0 rubocop-select \
  | xargs rubocop \
      --require rubocop/formatter/checkstyle_formatter \
      --format RuboCop::Formatter::CheckstyleFormatter \
  | checkstyle_filter-git diff origin/master
<?xml version='1.0'?>
<checkstyle>
  <file name='/Users/sane/work/tmp/packsaddle/example-ruby-travis-ci/lib/example/travis_ci.rb'>
    <error column='120' line='7' message='Line is too long. [163/120]' severity='info' source='com.puppycrawl.tools.checkstyle.Metrics/LineLength'/>

  </file>
</checkstyle>

{% endhighlight %}

checkstyle formatのなかから、増えた行のエラーだけ抽出する。
この場合7行目はdiffにあったけど、16行目はdiffに無かったのですね。

### 増えた行のエラーをPull requestにコメントする

{% highlight bash %}

$ git diff -z --name-only origin/master \
  | xargs -0 rubocop-select \
  | xargs rubocop \
      --require rubocop/formatter/checkstyle_formatter \
      --format RuboCop::Formatter::CheckstyleFormatter \
  | checkstyle_filter-git diff origin/master \
  | saddler report \
     --require saddler/reporter/github \
     --reporter Saddler::Reporter::Github::PullRequestReviewComment

{% endhighlight %}

増えた行のエラーをPull requestにコメントする。
おめでとうございます!

## demo

* TravisCI
    * [rubocop](https://github.com/packsaddle/example-travis_ci-pull_request_review),
    [実際に実行するスクリプト](https://github.com/packsaddle/example-travis_ci-pull_request_review/blob/master/bin/run-rubocop.sh)
    * [jscs](https://github.com/packsaddle/example-travis_ci-pull_request_review-jscs),
    [実際に実行するスクリプト](https://github.com/packsaddle/example-travis_ci-pull_request_review-jscs/blob/master/bin/run-jscs.sh)
* CircleCI
    * [rubocop](https://github.com/packsaddle/example-circle_ci-pull_request_review),
    [実際に実行するスクリプト](https://github.com/packsaddle/example-circle_ci-pull_request_review/blob/master/bin/run-rubocop.sh)
    * [jscs](https://github.com/packsaddle/example-circle_ci-pull_request_review-jscs),
    [実際に実行するスクリプト](https://github.com/packsaddle/example-circle_ci-pull_request_review-jscs/blob/master/bin/run-jscs.sh)

TravisCIとCircleCIで確認してます。
CircleCIのほうは、外部リポジトリからのPull Requestでbuildしないようにしているので、
TravisCIのリポジトリで、rubocop/jscsが指摘する項目があるプルリクエストを出して試してみてください!
自分のリポジトリに設定する場合、GitHub Tokenの設定が必要です。
各デモのREADMEとスクリプトを参考に設定してください。

## まとめ

* 変更したファイルにrubocopやjscsを実行して、pull requestに自動でコメントさせる方法。
* コマンドをパイプでつないで、CIからsaddlerコマンドで書き込みする。
* デモリポジトリに
[rubocop, travis ci](https://github.com/packsaddle/example-travis_ci-pull_request_review),
[jscs, travis ci](https://github.com/packsaddle/example-travis_ci-pull_request_review-jscs)
エラーになるpull requestしてみてね!

使ってみたをブログやqiitaに書いてもらえると助かる！フィードバックください！！
詰まったら聞いてください。このサイトやREADMEなどに反映します。
gitlabで使いたい！stashで、bitbucketで、etcなどもこれから作りやすいようにしてます。reporterの作り方書きます。
コマンドラインツールなんでも行けるはずなので他にもeslintなど行ける！！
[ruby-saddlerにスター](https://github.com/packsaddle/ruby-saddler)ください!
