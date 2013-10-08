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

----------------------------------------------------------------------------------------------

以下、まだ途中なのです。  
ホストOSも混在しているのです。

----------------------------------------------------------------------------------------------

## knife-solo

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
