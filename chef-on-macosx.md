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

確認がうざいときは `-f` オプションをつける。

```
vagrant destroy -f
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
```

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

CentOSだとうまくいかない。  
Amazon Linuxじゃないときつめか。

## Opscode Communityのクックボックをインポートする

膨大なクックブックの集積を利用する。

サインアップして秘密鍵を取得。パーミッションは600に設定。

```
cat <<EOF >>~/.chef/knife.rb
client_key '~/Dropbox/chef/wnoguchi.pem'
cookbook_path ['./cookbooks']
EOF
```

* yumクックブックを使ってみる

リポジトリはバージョン管理下にあり、ワークツリーはクリーンな状態であるとする。

```
bundle exec knife solo init chef-repo
cd chef-repo/
git init && git add -A && git commit -m "Initial commit."
```

```
bundle exec knife cookbook site vendor yum

Removing pre-existing version.
Uncompressing yum version 2.3.4.
removing downloaded tarball
No changes made to yum
Checking out the master branch.
```

yumのクックブックは入ったみたいだけど、自動的にコミットは行われていないように見えるなり。  
`.gitignore` を見ると `/cookbook/` 以下がまるごと無視されるいるようです。

```
.
├── cookbooks
│   └── yum
│       ├── CHANGELOG.md
│       ├── README.md
│       ├── attributes
│       │   ├── default.rb
│       │   ├── elrepo.rb
│       │   ├── epel.rb
│       │   └── remi.rb
│       ├── files
│       │   └── default
│       │       └── tests
│       │           └── minitest
│       │               ├── default_test.rb
│       │               ├── support
│       │               │   └── helpers.rb
│       │               └── test_test.rb
│       ├── metadata.json
│       ├── metadata.rb
│       ├── providers
│       │   ├── key.rb
│       │   └── repository.rb
│       ├── recipes
│       │   ├── default.rb
│       │   ├── elrepo.rb
│       │   ├── epel.rb
│       │   ├── ius.rb
│       │   ├── remi.rb
│       │   ├── repoforge.rb
│       │   ├── test.rb
│       │   └── yum.rb
│       ├── resources
│       │   ├── key.rb
│       │   └── repository.rb
│       └── templates
│           └── default
│               ├── repo.erb
│               ├── yum-rhel5.conf.erb
│               └── yum-rhel6.conf.erb
├── data_bags
├── nodes
├── roles
└── site-cookbooks

17 directories, 26 files

```

* epelを有効にする

```
{"run_list":["yum::epel"]}
```

* 料理する

```
Running Chef on yunocchi...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
Starting Chef Client, version 11.6.2
Compiling Cookbooks...
Converging 2 resources
Recipe: yum::epel
  * yum_key[RPM-GPG-KEY-EPEL-6] action add (up to date)
Recipe: <Dynamically Defined Resource>
  * package[gnupg2] action install (up to date)
  * execute[import-rpm-gpg-key-RPM-GPG-KEY-EPEL-6] action nothing (skipped due to action :nothing)
  * remote_file[/etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6] action create
    - create new file /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    - update content in file /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6 from none to 626e18
        --- /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6	2013-10-15 15:27:44.459652833 +0000
        +++ /tmp/chef-rest20131015-3515-1qe13r7	2013-10-15 15:27:50.111818943 +0000
        @@ -0,0 +1,29 @@
        +-----BEGIN PGP PUBLIC KEY BLOCK-----
        +Version: GnuPG v1.4.5 (GNU/Linux)
        +
        +mQINBEvSKUIBEADLGnUj24ZVKW7liFN/JA5CgtzlNnKs7sBg7fVbNWryiE3URbn1
        +JXvrdwHtkKyY96/ifZ1Ld3lE2gOF61bGZ2CWwJNee76Sp9Z+isP8RQXbG5jwj/4B
        +M9HK7phktqFVJ8VbY2jfTjcfxRvGM8YBwXF8hx0CDZURAjvf1xRSQJ7iAo58qcHn
        +XtxOAvQmAbR9z6Q/h/D+Y/PhoIJp1OV4VNHCbCs9M7HUVBpgC53PDcTUQuwcgeY6
        +pQgo9eT1eLNSZVrJ5Bctivl1UcD6P6CIGkkeT2gNhqindRPngUXGXW7Qzoefe+fV
        +QqJSm7Tq2q9oqVZ46J964waCRItRySpuW5dxZO34WM6wsw2BP2MlACbH4l3luqtp
        +Xo3Bvfnk+HAFH3HcMuwdaulxv7zYKXCfNoSfgrpEfo2Ex4Im/I3WdtwME/Gbnwdq
        +3VJzgAxLVFhczDHwNkjmIdPAlNJ9/ixRjip4dgZtW8VcBCrNoL+LhDrIfjvnLdRu
        +vBHy9P3sCF7FZycaHlMWP6RiLtHnEMGcbZ8QpQHi2dReU1wyr9QgguGU+jqSXYar
        +1yEcsdRGasppNIZ8+Qawbm/a4doT10TEtPArhSoHlwbvqTDYjtfV92lC/2iwgO6g
        +YgG9XrO4V8dV39Ffm7oLFfvTbg5mv4Q/E6AWo/gkjmtxkculbyAvjFtYAQARAQAB
        +tCFFUEVMICg2KSA8ZXBlbEBmZWRvcmFwcm9qZWN0Lm9yZz6JAjYEEwECACAFAkvS
        +KUICGw8GCwkIBwMCBBUCCAMEFgIDAQIeAQIXgAAKCRA7Sd8qBgi4lR/GD/wLGPv9
        +qO39eyb9NlrwfKdUEo1tHxKdrhNz+XYrO4yVDTBZRPSuvL2yaoeSIhQOKhNPfEgT
        +9mdsbsgcfmoHxmGVcn+lbheWsSvcgrXuz0gLt8TGGKGGROAoLXpuUsb1HNtKEOwP
        +Q4z1uQ2nOz5hLRyDOV0I2LwYV8BjGIjBKUMFEUxFTsL7XOZkrAg/WbTH2PW3hrfS
        +WtcRA7EYonI3B80d39ffws7SmyKbS5PmZjqOPuTvV2F0tMhKIhncBwoojWZPExft
        +HpKhzKVh8fdDO/3P1y1Fk3Cin8UbCO9MWMFNR27fVzCANlEPljsHA+3Ez4F7uboF
        +p0OOEov4Yyi4BEbgqZnthTG4ub9nyiupIZ3ckPHr3nVcDUGcL6lQD/nkmNVIeLYP
        +x1uHPOSlWfuojAYgzRH6LL7Idg4FHHBA0to7FW8dQXFIOyNiJFAOT2j8P5+tVdq8
        +wB0PDSH8yRpn4HdJ9RYquau4OkjluxOWf0uRaS//SUcCZh+1/KBEOmcvBHYRZA5J
        +l/nakCgxGb2paQOzqqpOcHKvlyLuzO5uybMXaipLExTGJXBlXrbbASfXa/yGYSAG
        +iVrGz9CE6676dMlm8F+s3XXE13QZrXmjloc6jwOljnfAkjTGXjiB7OULESed96MR
        +XtfLk0W5Ab9pd7tKDR6QHI7rgHXfCopRnZ2VVQ==
        +=V/6I
        +-----END PGP PUBLIC KEY BLOCK-----
    - change mode from '' to '0644'

  * execute[import-rpm-gpg-key-RPM-GPG-KEY-EPEL-6] action run
    - execute rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

Recipe: yum::epel
  * yum_repository[epel] action add (up to date)
Recipe: <Dynamically Defined Resource>
  * yum_key[epel-key] action add (up to date)
  * execute[yum-makecache-epel] action nothing (skipped due to action :nothing)
  * ruby_block[reload-internal-yum-cache-for-epel] action nothing (skipped due to action :nothing)
  * template[/etc/yum.repos.d/epel.repo] action create
    - create new file /etc/yum.repos.d/epel.repo
    - update content in file /etc/yum.repos.d/epel.repo from none to 18fb55
        --- /etc/yum.repos.d/epel.repo	2013-10-15 15:27:50.316722833 +0000
        +++ /tmp/chef-rendered-template20131015-3515-1nutqve	2013-10-15 15:27:50.318721833 +0000
        @@ -0,0 +1,8 @@
        +# Generated by Chef for localhost
        +# Local modifications will be overwritten.
        +[epel]
        +name=Extra Packages for Enterprise Linux
        +mirrorlist=http://mirrors.fedoraproject.org/mirrorlist?repo=epel-6&arch=$basearch
        +gpgcheck=1
        +gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
        +enabled=1
    - change mode from '' to '0644'

  * execute[yum-makecache-epel] action run
    - execute yum -q makecache --disablerepo=* --enablerepo=epel

  * ruby_block[reload-internal-yum-cache-for-epel] action create
    - execute the ruby block reload-internal-yum-cache-for-epel

Chef Client finished, 5 resources updated
```

epelが入ったかどうか確認する。

```
noguchiwataru-no-MacBook-Air:vagrant1 noguchiwataru$ vagrant ssh
Last login: Tue Oct 15 15:27:42 2013 from 10.0.2.2
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$ sudo yum repolist
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.fairway.ne.jp
 * epel: ftp.kddilabs.jp
 * extras: mirror.fairway.ne.jp
 * updates: mirror.fairway.ne.jp
repo id                           repo name                                                      status
base                              CentOS-6 - Base                                                6,381
epel                              Extra Packages for Enterprise Linux                            9,789
extras                            CentOS-6 - Extras                                                 13
updates                           CentOS-6 - Updates                                             1,367
repolist: 17,550

```

入ってるね。

じゃあさっき失敗したnginxのクックブックももう一回トライしてみるか。

```
bundle exec knife cookbook create nginx -o site-cookbooks
cat <<EOF >>site-cookbooks/nginx/recipes/default.rb
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
EOF
```

```
// nodes/yunocchi.json
{"run_list":["yum::epel", "nginx"]}
```

```
bundle exec knife solo cook yunocchi

Running Chef on yunocchi...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
Starting Chef Client, version 11.6.2
Compiling Cookbooks...
Converging 5 resources
Recipe: yum::epel
  * yum_key[RPM-GPG-KEY-EPEL-6] action add (up to date)
  * yum_repository[epel] action add (up to date)
Recipe: nginx::default
  * package[nginx] action install
    - install version 1.0.15-5.el6 of package nginx

  * service[nginx] action enable
    - enable service service[nginx]

  * service[nginx] action start
================================================================================
Error executing action `start` on resource 'service[nginx]'
================================================================================


Mixlib::ShellOut::ShellCommandFailed
------------------------------------
Expected process to exit with [0], but received '1'
---- Begin output of /sbin/service nginx start ----
STDOUT: Starting nginx: [FAILED]
STDERR: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] still could not bind()
---- End output of /sbin/service nginx start ----
Ran /sbin/service nginx start returned 1


Resource Declaration:
---------------------
# In /home/vagrant/chef-solo/cookbooks-2/nginx/recipes/default.rb

 13: service "nginx" do
 14:   supports :status => true, :restart => true, :reload => true
 15:   action [ :enable, :start ]
 16: end
 17: 



Compiled Resource:
------------------
# Declared in /home/vagrant/chef-solo/cookbooks-2/nginx/recipes/default.rb:13:in `from_file'

service("nginx") do
  action [:enable, :start]
  updated true
  supports {:status=>true, :restart=>true, :reload=>true}
  retries 0
  retry_delay 2
  service_name "nginx"
  enabled true
  pattern "nginx"
  startup_type :automatic
  cookbook_name :nginx
  recipe_name "default"
end



[2013-10-15T15:37:32+00:00] ERROR: Running exception handlers
[2013-10-15T15:37:32+00:00] ERROR: Exception handlers complete
[2013-10-15T15:37:32+00:00] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
Chef Client failed. 2 resources updated
[2013-10-15T15:37:32+00:00] FATAL: Chef::Exceptions::ChildConvergeError: Chef run process exited unsuccessfully (exit code 1)
ERROR: RuntimeError: chef-solo failed. See output above.
```

うっ、おかしくなった。
ログを見るにApacheとリッスンしてるポートが干渉してる気がする。

やりなおし、saharaでスナップショットとってロールバックできるようにしてやってみる。  
こういうとき便利だなあ。

```
vagrant sandbox on
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%


bundle exec knife solo cook yunocchi

Running Chef on yunocchi...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
Starting Chef Client, version 11.6.2
Compiling Cookbooks...
Converging 5 resources
Recipe: yum::epel
  * yum_key[RPM-GPG-KEY-EPEL-6] action add (up to date)
Recipe: <Dynamically Defined Resource>
  * package[gnupg2] action install (up to date)
  * execute[import-rpm-gpg-key-RPM-GPG-KEY-EPEL-6] action nothing (skipped due to action :nothing)
  * remote_file[/etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6] action create
    - create new file /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    - update content in file /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6 from none to 626e18
        --- /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6	2013-10-15 15:44:25.078250177 +0000
        +++ /tmp/chef-rest20131015-2525-1xejhrx	2013-10-15 15:44:30.770094755 +0000
        @@ -0,0 +1,29 @@
        +-----BEGIN PGP PUBLIC KEY BLOCK-----
        +Version: GnuPG v1.4.5 (GNU/Linux)
        +
        +mQINBEvSKUIBEADLGnUj24ZVKW7liFN/JA5CgtzlNnKs7sBg7fVbNWryiE3URbn1
        +JXvrdwHtkKyY96/ifZ1Ld3lE2gOF61bGZ2CWwJNee76Sp9Z+isP8RQXbG5jwj/4B
        +M9HK7phktqFVJ8VbY2jfTjcfxRvGM8YBwXF8hx0CDZURAjvf1xRSQJ7iAo58qcHn
        +XtxOAvQmAbR9z6Q/h/D+Y/PhoIJp1OV4VNHCbCs9M7HUVBpgC53PDcTUQuwcgeY6
        +pQgo9eT1eLNSZVrJ5Bctivl1UcD6P6CIGkkeT2gNhqindRPngUXGXW7Qzoefe+fV
        +QqJSm7Tq2q9oqVZ46J964waCRItRySpuW5dxZO34WM6wsw2BP2MlACbH4l3luqtp
        +Xo3Bvfnk+HAFH3HcMuwdaulxv7zYKXCfNoSfgrpEfo2Ex4Im/I3WdtwME/Gbnwdq
        +3VJzgAxLVFhczDHwNkjmIdPAlNJ9/ixRjip4dgZtW8VcBCrNoL+LhDrIfjvnLdRu
        +vBHy9P3sCF7FZycaHlMWP6RiLtHnEMGcbZ8QpQHi2dReU1wyr9QgguGU+jqSXYar
        +1yEcsdRGasppNIZ8+Qawbm/a4doT10TEtPArhSoHlwbvqTDYjtfV92lC/2iwgO6g
        +YgG9XrO4V8dV39Ffm7oLFfvTbg5mv4Q/E6AWo/gkjmtxkculbyAvjFtYAQARAQAB
        +tCFFUEVMICg2KSA8ZXBlbEBmZWRvcmFwcm9qZWN0Lm9yZz6JAjYEEwECACAFAkvS
        +KUICGw8GCwkIBwMCBBUCCAMEFgIDAQIeAQIXgAAKCRA7Sd8qBgi4lR/GD/wLGPv9
        +qO39eyb9NlrwfKdUEo1tHxKdrhNz+XYrO4yVDTBZRPSuvL2yaoeSIhQOKhNPfEgT
        +9mdsbsgcfmoHxmGVcn+lbheWsSvcgrXuz0gLt8TGGKGGROAoLXpuUsb1HNtKEOwP
        +Q4z1uQ2nOz5hLRyDOV0I2LwYV8BjGIjBKUMFEUxFTsL7XOZkrAg/WbTH2PW3hrfS
        +WtcRA7EYonI3B80d39ffws7SmyKbS5PmZjqOPuTvV2F0tMhKIhncBwoojWZPExft
        +HpKhzKVh8fdDO/3P1y1Fk3Cin8UbCO9MWMFNR27fVzCANlEPljsHA+3Ez4F7uboF
        +p0OOEov4Yyi4BEbgqZnthTG4ub9nyiupIZ3ckPHr3nVcDUGcL6lQD/nkmNVIeLYP
        +x1uHPOSlWfuojAYgzRH6LL7Idg4FHHBA0to7FW8dQXFIOyNiJFAOT2j8P5+tVdq8
        +wB0PDSH8yRpn4HdJ9RYquau4OkjluxOWf0uRaS//SUcCZh+1/KBEOmcvBHYRZA5J
        +l/nakCgxGb2paQOzqqpOcHKvlyLuzO5uybMXaipLExTGJXBlXrbbASfXa/yGYSAG
        +iVrGz9CE6676dMlm8F+s3XXE13QZrXmjloc6jwOljnfAkjTGXjiB7OULESed96MR
        +XtfLk0W5Ab9pd7tKDR6QHI7rgHXfCopRnZ2VVQ==
        +=V/6I
        +-----END PGP PUBLIC KEY BLOCK-----
    - change mode from '' to '0644'

  * execute[import-rpm-gpg-key-RPM-GPG-KEY-EPEL-6] action run
    - execute rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

Recipe: yum::epel
  * yum_repository[epel] action add (up to date)
Recipe: <Dynamically Defined Resource>
  * yum_key[epel-key] action add (up to date)
  * execute[yum-makecache-epel] action nothing (skipped due to action :nothing)
  * ruby_block[reload-internal-yum-cache-for-epel] action nothing (skipped due to action :nothing)
  * template[/etc/yum.repos.d/epel.repo] action create
    - create new file /etc/yum.repos.d/epel.repo
    - update content in file /etc/yum.repos.d/epel.repo from none to 18fb55
        --- /etc/yum.repos.d/epel.repo	2013-10-15 15:44:30.975197172 +0000
        +++ /tmp/chef-rendered-template20131015-2525-woqbwo	2013-10-15 15:44:30.976197672 +0000
        @@ -0,0 +1,8 @@
        +# Generated by Chef for localhost
        +# Local modifications will be overwritten.
        +[epel]
        +name=Extra Packages for Enterprise Linux
        +mirrorlist=http://mirrors.fedoraproject.org/mirrorlist?repo=epel-6&arch=$basearch
        +gpgcheck=1
        +gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
        +enabled=1
    - change mode from '' to '0644'

  * execute[yum-makecache-epel] action run
    - execute yum -q makecache --disablerepo=* --enablerepo=epel

  * ruby_block[reload-internal-yum-cache-for-epel] action create
    - execute the ruby block reload-internal-yum-cache-for-epel

Recipe: nginx::default
  * package[nginx] action install
    - install version 1.0.15-5.el6 of package nginx

  * service[nginx] action enable
    - enable service service[nginx]

  * service[nginx] action start
    - start service service[nginx]

  * service[iptables] action enable (up to date)
  * service[iptables] action stop
    - stop service service[iptables]

Chef Client finished, 9 resources updated
```

![nginx](img/nginx.png)

うまくいった！

サードパーティのクックブックをガシガシ使うのはchef-soloに使い慣れてからのほうがいいそうです。

## 代表的なレシピのサンプルを読む: td-agent

いろいろエッセンシャルなものが詰まってるらしい。Fluentd（ログ収集パッケージ）

- [treasure-data/chef-td-agent](https://github.com/treasure-data/chef-td-agent/)

まずはchefリポジトリを作ってそこにレシピをクローンしてくる？

```
bundle exec knife solo init chef-repo
git submodule add -f git@github.com:wnoguchi/chef-td-agent.git chef-repo/cookbooks/chef-td-agent

Cloning into 'chef-repo/cookbooks/chef-td-agent'...
remote: Counting objects: 64, done.
remote: Compressing objects: 100% (40/40), done.
remote: Total 64 (delta 18), reused 46 (delta 7)
Receiving objects: 100% (64/64), 10.96 KiB | 4.00 KiB/s, done.
Resolving deltas: 100% (18/18), done.
Checking connectivity... done

```

### クックブックchef-td-agentのツリー構造。

```
.
├── README.rdoc
├── attributes
│   └── default.rb
├── libraries
│   └── provider_td_rubygems.rb
├── metadata.rb
├── recipes
│   └── default.rb
├── resources
│   └── gem.rb
└── templates
    └── default
        └── td-agent.conf.erb

```

### 構成要素

* Resource:
  * Group
  * User
  * Template
  * apt_repository
  * yum_repository
  * Package
* Attribute: node['foo']

### ohai

システム上のいろんな値を取得するライブラリ。  
Cehfをインストールすると一緒に入る。

* Mac

```
noguchiwataru-no-MacBook-Air:chef-td-agent noguchiwataru$ bundle exec ohai | head
{
  "languages": {
    "ruby": {
      "platform": "x86_64-darwin12.4.1",
      "version": "1.9.3",
      "release_date": "2013-06-27",
      "target": "x86_64-apple-darwin12.4.1",
      "target_cpu": "x86_64",
      "target_vendor": "apple",
      "target_os": "darwin12.4.1",
```

* CentOS on Vagrant VM

```
[vagrant@localhost ~]$ ohai | head
{
  "languages": {
    "ruby": {
      "platform": "x86_64-linux",
      "version": "1.8.7",
      "release_date": "2011-06-30",
      "target": "x86_64-redhat-linux-gnu",
      "target_cpu": "x86_64",
      "target_vendor": "redhat",
      "target_os": "linux",
```

* キー名を指定して値を調べられる

以下はプラットフォームを取得する例。

```
noguchiwataru-no-MacBook-Air:chef-repo-example1 noguchiwataru$ bundle exec ohai platform
[
  "mac_os_x"
]
```

## Resourceについて

### Notification

リソースタイプ、リソース名に対してアクションを指定する。

```ruby
template 'httpd.conf' do
  path "/etc/httpd/conf/httpd.conf"
  source "httpd.conf.erb"
  owner "root"
  group "root"
  mode 0644
  # (snip)
  notifies :restart, 'service[httpd]'
end
```

* And Subscribes in Resource Fire action!!

### Template Resource

ohaiで取得してきた値はシンボル参照、jsonに記述した値は文字列参照を使う。

#### templateを生成してみる

* `node/yunocchi.json`

```
{
  "node_attrs": {
    "key1": "wooo!!"
  },
  "run_list":["hello"]
}
```

* `recipes/default.rb`

```
# generate sample template result with Attribute values

template "/tmp/template_test.txt" do
  source "template_test.txt.erb"
  mode 0644
end
```

* `templates/default/template_test.txt.rb`

```
Attribute read template test:

* Platform: <%= node[:platform] %>
* Ruby: <%= node[:languages][:ruby][:version] %>
* IP Address: <%= node[:ipaddress] %>

And node Attribute value

* key1: <%= node['node_attrs']['key1'] %>
```

* 実行してみる

```
bundle exec knife solo prepare yunocchi
bundle exec knife solo cook yunocchi
```

できたかな？

```
[vagrant@vagrant1-berkshelf ~]$ cat /tmp/template_test.txt 
Attribute read template test:

* Platform: centos
* Ruby: 1.8.7
* IP Address: 10.0.2.15

And node Attribute value

* key1: wooo!!
```

期待どおり。

------------------------------------------------------------------------

時間が空いてしまった・・・。 15章 から。

```
bash "install perlbrew" do

  user 'vagrant'
  group 'vagrant'
  
  cwd '/home/vagrant'
  environment "HOME" => '/home/vagrant'
  code <<-EOC
    curl -kL http://install.perlbrew.pl | bash
  EOC
  creates "/home/vagrant/perl5/perlbrew/bin/perlbrew"

end

```

なんか以下の警告が表示される。んーなにこれ。。。今までこんなの出てこなかったんだけど。。。なんかそれっぽく動いているし。

```
WARNING: Local cookbook_path '/Users/noguchiwataru/Documents/repositories/github/chef-vagrant-working/vagrant1/chef-repo/cookbooks' does not exist
[2013-10-30T13:28:08+00:00] FATAL: No cookbook found in ["/home/vagrant/chef-solo/cookbooks-1", "/home/vagrant/chef-solo/cookbooks-2", "/home/vagrant/chef-solo/cookbooks-3"], make sure cookbook_path is set correctly.
```

```
Running Chef...
Starting Chef Client, version 11.6.2
Compiling Cookbooks...
```

マイナーバージョン上がってる。

- [Mac - Chefに入門 (1) - Qiita キータ](http://qiita.com/us10096698/items/d75a16e82cc1b511b883)

とりあえず意味がわからないことはよくわかった。

* EC2のマイクロインスタンススワップ作成

```
be knife cookbook create ec2-mkswap -o site-cookbooks/
be knife solo prepare pg1x
(nodeファイルいじくる)
ks cook pg1x

```

・・・実行されない。

- [Ruby - Chef で Amazon VPC 内に配置したインスタンスの node["ec2"] が nil になってしまう場合 - Qiita [キータ]](http://qiita.com/labocho/items/2f08cc3d249303122917)
- [Ohai 6.14.0 released | Opscode Blog](http://www.opscode.com/blog/2012/05/30/ohai-6-14-0-released/)

ec2上で

```
sudo -i
mkdir -p /etc/chef/ohai/hints
echo {} > /etc/chef/ohai/hints/ec2.json
```

```
ohai ec2
```

表示されるようになった。

```
log "this is ec2 swap creation recipe."
log "Instance Type is: #{node[:ec2][:instance_type]}"

bash 'create swapfile' do

  code <<-EOC
    dd if=/dev/zero of=/swap.img bs=1M count=2048 && chmod 600 /swap.img
    mkswap /swap.img
  EOC
  only_if { not node[:ec2].nil? and node[:ec2][:instance_type] == 't1.micro' }
  creates "/swap.img"

end

# swapファイルの定義をfstabに書くやつ
# 何回走らせても1エントリだけ？
mount '/dev/null' do
  action :enable
  device '/swap.img'
  fstype 'swap'
  only_if { not node[:ec2].nil? and node[:ec2][:instance_type] == 't1.micro' }
end

bash 'activate swap' do
  code 'swapon -ae'
  # スワップ領域がひとつもなければスワップを有効にする
  only_if "test `cat /proc/swaps | wc -l` -eq 1"
end

```

```
/swap.img /dev/null swap defaults 0 2



Filename				Type		Size	Used	Priority
/swap.img                               file		2097148	0	-1
```

何回走らせてもswapのエントリが増えないのですばらしい。

## Berkshelf

* Gemfile

```
source "https://rubygems.org"

gem 'knife-solo', '0.3.0'
gem 'chef'
gem 'berkshelf'
```

* Berksfile

chefレポジトリのルートに置く？

```
site :opscode
cookbook 'yum'
cookbook 'nginx'
```

とりあえず `cookbooks/` 以下のサードパーティ製クックブックを全削除して

```
bundle exec knife solo cook vagrant1

Running Chef on vagrant1...
Checking Chef version...
Installing Berkshelf cookbooks to 'cookbooks'...
Installing yum (2.4.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing nginx (2.0.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing apt (2.3.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing bluepill (2.3.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing rsyslog (1.9.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing build-essential (1.4.2) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing ohai (1.1.12) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Installing runit (1.3.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
Uploading the kitchen...
Generating solo config...
Running Chef...
```

が失敗するかやってみたらこのタイミングで不足しているクックブックは勝手に取得してくれちゃう感じでした。

で、明示的なbundle installに相当するコマンドが

```
bundle exec berks --path cookbooks

Using yum (2.4.0)
Using nginx (2.0.0)
Using apt (2.3.0)
Using bluepill (2.3.0)
Using rsyslog (1.9.0)
Using build-essential (1.4.2)
Using ohai (1.1.12)
Using runit (1.3.0)
```

になるんだけど、`cookbooks/` 以下を改めて削除しても改めて外から取ってくる気配はない。  
予めどっかにキャッシュしてるのかな？

### Berkshef + Vagrant 連携

```
bundle exec berks cookbook vagrant2

      create  vagrant2/files/default
      create  vagrant2/templates/default
      create  vagrant2/attributes
      create  vagrant2/definitions
      create  vagrant2/libraries
      create  vagrant2/providers
      create  vagrant2/recipes
      create  vagrant2/resources
      create  vagrant2/recipes/default.rb
      create  vagrant2/metadata.rb
      create  vagrant2/LICENSE
      create  vagrant2/README.md
      create  vagrant2/Berksfile
      create  vagrant2/Thorfile
      create  vagrant2/chefignore
      create  vagrant2/.gitignore
         run  git init from "./vagrant2"
      create  vagrant2/Gemfile
      create  vagrant2/Vagrantfile

cd vagrant2

.
├── Berksfile
├── Gemfile
├── LICENSE
├── README.md
├── Thorfile
├── Vagrantfile
├── attributes
├── chefignore
├── definitions
├── files
│   └── default
├── libraries
├── metadata.rb
├── providers
├── recipes
│   └── default.rb
├── resources
└── templates
    └── default

```

以上を見るとVagrantfileとBerkshelf、クックブックの内容がごちゃまぜになってる。  
混乱している。

```
bundle
```

でなんかインストール。このままだとグローバルなgemとしてインストールされる気がする。。。

Attributeやrun_listの記述はVagrantfileに統合するようです。

- [今っぽい Vagrant + Chef Solo チュートリアル - Qiita [キータ]](http://qiita.com/taiki45/items/b46a2f32248720ec2bae)

```ruby
site :opscode

metadata
cookbook 'yum'
cookbook 'nginx'
```

```ruby
  config.vm.provision :chef_solo do |chef|
    chef.json = {
      :mysql => {
        :server_root_password => 'rootpass',
        :server_debian_password => 'debpass',
        :server_repl_password => 'replpass'
      }
    }

    chef.run_list = [
        "recipe[sandbox::default]",
        "recipe[yum::epel]",
        "recipe[nginx]",
    ]
  end
```

```
bundle exec vagrant up

(snip)

There are errors in the configuration of this machine. Please fix
the following errors and try again:

SSH:
* The following settings don't exist: max_tries, timeout

```

```
vagrant --version
Vagrant 1.3.1
```

- [Getting Started Writing Chef Cookbooks the Berkshelf Way, Part 1 - Mischa Taylor's Coding Blog](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/)

によると

```
  config.ssh.max_tries = 40
  config.ssh.timeout   = 120

#↓

config.vm.boot_timeout = 120
```

としないといけないみたい

そして稼働中のVMに対して再度レシピ転送するときは以下のようにする。

```
bundle exec vagrant provision
```

おお、しっかりnginx立ち上がっとる。すごいな。

## VagrantのマルチVM機能を使ってみる

Chef Serverは敷居高めなので今回は見送り。  
`vagrant init` して `Vagrantfile` に以下の記述を追加する。  
ちなみにバージョンは Vagrant 1.3.1 です。  
バージョン上がるとホストオンリーネットワークの記述とか微妙に違っているっぽいので注意。

以下は5つのVMを立ち上げる例。

```ruby
  config.vm.define :vm1 do |cfg|
    cfg.vm.box = "base"
    cfg.vm.network :private_network, ip: "192.168.30.10"
    cfg.vm.host_name = "vm1"
  end

  config.vm.define :vm2 do |cfg|
    cfg.vm.box = "base"
    cfg.vm.network :private_network, ip: "192.168.30.11"
    cfg.vm.host_name = "vm2"
  end

  config.vm.define :vm3 do |cfg|
    cfg.vm.box = "base"
    cfg.vm.network :private_network, ip: "192.168.30.12"
    cfg.vm.host_name = "vm3"
  end

  config.vm.define :vm4 do |cfg|
    cfg.vm.box = "base"
    cfg.vm.network :private_network, ip: "192.168.30.13"
    cfg.vm.host_name = "vm4"
  end

  config.vm.define :vm5 do |cfg|
    cfg.vm.box = "base"
    cfg.vm.network :private_network, ip: "192.168.30.14"
    cfg.vm.host_name = "vm5"
  end

```

そして

```
vagrant up
```

```
Macintosh:vagrant2 noguchiwataru$ vagrant up
Bringing machine 'vm1' up with 'virtualbox' provider...
Bringing machine 'vm2' up with 'virtualbox' provider...
Bringing machine 'vm3' up with 'virtualbox' provider...
Bringing machine 'vm4' up with 'virtualbox' provider...
Bringing machine 'vm5' up with 'virtualbox' provider...
[vm1] Importing base box 'base'...
[vm1] Matching MAC address for NAT networking...
[vm1] Setting the name of the VM...
[vm1] Clearing any previously set forwarded ports...
[vm1] Creating shared folders metadata...
[vm1] Clearing any previously set network interfaces...
[vm1] Preparing network interfaces based on configuration...
[vm1] Forwarding ports...
[vm1] -- 22 => 2222 (adapter 1)
[vm1] Booting VM...
[vm1] Waiting for machine to boot. This may take a few minutes...
[vm1] Machine booted and ready!
[vm1] Setting hostname...
[vm1] Configuring and enabling network interfaces...
[vm1] Mounting shared folders...
[vm1] -- /vagrant
[vm2] Importing base box 'base'...
[vm2] Matching MAC address for NAT networking...

(snip)


[vm5] Setting hostname...
[vm5] Configuring and enabling network interfaces...
[vm5] Mounting shared folders...
[vm5] -- /vagrant

```

すごいですねー。

SSHはホスト名を指定して

```
vagrant ssh vm1
```

VMのステータス一覧を取得するには

```
vagrant status

Current machine states:

vm1                       running (virtualbox)
vm2                       running (virtualbox)
vm3                       running (virtualbox)
vm4                       running (virtualbox)
vm5                       running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

全部の仮想マシンを停止するには

```
vagrant halt

[vm1] Attempting graceful shutdown of VM...
[vm2] Attempting graceful shutdown of VM...
[vm3] Attempting graceful shutdown of VM...
[vm4] Attempting graceful shutdown of VM...
[vm5] Attempting graceful shutdown of VM...
```

## 参考サイト

* [Chef Soloの正しい始め方 | tsuchikazu blog](http://tsuchikazu.net/chef_solo_start/)
