# Chef and Vagrant Study Repository

chef-solo + Vagrant勉強用リポジトリー。  
ほぼ以下の電子書籍を参考書に勉強しています。  
伊藤直也さんの本で必ず買ったほうがいいです。

- [Amazon.co.jp： 入門Chef Solo - Infrastructure as Code eBook: 伊藤直也: Kindleストア](http://www.amazon.co.jp/%E5%85%A5%E9%96%80Chef-Solo-Infrastructure-Code-ebook/dp/B00BSPH158)

## Vagrant Host-only network IP address and HostName design policy

### MacBook Air in Home

#### About

vmach(Virtual MAC home)

* ex. `192.168.100.10` `vmach10`

### MacBook Air in Company

#### About

vmacc(Virtual MAC Company)

* ex. `192.168.100.10` `vmacc10`

#### SSH configuration generation example

```
vagrant ssh-config --host vmacc10 | tee -a ~/.ssh/config
```

and connect to VirtualBox host like this

```
ssh vmacc10
```

#### Prepare Chef-solo into VM

```
bundle exec knife solo prepare vmacc10
```

## 最近のはなし

僕の環境はVirtualBox4.2だけど、Mavericksにしてからしょっちゅうエラーになる。なんなの。

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Clearing any previously set forwarded ports...
[Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
[Berkshelf] You should check for a newer version of vagrant-berkshelf.
[Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
[Berkshelf] You can also join the discussion in #berkshelf on Freenode.
[Berkshelf] Updating Vagrant's berkshelf: '/Users/noguchiwataru/.berkshelf/default/vagrant/berkshelf-20131109-10615-o2kbg1-default'
[Berkshelf] Using hello (0.1.0) at './site-cookbooks/hello'
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["hostonlyif", "create"]

Stderr: 0%...
Progress state: NS_ERROR_FAILURE
VBoxManage: error: Failed to create the host-only adapter
VBoxManage: error: VBoxNetAdpCtl: Error while adding new interface: failed to open /dev/vboxnetctl: No such file or directory

VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component HostNetworkInterface, interface IHostNetworkInterface
VBoxManage: error: Context: "int handleCreate(HandlerArg*, int, int*)" at line 68 of file VBoxManageHostonly.cpp
```

- [Mavericks + VirtualBox 4.3で「Failed to create the host-only adapter」のエラーが出る場合の対処法 - F.Ko-Jiの「一秒後は未来」](http://blog.fkoji.com/2013/10260009.html)

によると

```
sudo /Library/StartupItems/VirtualBox/VirtualBox restart
```

## 参考サイト

### マニュアル

- [About Resources and Providers — Chef Docs](http://docs.opscode.com/resource.html)

### 公認

- [Berkshelf](http://berkshelf.com/)

#### Opscode community

膨大なcookbooksが集積されている。  
サインアップして秘密鍵を保存するのです。

- [Opscode Community](http://community.opscode.com/cookbooks)
- [opscode-cookbooks (Opscode Public Cookbooks)](https://github.com/opscode-cookbooks)

### 実践的

#### Rails + Vagrant + Chef Solo

- [ASCIIcasts - “Episode 292 - Vagrantで仮想マシン”](http://ja.asciicasts.com/episodes/292-virtual-machines-with-vagrant)

### ナウいトピック

- [Ruby - 今っぽい Vagrant + Chef Solo チュートリアル - Qiita [キータ]](http://qiita.com/taiki45/items/b46a2f32248720ec2bae)
- [Vagrant 1.1 で EC2 を vagrant up - naoyaのはてなダイアリー](http://d.hatena.ne.jp/naoya/20130315/1363340698)  
Amazon EC2のインスタンスをupする。近いうちに読みたい。
- [PackerでVagrant Boxを作成する際のトラブルシューティング | ユニキャストラボ](http://lab.unicast.ne.jp/2013/09/09/troubleshooting-create-vagrant-box-with-packer/)

### Chef系

- [Amazon.co.jp： 入門Chef Solo - Infrastructure as Code eBook: 伊藤直也: Kindleストア](http://www.amazon.co.jp/%E5%85%A5%E9%96%80Chef-Solo-Infrastructure-Code-ebook/dp/B00BSPH158)  
主にこれを参考にさせてもらっています。

#### misc

- [特集　DevOps時代の必須知識：インフラストラクチャ自動化フレームワーク「Chef」の基本 (1/2) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1305/24/news003.html)
- [Rubyist Magazine - Chef でサーバ管理を楽チンにしよう！ (第 1 回)](http://magazine.rubyist.net/?0035-ChefInDECOLOG)
- [Chef Soloの正しい始め方 | tsuchikazu blog](http://tsuchikazu.net/chef_solo_start/)
- [Chef Soloと Knife Soloでの　ニコニコサーバー構築 (1):dwango エンジニア ブロマガ:ドワンゴ研究開発チャンネル(ドワンゴグループのエンジニア) - ニコニコチャンネル:生活](http://ch.nicovideo.jp/dwango-engineer/blomaga/ar311555)
- [Windows7上で Vagrant + Chef solo + knife-soloを使い、Ubuntu + ubuntu-desktopの環境を構築してみた - メモ的な思考的な](http://d.hatena.ne.jp/thinkAmi/20130407/1365310673)
- [入門Chef Solo - Infrastructure as Code - 達人出版会](http://tatsu-zine.com/books/chef-solo)

### Vagrant系

- [vagrant windows環境でSSH for Teraterm - clash_m45の開発日記](http://d.hatena.ne.jp/clash_m45/20130716/1373975271)
- [Ruby - Vagrant and Chef on Windows - Qiita [キータ]](http://qiita.com/ogomr/items/98a33f47f6ba050adac4)
- [Window 7 でVagrantでCent OS 6.3入れてみた - 僕の車輪の再発明](http://kazuph.hateblo.jp/entry/2013/02/05/234243)
