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

.
├── Vagrantfile
├── chef-repo
│   ├── cookbooks
│   ├── data_bags
│   ├── nodes
│   ├── roles
│   └── site-cookbooks
└── nodes
    └── yunocchi.json

7 directories, 2 files
```

* ポイントは 0.3.0 以降からは `solo.rb` がいらないということ。
* 生成した Chefレポジトリ chef-solo はgitで管理する。

### cookbookを作る

```
knife cookbook create hello -o site-cookbooks

** Creating cookbook hello
** Creating README for cookbook: hello
** Creating CHANGELOG for cookbook: hello
** Creating metadata for cookbook: hello
```

* VMにcookbookを転送してchef-soloを実行する

```
knife solo cook yunocchi

Running Chef on yunocchi...
Checking Chef version...
ERROR: Network Error: Connection refused - connect(2)
Check your knife configuration and network settings
```

なにこれ。

- [Saheb's Blog: Check your knife configuration and network settings, unable to upload cookbooks](http://sahebjade.blogspot.jp/2013/05/check-your-knife-configuration-and.html)
- [Webサービスって作れるのか - わくぶろぐ](http://kazu-waku.hatenablog.com/entry/2013/07/28/004324)

```
ssh yunocchi
```

なんかしばらく触ってなかったからsshできないっぽい。  
しかもなんかポート番号変わってるでよ。  
configのyunocchiエントリー削除して再生成した。

```
vagrant ssh-config --host yunocchi | tee -a ~/.ssh/config

Host yunocchi
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/noguchiwataru/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL

```

もいっかい。

```
cd chef-repo/

knife solo cook yunocchi
Running Chef on yunocchi...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/yajl-ruby-1.1.0/lib/yajl.rb:36:in `parse': lexical error: invalid char in json text. (Yajl::ParseError)
                          {"run_list":[chef-repo::hello]} 
                     (right here) ------^
	from /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/yajl-ruby-1.1.0/lib/yajl.rb:36:in `parse'
	from /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/json_compat.rb:56:in `from_json'
	from /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/application/solo.rb:198:in `reconfigure'
	from /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/application.rb:64:in `run'
	from /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/bin/chef-solo:25:in `<top (required)>'
	from /usr/bin/chef-solo:23:in `load'
	from /usr/bin/chef-solo:23:in `<main>'
ERROR: RuntimeError: chef-solo failed. See output above.

```

- [chef-soloとknife-soloで手軽に環境構築をする - Bouldering & Com.](http://shrkw.hatenablog.com/entry/configure_with_chef-solo_and_knife-solo)

今いるディレクトリがまちがってるんじゃないか？

```
cd ..

knife solo cook yunocchi

Running Chef on yunocchi...
Checking Chef version...
Uploading the kitchen...
WARNING: Local cookbook_path '/var/chef/cookbooks' does not exist
WARNING: Local cookbook_path '/var/chef/site-cookbooks' does not exist
WARNING: Local role_path '/var/chef/roles' does not exist
WARNING: Local data_bag_path '/var/chef/data_bags' does not exist
Generating solo config...
Running Chef...
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 0 resources
Chef Client finished, 0 resources updated
```

[Chef Soloの正しい始め方 | tsuchikazu blog](http://tsuchikazu.net/chef_solo_start/)

```
#
# Cookbook Name:: hello
# Recipe:: default
#
# Copyright 2013, YOUR_COMPANY_NAME
#
# All rights reserved - Do Not Redistribute
#

log "hello, chef-solo."

package "httpd" do
  action :install
end

service "httpd" do
  supports :status => true, :restart => true, :reload => true
  action [ :enable, :start ]
end

service "iptables" do
  supports :status => true, :restart => true, :reload => true
  action [ :enable, :stop ]
end

```

```
[vagrant@localhost ~]$ sudo cat /var/chef/cache/chef-stacktrace.out
Generated at 2013-10-14 00:47:01 +0000
Chef::Exceptions::CookbookNotFound: Cookbook chef-repo not found. If you're loading chef-repo from another cookbook, make sure you configure the dependency in your metadata
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/cookbook/cookbook_collection.rb:38:in `block in initialize'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/ohai-6.18.0/lib/ohai/mash.rb:77:in `yield'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/ohai-6.18.0/lib/ohai/mash.rb:77:in `default'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/ohai-6.18.0/lib/ohai/mash.rb:77:in `default'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:265:in `[]'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:265:in `each_cookbook_dep'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:243:in `add_cookbook_with_deps'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:87:in `block in cookbook_order'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:85:in `each'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:85:in `cookbook_order'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:97:in `compile_libraries'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context/cookbook_compiler.rb:70:in `compile'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/run_context.rb:86:in `load'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/client.rb:249:in `setup_run_context'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/client.rb:492:in `do_run'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/client.rb:199:in `block in run'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/client.rb:193:in `fork'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/client.rb:193:in `run'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/application.rb:183:in `run_chef_client'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/application/solo.rb:239:in `block in run_application'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/application/solo.rb:231:in `loop'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/application/solo.rb:231:in `run_application'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/lib/chef/application.rb:66:in `run'
/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.6.0/bin/chef-solo:25:in `<top (required)>'
/usr/bin/chef-solo:23:in `load'
/usr/bin/chef-solo:23:in `<main>'[vagrant@localhost ~]$ 
```

* [chef (11.6.0) + knife-solo (0.3.0.pre5)は相性が悪いっぽい? - じゅにゃくんのはてブロ。](http://jun-ya.hatenablog.com/entry/2013/07/25/111756)

11.4にダウングレードしてみる。

```
source "https://rubygems.org"

gem 'knife-solo', '0.3.0'
gem 'chef', '11.4.4'
```

```
bundle install --path vendor/bundle

(snip)

Post-install message from knife-solo:
Thanks for installing knife-solo!

If you run into any issues please let us know at:
  https://github.com/matschaffer/knife-solo/issues

If you are upgrading knife-solo please uninstall any old versions by
running `gem clean knife-solo` to avoid any errors.

See http://bit.ly/CHEF-3255 for more information on the knife bug
that causes this.
```

```
knife configure

WARNING: No knife configuration file found
Where should I put the config file? [/Users/noguchiwataru/.chef/knife.rb] 
Please enter the chef server URL: [https://localhost:443] 
Please enter an existing username or clientname for the API: [noguchiwataru] 
Please enter the validation clientname: [chef-validator] 
Please enter the location of the validation key: [/etc/chef-server/chef-validator.pem] 
Please enter the path to a chef repository (or leave blank): 
*****

You must place your client key in:
  /Users/noguchiwataru/.chef/noguchiwataru.pem
Before running commands with Knife!

*****

You must place your validation key in:
  /etc/chef-server/chef-validator.pem
Before generating instance data with Knife!

*****
Configuration file written to /Users/noguchiwataru/.chef/knife.rb

```

```
cat <<EOF >>~/.chef/knife.rb
knife[:solo_path] = '/tmp/chef-solo'
EOF
```

```
bundle exec knife solo init chef-repo

Creating kitchen...
Creating knife.rb in kitchen...
Creating cupboards...

```

以降、`chef-repo`を起点ディレクトリとする。

```
cd chef-repo/
bundle exec knife cookbook create hello -o site-cookbooks

** Creating cookbook hello
** Creating README for cookbook: hello
** Creating CHANGELOG for cookbook: hello
** Creating metadata for cookbook: hello
```

```
# nodes/yunocchi.json
{"run_list":["hello::default"]}
```

```
bundle exec knife solo cook yunocchi

Running Chef on yunocchi...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 4 resources
Recipe: hello::default
  * log[hello, chef-solo.] action write

  * package[httpd] action install
    - install version 2.2.15-29.el6.centos of package httpd

  * service[httpd] action enable
    - enable service service[httpd]

  * service[httpd] action start
    - start service service[httpd]

  * service[iptables] action enable (up to date)
  * service[iptables] action stop
    - stop service service[iptables]

Chef Client finished, 5 resources updated
```

ふむ。

```
# nodes/yunocchi.json
{"run_list":["hello"]}
```

に変えてみた。動く。

`{"run_list":["hello"]}` とダブルクォートでくくらないで書くとエラーになる。  
書き方の問題か・・・。  
とりあえず動いた。よかった。

* [入門 Chef Solo 第17章 レシピ落ち穂拾い - run_list, ファイル分け, include_recipe - 毎朝30分読書会](http://d.hatena.ne.jp/morning_reading/20130806/p1)

### nginxの立ち上げ

* クックブック作成

```
bundle exec knife cookbook create nginx -o site-cookbooks
```

* `nodes/yunocchi.json` の編集

nginxクックブックを実行するように指定。

```
{"run_list":["nginx"]}
```

* レシピの編集

```
package "nginx" do
  action :install
end

service "nginx" do
  supports :status => true, :restart => true, :reload => true
  action [ :enable, :start ]
end

service "iptables" do
  supports :status => true, :restart => true, :reload => true
  action [ :enable, :stop ]
end
```

* chef-solo実行

knife solo経由で。

```
bundle exec knife solo cook yunocchi
Running Chef on yunocchi...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 3 resources
Recipe: nginx::default
  * package[nginx] action install
    * No version specified, and no candidate version available for nginx
================================================================================
Error executing action `install` on resource 'package[nginx]'
================================================================================


Chef::Exceptions::Package
-------------------------
No version specified, and no candidate version available for nginx


Resource Declaration:
---------------------
# In /home/vagrant/chef-solo/cookbooks-2/nginx/recipes/default.rb

 10: package "nginx" do
 11:   action :install
 12: end
 13: 



Compiled Resource:
------------------
# Declared in /home/vagrant/chef-solo/cookbooks-2/nginx/recipes/default.rb:10:in `from_file'

package("nginx") do
  action [:install]
  retries 0
  retry_delay 2
  package_name "nginx"
  cookbook_name :nginx
  recipe_name "default"
end



[2013-10-14T13:45:08+00:00] ERROR: Running exception handlers
[2013-10-14T13:45:08+00:00] ERROR: Exception handlers complete
[2013-10-14T13:45:08+00:00] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
Chef Client failed. 0 resources updated
[2013-10-14T13:45:08+00:00] FATAL: Chef::Exceptions::ChildConvergeError: Chef run process exited unsuccessfully (exit code 1)
ERROR: RuntimeError: chef-solo failed. See output above.
```

Amazon Linuxじゃないときつめか。

## 参考サイト

* [Chef Soloの正しい始め方 | tsuchikazu blog](http://tsuchikazu.net/chef_solo_start/)
