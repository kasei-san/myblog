title: Vagrant で hexo 3 環境構築
date: 2015-03-02 06:45:17
tags:
- hexo
- vagrant
---

[こっち](http://qiita.com/kasei-san/items/55b075b302c6c7793396) でVagrant環境にHexoを導入する手順を書いたけど、
インストールされるのが古いHexoだったので、差分的にHexo3の導入に必要な事項をメモ

<!-- more -->

## リポジトリ

- https://github.com/kasei-san/hexo_server

## npm install

[公式](http://hexo.io/)にあるように、hexo-cli をインストールする

{% codeblock cookbooks/hexo/recipes/default.rb %}
execute 'hexo install' do
  command 'npm install -g hexo-cli'
  not_if { File.exists?('/usr/bin/hexo') }
end
{% endcodeblock %}

## Synced folderにrsyncを使用する

デフォルト(virtualbox) だと、ファイルの更新を認識してくれなかったので、rsyncを使用する

{% codeblock Vagrantfile %}
  config.vm.synced_folder 'mypage', '/var/www/mypage',
    create: true,
    type: 'rsync'
{% endcodeblock %}

### vagrant rsync-auto で、更新を監視

```sh-session
$ vagrant rsync-auto
```
