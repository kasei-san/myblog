title: Vagrant環境に作成したHexo環境から、Github Pages にアップロードしてみる
date: 2015-03-07 17:49:01
tags: Hexo Github
---

Vagrant環境に作成したHexo環境から、Github Pages にアップロードしてみる

<!-- more -->

## ホストマシンに git をインストール

```sh-session
$ knife cookbook create git -o cookbooks
```

{% codeblock cookbooks/git/recipes/default.rb %}
package 'git'
{% endcodeblock %}

```sh-session
$ vagrant provison
```

## hexo-deployer-git をインストール

```sh-session
$ vagrant ssh
$ cd /var/www/myblog/
$ npm install hexo-deployer-git --save
```

## 設定
 
{% codeblock _config.yml %}
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:kasei-san/hexo_server.git
{% endcodeblock %}

## vagrant で、ゲストの秘密鍵を使えるようにする

{% codeblock Vagrantfile type:ruby %}
  config.ssh.forward_agent = true
{% endcodeblock %}


## deploy

```sh-session
$ vagrant ssh -c "cd /var/www/myblog; hexo deploy"
```

upされた!!

http://kasei-san.github.io/myblog/

## けど、cssやら画像やらが正しく表示されていない...

あー、project の github page だからディレクティブが異なるからだ...
-> 独自ドメインを持たせることで対応できるはず

## ハマった所

hexo開発用のVagrant環境を作っていて、そこのサブディレクトリにhexoの作業ディレクトリがあり、それを1つのgitリポジトリで管理していたら
deploy する時に、hexo うまいこと github-page を作ってくれなかったので、hexoの作業ディレクトリを別リポジトリにした

## 参考

- [Deployment | Hexo](http://hexo.io/docs/deployment.html)
- [ssh-agentを使ってVagrant上のゲストOSからMac側の秘密鍵を使えるようにする | Firegoby](https://firegoby.jp/archives/5694)
- [チームブログをGitHubとHexoではじめよう！ | Tokyo Otaku Mode Blog](http://blog.otakumode.com/2014/08/08/Blogging-with-hexoio/)

