# chef-solo + Vagrant on Mac OS X

Mac OS XをホストOSとしてchef-solo。

## Getting Started

### chef-soloインストール

```
curl -L http://www.opscode.com/chef/install.sh | sudo bash
Password:  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6789  100  6789    0     0  12087      0 --:--:-- --:--:-- --:--:-- 20085

Downloading Chef  for mac_os_x...
.......................................................................................................................................................................................................................................................................................................
Thank you for installing Chef!
```

最初躓いたんだけど、「おいおい、chefインストールできないじゃん」って思ってたらこれ、最初にsudoかましてるところでcurlの出力にPasswordのプロンプト出てたし。。。パスワード入力してsudo通すようにしたらインストール出来ました。

### Knifeのお手入れ

切れ味を良くしましょう。  
初回のみでOK。

```
knife configure
```

### レポジトリの作成

```
git clone git://github.com/opscode/chef-repo.git
```

して、

```
cd chef-repo/
knife cookbook create hello -o cookbooks

** Creating cookbook hello
** Creating README for cookbook: hello
** Creating CHANGELOG for cookbook: hello
** Creating metadata for cookbook: hello
```

### レシピ編集

```ruby
# cookbooks/hello/recipes/default.rb
#
# Cookbook Name:: hello
# Recipe:: default
#
# Copyright 2013, YOUR_COMPANY_NAME
#
# All rights reserved - Do Not Redistribute
#

log "Hello, Chef!"
```

### 実行するレシピの一覧の定義

```json
// localhost.json
{
  "run_list": [
    "recipe[hello]"
  ]
}

```

### chef-soloの設定

```ruby
# solo.rb
file_cache_path "/tmp/chef-solo"
cookbook_path [ "/home/wnoguchi/chef-repo/cookbooks" ]
```

### 実行

```
sudo chef-solo -c solo.rb -j ./localhost.json

Starting Chef Client, version 11.6.0
Unable to find any JVMs matching version "(null)".
No Java runtime present, try --request to install.
Compiling Cookbooks...
Converging 1 resources
Recipe: hello::default
  * log[Hello, Chef!] action write

Chef Client finished, 1 resources updated
```

Hello, Chef!は無事表示されたけど、

```
Unable to find any JVMs matching version "(null)".
No Java runtime present, try --request to install.
```

なんじゃこのメッセージは。。。

- [OSX Mountain Lion and Java | The Syncing Apple](http://dillernet.com/apple/2012/07/27/osx-mountain-lion-and-java/)

jdkがないのが原因らしい。

```
javac
```

って打ったらなんかダイアログ出てきて、自動的にインストールできた。  
chefは内部的にjdk使ってるのか？  
今度はエラーメッセージ表示されなくなったのでモーマンタイ。

```
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 1 resources
Recipe: hello::default
  * log[Hello, Chef!] action write

Chef Client finished, 1 resources updated
```

chef-soloで僕のmac環境を壊すのは嫌なので、Vagrantで立ち上げたVMにchef-soloを入れようと思う。  
そうしよう。

## Vagrant導入

http://www.vagrantbox.es/

気軽に仮想マシンを立ち上げて希望の環境を作るための最高の環境。  
構成ファイル `Vagrantfile` とかで仮想マシンのスペックを記述して、  
コマンドで数発叩いてイニシャルな状態の仮想マシンが立ち上げ、停止、削除が気軽にできる。  
さらにはchef-soloとの合わせ技で起動した仮想マシンの構成管理までできてしまう。  
エンジニアのスキルによらず冪等性があり、再現性の高い仮想マシンを構築することができる。  
動いた動かないの話が少なくなる。

Vagrantのインストールの仕方は忘れた。  
何か特別なこと考えなくてもインストールできます。  
基本的にVirtualBox必須。  
というかそれ以外のやり方知らない。

* boxファイル追加

```
# CentOS 6.3
vagrant box add base http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.3-x86_64-v20130101.box
（...とても時間がかかる）

# CentOS 6.4
vagrant box add base http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130731.box
```

* Vagrantファイルその他作成

```
mkdir vagrant1
cd vagrant1
vagrant init

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

* ホストオンリーネットワークの記述

```ruby
# Vagrantfile
Vagrant::Config.run do |config|
  config.vm.box = "base"
#(snip)
  config.vm.network :private_network, ip: "192.168.50.12"
#(snip)
```

* 仮想マシン起動

```
vagrant up

Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'base'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Fixed port collision for 22 => 2222. Now on port 2200.
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2200 (adapter 1)
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
[default] Machine booted and ready!
[default] Configuring and enabling network interfaces...
[default] Mounting shared folders...
[default] -- /vagrant
```

* SSH

```
vagrant ssh

Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$ cat /etc/redhat-release 
CentOS release 6.4 (Final)
[vagrant@localhost ~]$ exit
logout
Connection to 127.0.0.1 closed.
```

あるいはSSHターミナルで普通にIPアドレス叩いてつながります。

- ID: `root`
- PASS: `vagrant`

あるいはvagrantユーザー上でパス無し `sudo -i` ができます。

* SSHアクセス設定をする

秘密鍵を設定する。  
`~/.ssh/config` を設定するんだけど、めんどくさいのでコマンドで流し込む。  
teeコマンドでどんな内容が追記されたのか一応確認。  
`-a` オプション付けないとconfigファイルが上書きされて無く事になるので注意。

`--host` オプションにはアクセスしたいこのVagrant VMの好きなホスト名を指定します。  
以下は `yunocchi` でアクセスできる。  
以下の設定ファイルを見るととても興味深くて、UserKnownHostsFileに`/dev/null`を指定していて  
Warningが出るのを防いでいるところ。  
実験用のVMだからフィンガープリントは変化しまくるのを見越してのことでしょう。

```
vagrant ssh-config --host yunocchi | tee -a ~/.ssh/config

Host yunocchi
  HostName 127.0.0.1
  User vagrant
  Port 2200
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/noguchiwataru/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

* つないでみる

```
localhost:vagrant1 noguchiwataru$ ssh yunocchi
Last login: Wed Oct  9 14:28:13 2013 from 10.0.2.2
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$ 
```

* 停止

```
vagrant halt

[default] Attempting graceful shutdown of VM...
```

* 破壊

```
vagrant destroy

Are you sure you want to destroy the 'default' VM? [y/N] y
[default] Destroying VM and associated drives...
```

### sahara でスナップショット、ロールバック

* インストール

```
>vagrant plugin install sahara

Installing the 'sahara' plugin. This can take a few minutes...
Installed the plugin 'sahara (0.0.15)'!
```

* sandboxモードを有効にする

この時点が起点となる。

```
vagrant up

vagrant sandbox on

0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
```

* 何か作業する

ためしにApacheでもいれてみよう。

```
service iptables stop
yum -y install httpd
service httpd start
```

* ロールバックする

よし、気に入らないから戻そう！

```
>vagrant sandbox rollback

0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
```

おーすごーい。

* コミット

気に入った設定になったらコミットします。状態を確定する操作です。  
これ、なんか重いんだよね。。。どうにかなんないのかしら。

```
>vagrant sandbox commit

>vagrant sandbox off
```

## knife-solo + Vagrantの連携

### knife-soloのインストール

0.3.0系推奨。

```
gem install knife-solo --no-ri --no-rdoc

gem list | grep knife-solo

knife-solo (0.3.0)
```

### VMにchef-solo環境を構築

```
knife solo prepare yunocchi

Bootstrapping Chef...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
101  6790  101  6790    0     0   1110      0  0:00:06  0:00:06 --:--:-- 18501
Downloading Chef 11.6.0 for el...
Installing Chef 11.6.0
警告: /tmp/tmp.7nhqIjwd/chef-11.6.0.x86_64.rpm: ヘッダ V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
準備中...                ########################################### [100%]
	パッケージ chef-11.6.0-1.el6.x86_64 は既にインストールされています。
Generating node config 'nodes/yunocchi.json'...
```

既にインストールされてるみたい。

### 新規Chefレポジトリを作成

```
knife solo init chef-repo

Creating kitchen...
Creating knife.rb in kitchen...
Creating cupboards...
```

----------------------------------------------------------------------------------------------

以下、まだ途中なのです。  
ホストOSも混在しているのです。

----------------------------------------------------------------------------------------------

```
sudo gem install knife-solo --no-ri --no-rdoc

Building native extensions.  This could take a while...
ERROR:  Error installing knife-solo:
	ERROR: Failed to build gem native extension.

        /usr/bin/ruby1.9.1 extconf.rb
/usr/lib/ruby/1.9.1/rubygems/custom_require.rb:36:in `require': cannot load such file -- mkmf (LoadError)
	from /usr/lib/ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
	from extconf.rb:1:in `<main>'



sudo apt-get -y install ruby-dev

sudo gem install knife-solo --no-ri --no-rdoc


gem list knife-solo

*** LOCAL GEMS ***

knife-solo (0.3.0)


vi ~/.chef/knife.rb

knife[:solo_path] = '/tmp/chef-solo'


```

### chef ready!

冪等性ヤッホウ。

#### レポジトリ作成

```
bundle exec knife solo init chef-repo

Creating kitchen...
Creating knife.rb in kitchen...
Creating cupboards...

ls -F chef-repo/
cookbooks/	data_bags/	nodes/		roles/		site-cookbooks/

cd chef-repo
git init && git add -A && git commit -m "Initial commit."
```

#### cookbook作成

helloというcookbookを作る。

```
bundle exec knife cookbook create hello -o site-cookbooks

** Creating cookbook hello
** Creating README for cookbook: hello
** Creating CHANGELOG for cookbook: hello
** Creating metadata for cookbook: hello

localhost:chef-repo noguchiwataru$ git add -A
localhost:chef-repo noguchiwataru$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   site-cookbooks/hello/CHANGELOG.md
#	new file:   site-cookbooks/hello/README.md
#	new file:   site-cookbooks/hello/metadata.rb
#	new file:   site-cookbooks/hello/recipes/default.rb
#



localhost:chef-repo noguchiwataru$ ls -F1 | cut -f14 -d " "
cookbooks/
data_bags/
nodes/
roles/
site-cookbooks/

```

それじゃ一発、Chef Solo Ready!にしてみますかぁ。

```
localhost:chef-repo noguchiwataru$ bundle exec knife solo prepare melody
Bootstrapping Chef...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
101  6790  101  6790    0     0    952      0  0:00:07  0:00:07 --:--:-- 20268
Downloading Chef 11.6.0 for el...
Installing Chef 11.6.0
warning: /tmp/tmp.juXVZSpi/chef-11.6.0.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
Preparing...                ########################################### [100%]
	package chef-11.6.0-1.el6.x86_64 is already installed
Generating node config 'nodes/melody.json'...

```

ちょっと燃料切れ。  
Vagrantで立ち上げたマシンにknife solo cookするのは次の日にしよう。
