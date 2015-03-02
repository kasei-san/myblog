title: hexo でページを作ってみる
date: 2015-03-02 10:00:00
tags:
- hexo
---

[Qiitaに書いた記事](http://qiita.com/kasei-san/items/55b075b302c6c7793396) をコピペしてみる

## ページ作成

```sh-session
$ vagrant ssh 
$ cd /var/www/mypage
$ hexo new "Vagrant で hexo 環境構築"
[info] File created at /var/www/mypage/source/_posts/Vagrant-で-hexo-環境構築.md
```

<!-- more -->

## 複数のタグを持たせたい場合は、yml の配列っぽい書き方を使う

{% codeblock lang:yaml %}
title: Vagrant で hexo 環境構築
date: 2015-03-02 06:45:17
tags:
- hexo
- vagrant
---
{% endcodeblock %}

## そのままだと、ファイル名が表示されないので、hexo 記法に変更

### 例

(実際は、{ と % の間を詰める)

{% codeblock lang:md %}
{ % codeblock Vagrantfile lang:ruby % }
## -*- mode: ruby -*-
## vi: set ft=ruby :

## All Vagrant configuration is done below. The "2" in Vagrant.configure
## configures the configuration version (we support older styles for
## backwards compatibility). Please don't change it unless you know what
## you're doing.
Vagrant.configure(2) do |config|
  config.vm.box = "chef/centos-7.0"
  config.vm.define :hexo_server do |hexo_server|
  end
end
{ % endcodeblock  % }
{% endcodeblock %}

## 一覧に全文でるのが鬱陶しい

以下のタグで区切られる

{% codeblock lang:md %}
<!-- more -->
{% endcodeblock %}

## 画像をupしたい

_config.yml の post_asset_folder を true にする

{% codeblock _config.yml lang:yaml %}
post_asset_folder: true
{% endcodeblock %}

その上で、hexo new すると、Asset Folder が生成される

```sh-session
$ new 20150302_2_hexo_markdown_syntax
 [info] File created at /var/www/mypage/source/_posts/20150302-2-hexo-markdown-syntax.md
$ ll source/_posts/
total 32
drwxr-xr-x  2 kasei_san  staff    68  3  2 17:17 20150302-2-hexo-markdown-syntax/
-rw-r--r--  1 kasei_san  staff  1965  3  2 17:24 20150302-2-hexo-markdown-syntax.md
```

画像を置く

```sh-session
/Users/kasei_san/work/hexo_server/mypage% ll source/_posts/20150302-2-hexo-markdown-syntax/
total 168
-rw-r--r--  1 kasei_san  staff  85933  3  2 17:30 01.png
```

こんな風に書ける

```
![テスト](01.png)
```

こんな風に表示される

![テスト](01.png)

ただ、このままだと相対pathなので、個別ページ以外だと正しく表示されないので、絶対PATHで書かなきゃいけないので面倒

```
![テスト](/2015/03/02/20150302-2-hexo-markdown-syntax/01.png)
```

## 参考

- [【Hexo】Hexoにおける記事の書き方 | AdMax Tech Blog](http://tech.admax.ninja/2014/09/11/how-to-write-article-in-hexo/)
- [Measure All The Things!!とHexoのタグプラグイン | masato's blog](http://masato.github.io/2014/06/27/measure-all-the-things-and-hexo-images/)
- [HexoのAsset | 研究開発日誌](http://www.graco.c.u-tokyo.ac.jp/~tody/blog/2014/12/20/hexo-asset/)

