---
layout: post
title: "restore_bundled_withを使ってBundlerのBUNDLED WITHをうまく取り扱う"
excerpt: >-
  tl;dr Bundler v1.10.0からBUNDLED WITHのsectionが導入された。
  restore_bundled_withを使うことで、Bundler開発チームの精神を逸脱して(!)、
  異なるversionのBundlerとBUNDLED WITHをうまく取り扱う。
categories: articles
tags: [restore_bundled_with, bundler, ja]
author: sanemat
comments: true
share: true
image:
  thumb: 2015-06-21-bundled-with.png
---

## tl;dr

* Bundler v1.10.0からBUNDLED WITHのsectionが導入された。
* [restore_bundled_with](https://rubygems.org/gems/restore_bundled_with)を使うことで、Bundler開発チームの精神を逸脱して(!)、異なるversionのBundlerとBUNDLED WITHをうまく取り扱う。

<figure>
  <img src="/images/2015-06-21-bundled-with.png" alt="BUNDLED WITH section">
  <figcaption>BUNDLED WITHのsectionで差分が出る</figcaption>
</figure>

## Bundler v1.10 とBUNDLED WITH

Bundler v1.10.0から`BUNDLED WITH`というsectionに`bundle update`を実行したBundlerのversionを
記録するようになった。
日本語詳細はこちらが詳しい。
[Ruby - BUNDLED WITH で Gemfile.lock が更新されてしまう件 - Qiita](http://qiita.com/suu_g/items/2b1630b8015d51c5292e)

### 挙動の確認

Bundler v1.10.2で`bundle update`して、BUNDLED WITHにversionを記録しておく。

{% highlight bash %}
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
{% endhighlight %}

#### Bundler v1.9.9で bundle update

BUNDLED WITHのsectionごと消える。悲しい。

{% highlight bash %}
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
{% endhighlight %}

#### Bundler v1.10.4(新しい)で bundle update

BUNDLED WITHの記録を更新する。

{% highlight bash %}
$ bundle update
(update)

$ git diff
(snip)
@@ -291,4 +292,4 @@ DEPENDENCIES
   webmock

 BUNDLED WITH
-   1.10.2
+   1.10.4
{% endhighlight %}

#### Bundler v1.10.1(古い)で bundle update

Warning出しつつ、BUNDLED WITHは更新しない。

{% highlight bash %}
$ bundle update
Warning: the running version of Bundler is older than the version that created the lockfile.
We suggest you upgrade to the latest version of Bundler by running `gem install bundler`.
(snip)

$ git diff
(no diff)
{% endhighlight %}

## 正しい解

まずは、Bundlerのversionをv1.10系の最新版に上げる。
そして、依存ライブラリのバージョンを上げていくと同時にBundlerも最新版が出るたびに上げていく。

## 現実解?

* そうは言っても、localではpreとかv2とか新しすぎるものも使いたい。
* 実際にproductionで実行するBundlerのversionと離れてしまいかねないのはいつか問題になりそう。
* Gemfileの中はBundler管轄だけど、Bundler自体はそこで管理してないわけで、lockの記述に引きずられるのはどうなんだろう(私見)
* minimalなbundlerのmini_bundler(仮)が薄くいて、bundle execなどのコマンドはその内側でバージョン管理されたbundlerで実行すればいい?(私見)

などの理由から、Bundler開発チームの精神を逸脱して(!)、[restore_bundled_with](https://rubygems.org/gems/restore_bundled_with)というgemの`restore-bundled-with`コマンドを使う。

## restore_bundled_with

`Gemfile.lock`のBUNDLED WITHにリポジトリとdiffがある状態で、`restore-bundled-with`コマンドを実行する。
つまり、`bundle update`したあと、commitする前に、`restore-bundled-with`する。
すると、リポジトリからBUNDLED WITHのsectionを取り出してきて、そこだけ元に戻る。

Bundler v1.9なら、消えたsectionが戻る。
より新しいBundler v1.10なら、使った記録がそこだけ戻る。

## まとめ

Bundler開発チームの精神を逸脱するので、あまり推奨はしない。
最新版を使って、もし壊れていたらみんなで直していくのが、アプリケーションを健全に保つ一番痛みが少ない方法なのです!

[詰まったら聞いて](https://github.com/packsaddle/ruby-restore_bundled_with/issues/new)ください。このサイトやREADMEなどに反映します。
[ruby-restore_bundled_withにスター](https://github.com/packsaddle/ruby-restore_bundled_with)ください!

----

## 参照

* [ruby-restore_bundled_withにスター](https://github.com/packsaddle/ruby-restore_bundled_with)
* [Ruby - BUNDLED WITH で Gemfile.lock が更新されてしまう件 - Qiita](http://qiita.com/suu_g/items/2b1630b8015d51c5292e)
* [Track Bundler version in lockfile by smlance #3485 bundler/bundler](https://github.com/bundler/bundler/pull/3485)
* [Do not update BUNDLED WITH if no other changes to the lock happened #3697 bundler/bundler](https://github.com/bundler/bundler/issues/3697)
