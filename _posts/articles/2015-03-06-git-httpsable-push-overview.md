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
2. Git HTTPSable Push を使い`git httpsable-push` で簡単に、ログ安全に、httpsでのgit pushが出来る

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

{% highlight bash %}

$ git config --get remote.origin.url
git@github.com:packsaddle/ruby-git_httpsable-push.git
# GITHUB_ACCESS_TOKEN=__MY_TOKEN__

# success
$ git httpsable-push origin refactor/logger-to-class --verbose
I, [2015-03-06T19:29:54.250481 #33391]  INFO -- GitHttpsable::Push: Starting Git
I, [2015-03-06T19:29:54.270408 #33391]  INFO -- GitHttpsable::Push: git '--git-dir=/Users/sane/work/ruby-study/git_httpsable-push/.git' '--work-tree=/Users/sane/work/ruby-study/git_httpsable-push' config '--list'  2>&1
I, [2015-03-06T19:29:54.281424 #33391]  INFO -- GitHttpsable::Push: git '--git-dir=/Users/sane/work/ruby-study/git_httpsable-push/.git' '--work-tree=/Users/sane/work/ruby-study/git_httpsable-push' config '--list'  2>&1
I, [2015-03-06T19:29:56.632766 #33391]  INFO -- GitHttpsable::Push: git '--git-dir=/Users/sane/work/ruby-study/git_httpsable-push/.git' '--work-tree=/Users/sane/work/ruby-study/git_httpsable-push' push 'https://MASKED@github.com/packsaddle/ruby-git_httpsable-push.git' 'refactor/logger-to-class'  2>&1
I, [2015-03-06T19:29:56.632944 #33391]  INFO -- GitHttpsable::Push: {:output=>nil}

# error occurs
$ git httpsable-push origin refactor/logger-to-class --verbose
I, [2015-03-06T19:33:47.692967 #34984]  INFO -- GitHttpsable::Push: Starting Git
I, [2015-03-06T19:33:47.701617 #34984]  INFO -- GitHttpsable::Push: git '--git-dir=/Users/sane/work/ruby-study/git_httpsable-push/.git' '--work-tree=/Users/sane/work/ruby-study/git_httpsable-push' config '--list'  2>&1
I, [2015-03-06T19:33:47.711167 #34984]  INFO -- GitHttpsable::Push: git '--git-dir=/Users/sane/work/ruby-study/git_httpsable-push/.git' '--work-tree=/Users/sane/work/ruby-study/git_httpsable-push' config '--list'  2>&1
I, [2015-03-06T19:33:50.657268 #34984]  INFO -- GitHttpsable::Push: git '--git-dir=/Users/sane/work/ruby-study/git_httpsable-push/.git' '--work-tree=/Users/sane/work/ruby-study/git_httpsable-push' push 'https://MASKED@github.com/packsaddle/ruby-git_httpsable-push.git' 'refactor/logger-to-class'  2>&1
E, [2015-03-06T19:33:50.657632 #34984] ERROR -- GitHttpsable::Push: Have a question? Please ask us:
E, [2015-03-06T19:33:50.657667 #34984] ERROR -- GitHttpsable::Push: https://github.com/packsaddle/ruby-git_httpsable-push/issues/new
E, [2015-03-06T19:33:50.657711 #34984] ERROR -- GitHttpsable::Push: options:
E, [2015-03-06T19:33:50.657778 #34984] ERROR -- GitHttpsable::Push: {"debug"=>false, "verbose"=>true, "force"=>false, "tags"=>false}
/Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/git-1.2.9.1/lib/git/lib.rb:917:in `command': (Git::GitExecuteError) git '--git-dir=/Users/sane/work/ruby-study/git_httpsable-push/.git' '--work-tree=/Users/sane/work/ruby-study/git_httpsable-push' push 'https://MASKED@github.com/packsaddle/ruby-git_httpsable-push.git' 'refactor/logger-to-class'  2>&1:remote: Invalid username or password. (GitHttpsable::Push::GitHttpsablePushError)
fatal: Authentication failed for 'https://MASKED@github.com/packsaddle/ruby-git_httpsable-push.git/'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/git-1.2.9.1/lib/git/lib.rb:727:in `push'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/git-1.2.9.1/lib/git/base.rb:329:in `push'
        from /Users/sane/work/ruby-study/git_httpsable-push/lib/git_httpsable/push/repository.rb:24:in `push'
        from /Users/sane/work/ruby-study/git_httpsable-push/lib/git_httpsable/push/cli.rb:29:in `push'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor/command.rb:27:in `run'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor/invocation.rb:126:in `invoke_command'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor.rb:359:in `dispatch'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor/base.rb:440:in `start'
        from /Users/sane/work/ruby-study/git_httpsable-push/lib/git_httpsable/push/cli.rb:59:in `method_missing'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor/command.rb:29:in `run'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor/command.rb:126:in `run'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor/invocation.rb:126:in `invoke_command'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor.rb:359:in `dispatch'
        from /Users/sane/work/ruby-study/git_httpsable-push/vendor/bundle/ruby/2.2.0/gems/thor-0.19.1/lib/thor/base.rb:440:in `start'
        from exe/git-httpsable-push:5:in `<main>'

{% endhighlight %}

ログとエラーの中の該当部分をマスクしている(MASKED部分)ので、ログに出ても安全。
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
