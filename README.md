# Chef and Vagrant Study Repository

chef-solo + Vagrant勉強用リポジトリー。  
ほぼ以下の電子書籍を参考書に勉強しています。  
伊藤直也さんの本で必ず買ったほうがいいです。

- [Amazon.co.jp： 入門Chef Solo - Infrastructure as Code eBook: 伊藤直也: Kindleストア](http://www.amazon.co.jp/%E5%85%A5%E9%96%80Chef-Solo-Infrastructure-Code-ebook/dp/B00BSPH158)

## Vagrant Host-only network IP address and HostName design policy

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

## 参考サイト

### マニュアル

- [About Resources and Providers — Chef Docs](http://docs.opscode.com/resource.html)

### 公認

#### Vagrant

- [A list of base boxes for Vagrant - Vagrantbox.es](http://www.vagrantbox.es/)  
Vagrantのboxイメージの集積所
- [PackerでVagrant Boxを作成する際のトラブルシューティング | ユニキャストラボ](http://lab.unicast.ne.jp/2013/09/09/troubleshooting-create-vagrant-box-with-packer/)

#### Opscode community

膨大なcookbooksが集積されている。  
サインアップして秘密鍵を保存するのです。

- [Opscode Community](http://community.opscode.com/cookbooks)

### ナウいトピック

- [Ruby - 今っぽい Vagrant + Chef Solo チュートリアル - Qiita [キータ]](http://qiita.com/taiki45/items/b46a2f32248720ec2bae)

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
