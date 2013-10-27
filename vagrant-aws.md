# VagrantでAmazon EC2のインスタンスを操作する

VirtualBoxだけではなく、Amazon EC2をプロバイダとして選択できるようになったそうです。  
たまらない。  
こういうの見るとやりたくてしょうがないわけです。

## インストール

* AWSプラグインをインストール

```
vagrant plugin install vagrant-aws
```

* ダミーのboxをadd

```
vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box

$ vagrant box list
base  (virtualbox)
dummy (aws)
```

ダミーのboxが追加された。

## 参考サイト

- [Vagrant 1.1 で EC2 を vagrant up - naoyaのはてなダイアリー](http://d.hatena.ne.jp/naoya/20130315/1363340698)
