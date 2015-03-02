title: Vagrant で hexo 環境構築
date: 2015-03-02 06:45:17
tags:
- hexo
- vagrant
---

つくってみた

## リポジトリ

- https://github.com/kasei-san/hexo_server

## 初手

{% codeblock Vagrantfile lang:ruby%}
$ git init
$ vagrant init
{% endcodeblock %}

<!-- more -->

## OSインストール

折角なので、最新のCentOSを入れてみる

### サーバ名が default のままだと、serverspec に失敗するので変更

参考 : [Carousel is a lie!: "Circular dependency detected" error in ServerSpec](https://saintaardvarkthecarpeted.com/blog/archive/2014/10/_Circular_dependency_detected__error_in_ServerSpec.html)

{% codeblock Vagrantfile lang:ruby%}
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
{% endcodeblock %}

```sh-session
$ vagrant up
```

### OSのバージョンを確認

```sh-session
$ vagrant ssh
Last login: Tue Jul 22 03:42:16 2014 from 10.0.2.2
[vagrant@localhost ~]$ cat /etc/redhat-release
CentOS Linux release 7.0.1406 (Core)
```

### .gitignore に /.vagrant を追加

```sh-session
echo  "/.vagrant/" >> .gitignore
```

## serverspec 導入

### bundle install

```sh-session
$ bundle init
$ cat <<EOS >> Gemfile
gem 'rake'
gem 'serverspec'
EOS
$ bundle --binstubs --path vendor/bundle
$ cat <<EOS >> .gitignore
/.bundle/
/vendor/bundle/
/bin
EOS
```

### serverspec-init

```sh-session
$ serverspec-init
Select OS type:

  1) UN*X
  2) Windows

Select number: 1

Select a backend type:

  1) SSH
  2) Exec (local)

Select number: 1

Vagrant instance y/n: y
Auto-configure Vagrant from Vagrantfile? y/n: y
 + spec/
 + spec/default/
 + spec/default/sample_spec.rb
 + spec/spec_helper.rb
 + Rakefile
 + .rspec
```

### テストが実行されることを確認

```sh-session
$ rake spec
```

テストは失敗する

## node.js インストール

### テスト実装

{% codeblock spec/hexo_server/node_spec.rb %}
require 'spec_helper'

describe package 'nodejs' do
  it { should be_installed.with_version('0.10') }
end

describe package 'npm' do
  it { should be_installed.with_version('1.3.6') }
end
{% endcodeblock %}

```sh-session
git rm spec/hexo_server/sample_spec.rb
```

```sh-session
$ rake spec
```

テストは失敗する

### vagrant-omnibus 導入

chef client を サーバに導入してくれる

プラグインを入れていなければインストール

```sh-session
$ vagrant plugin install vagrant-omnibus
```

{% codeblock Vagrantfile lang:ruby%}
Vagrant.configure(2) do |config|
  config.omnibus.chef_version = :latest
...
{% endcodeblock %}

```sh-session
$ vagrant provision
```

#### インストールされたことを確認

```sh-session
$ vagrant ssh
[vagrant@localhost ~]$ knife --version
Chef: 12.0.3
```

### chef 導入

```sh-session
$ echo "gem 'knife-solo'" >> Gemfile
$ bundle install
```

### epel リポジトリ導入

node.js がデフォルトのyumリポジトリにないので、EPELを追加する

    エンタープライズ Linux 用の拡張パッケージ(EPEL) は、 Red Hat Enterprise Linux (RHEL) 向けの高品質なアドオンパッケージ

https://fedoraproject.org/wiki/EPEL/ja

```sh-session
$ knife cookbook create epel_repo -o cookbooks
$ cd cookbooks/epel_repo
$ rm -rf CHANGELOG.md attributes/ definitions/ files/ libraries/ providers/ resources/ templates/
$ echo package \'epel-release\' > recipes/default.rb
```

### node.js 導入

```sh-session
$ knife cookbook create node -o cookbooks
$ cd cookbooks/node/
$ rm -rf CHANGELOG.md attributes/ definitions/ files/ libraries/ providers/ resources/ templates/
$ echo depends          \'epel_repo\' >> metadata.rb
$ cat <<EOS > recipes/default.rb
include_recipe 'apache'
package 'nodejs'
package 'npm'
EOS
```


### Vagrantfile に実行するレシピを追記

{% codeblock Vagrantfile lang:ruby%}
Vagrant.configure(2) do |config|
  config.omnibus.chef_version = :latest
  config.vm.box = "chef/centos-7.0"
  config.vm.define :hexo_server do |hexo_server|
    hexo_server.vm.provision :chef_solo do |chef|
      chef.cookbooks_path = './cookbooks'
      chef.log_level = 'debug'
      chef.add_recipe 'epel_repo'
      chef.add_recipe 'node'
    end
  end
end
{% endcodeblock %}

```
$ vagrant reload
```

### テスト実行

```
$ rake spec
```

今度は成功

### vagrant-cachier 導入

vagrant-cachier は、仮想マシン内でダウンロードしたパッケージをキャッシュしてくれるプラグイン
開発中に何度も vagrant destroy, vagrant up しているときに、その都度パッケージをダウンロードするのを防いで、
仮想マシンの構築を高速化してくれる

```sh-session
$ vagrant plugin install vagrant-cachier
```

{% codeblock Vagrantfile lang:ruby%}
Vagrant.configure(2) do |config|
...
  config.cache.scope = :box if Vagrant.has_plugin?("vagrant-cachier")
end
{% endcodeblock %}

## Hexo導入

インストールとinit

```sh-session
$ knife cookbook create hexo -o cookbooks
$ rm -rf attributes/ definitions/ files/ libraries/ providers/ resources/ templates/
$ echo depends \'node\' >> metadata.rb
$ cat <<EOF > recipes/default.rb
include_recipe 'node'
execute 'hexo install' do
  command 'npm install hexo-cli -g'
  not_if { File.exists?('/usr/bin/hexo') }
end

execute 'hexo init' do
  cwd '/var/www/mypage'
  command <<EOS
hexo init .
npm install
EOS
end
EOF
```

hexo用ディレクトリを synced_folder に
あと、ホストから見る時の為にIPアドレスを固定

{% codeblock Vagrantfile lang:ruby%}
Vagrant.configure(2) do |config|
...
  config.vm.define :hexo_server do |hexo_server|
    hexo_server.vm.provision :chef_solo do |chef|
...
       chef.add_recipe 'hexo'
     end
...
  config.vm.synced_folder 'mypage', '/var/www/mypage', create: true
  config.vm.network :private_network, ip: "192.168.33.10"
{% endcodeblock %}

## 動作確認

```sh-session
$ vagrant ssh
$ cd /var/www/mypage/
$ hexo server -i 192.168.33.10
[info] Hexo is running at http://192.168.33.10:4000/. Press Ctrl+C to stop.hexo server
```

http://192.168.33.10:4000/

![Hexo_-_Vimperator.png](https://qiita-image-store.s3.amazonaws.com/0/5329/4257f18d-b217-b324-62ee-8cd876395f51.png "Hexo_-_Vimperator.png")

見れたー!!

