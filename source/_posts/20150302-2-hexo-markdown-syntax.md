title: hexo 記法色々メモ
date: 2015-03-02 10:00:00
tags:
- hexo
---

初めてページ作ってみてわかったこと色々メモ

<!-- more -->

## ページ作成方法

```sh-session
$ vagrant ssh 
$ cd /var/www/mypage
$ hexo new "Vagrant で hexo 環境構築"
[info] File created at /var/www/mypage/source/_posts/Vagrant-で-hexo-環境構築.md
```

基本的にはマークダウンで、ヘッダに title やら、tag やらを設定する

## 複数のタグを持たせたい場合は、yml の配列っぽい書き方を使う

{% codeblock lang:yaml %}
title: Vagrant で hexo 環境構築
date: 2015-03-02 06:45:17
tags:
- hexo
- vagrant
---
{% endcodeblock %}

## codeblock は、github記法だとファイル名が表示されない

hexo 記法にする

### 例

(実際は、{ と % の間を詰める)

{% codeblock lang:md %}
{ % codeblock Vagrantfile lang:ruby % }
...
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

こんな風に書く

```
{% asset_img 01.png %}
```

こんな風に表示される

{% asset_img 01.png %}


github記法で書くと、相対Pathになってしまうので、詳細ページ以外のディレクティブで表示できない

```.md
# これだと詳細ページ以外のディレクティブで表示できない!
![](01.png)
```

## 参考

- [【Hexo】Hexoにおける記事の書き方 | AdMax Tech Blog](http://tech.admax.ninja/2014/09/11/how-to-write-article-in-hexo/)
- [Tag Plugins | Hexo](http://hexo.io/docs/tag-plugins.html)
- [Asset Folders | Hexo](http://hexo.io/docs/asset-folders.html)
