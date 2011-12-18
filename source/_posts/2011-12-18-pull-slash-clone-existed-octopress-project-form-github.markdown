---
layout: post
title: "從現有的 Octopress repo 架構相同的寫作環境"
date: 2011-12-18 14:57
comments: true
categories:
- Octopress
- Ruby
---
# 前言 #

用 Octopress 寫作，其中一個重要的動機就是直接把部落格內容與版本控制整合。這篇文章的重點，即在當某一 Octopress repository 已存在的情況下，如何在另一個空白的環境中建構相同的寫作環境。

這篇的內容主要參考[這篇](http://huanggang.me/archives/667)，遠端的 repo host 則是 GitHub。

# 讓我們開始做這件事 #

## Step 1. 完全空白的 Ruby 開發環境 ##

Octopress 用 Ruby 開發，當然也要先裝 Ruby 才能用。無論如何，先把 RVM 裝起來吧！當然，如果你已經裝過 Ruby，可以先跳過這部份。

[http://beginrescueend.com/](http://beginrescueend.com/)

在寫下這篇的時間點，Octopress 是基於 Ruby 1.9.2 開發的，於是乎：

{% codeblock %}
rvm install 1.9.2
{% endcodeblock %}

如果不是 1.9.2，可能要用下面這個指令查一下哪個版本才對：

{% codeblock %}
rvm list known
{% endcodeblock %}

## Step 2. 從遠端拉回 repository ##

從 GitHub 上把我們的 Octopress 中的 ``source`` branch 拉下來裝。記的把下面的 ``myname`` 改成自己的名字啊！

{% codeblock %}
git clone -b source git@github.com:myname/myname.github.com.git myblog
{% endcodeblock %}

然後把 main branch 拉到 ``_deploy`` 下面

{% codeblock %}
cd myblog
git clone git@github.com:myname/myname.github.com.git _deploy
{% endcodeblock %}

## Step 3. 用 gem 裝回所需要的套件們 ##

一樣在你拉回的 repo 根目錄下（即例中的 ``myblog``）

{% codeblock %}
gem install bundler
bundle install
rake install          # install the default Octopress theme
{% endcodeblock %}

**完工！**開始快樂地寫作吧！

# 後記 #

其實整理這篇的主要用意，是因為我自己並非 Ruby programmer，對於 Ruby 界常用的工具不熟悉 :p

本以為只要簡單地 git clone 就能解決，卻也因此學習了 gem、bundler 這些工具。用起來真是舒服啊！真是羨慕 Ruby 的開發者們…
