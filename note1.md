# Note1

taki45さんのをおおいに参考にさせていただいています。  
感謝。

## Basics

```
vagrant plugin install vagrant-omnibus
vagrant plugin install vagrant-berkshelf
```

```
vagrant init
```

```
cat <<EOF >Berksfile
site :opscode
EOF
```

```
mkdir site-cookbooks
cd site-cookbooks
bundle exec berks cookbook hello
```

```
  config.omnibus.chef_version = :latest
  config.berkshelf.enabled = true
```

* `site-cookbooks/hello/recipes/default.rb` の編集。

```
log "Hello, Vagrant and Chef solo!"
```

```
vagrant up

Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'base'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
[Berkshelf] You should check for a newer version of vagrant-berkshelf.
[Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
[Berkshelf] You can also join the discussion in #berkshelf on Freenode.
[Berkshelf] Updating Vagrant's berkshelf: '/Users/noguchiwataru/.berkshelf/default/vagrant/berkshelf-20131109-10615-o2kbg1-default'
[Berkshelf] Using hello (0.1.0) at './site-cookbooks/hello'
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
[default] Machine booted and ready!
[default] Configuring and enabling network interfaces...
[default] Mounting shared folders...
[default] -- /vagrant
[default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
[default] Installing Chef 11.8.0 Omnibus package...
[default] Downloading Chef 11.8.0 for el...
[default] Installing Chef 11.8.0
[default] warning: 
[default] /tmp/tmp.qpbLURGa/chef-11.8.0.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
[default] Preparing...                
[default] #########################
[default] #########################
[default] 
[default] chef                        
[default] #

(snip)

[default] #
[default] 
[default] Thank you for installing Chef!
[default] Running provisioner: chef_solo...
Generating chef JSON and uploading...
Running chef-solo...
[2013-11-09T10:55:28+00:00] INFO: Forking chef instance to converge...
[2013-11-09T10:55:28+00:00] INFO: *** Chef 11.8.0 ***
[2013-11-09T10:55:28+00:00] INFO: Chef-client pid: 2523
[2013-11-09T10:55:28+00:00] INFO: Setting the run_list to ["hello"] from JSON
[2013-11-09T10:55:28+00:00] INFO: Run List is [recipe[hello]]
[2013-11-09T10:55:28+00:00] INFO: Run List expands to [hello]
[2013-11-09T10:55:28+00:00] INFO: Starting Chef Run for localhost
[2013-11-09T10:55:28+00:00] INFO: Running start handlers
[2013-11-09T10:55:28+00:00] INFO: Start handlers complete.
[2013-11-09T10:55:28+00:00] INFO: Hello, Vagrant and Chef solo!
[2013-11-09T10:55:28+00:00] INFO: Chef Run complete in 0.039802787 seconds
[2013-11-09T10:55:28+00:00] INFO: Running report handlers
[2013-11-09T10:55:28+00:00] INFO: Report handlers complete
[2013-11-09T10:55:28+00:00] INFO: Forking chef instance to converge...
```

## MySQLのクックブックを入れてみる

- [opscode-cookbooks/mysql](https://github.com/opscode-cookbooks/mysql)

さて、

Berksfileは以下

```ruby
site :opscode
cookbook "hello", path: "site-cookbooks/hello"
cookbook "mysql"
```

Vagrantfileのプロビジョニング定義部分は以下

```ruby
  config.vm.provision :chef_solo do |chef|
    chef.run_list = [
      "hello",
      "mysql::client",
      "mysql::server",
    ]

    chef.json = {
      mysql: {
        server_root_password: "testtest",
        server_repl_password: "testtest",
        server_debian_password: "testtest",
        bind_address: "127.0.0.1"
      }
    }
  end
```

すでにVirtualBoxのインスタンスは立ち上がってるので、以下のコマンドでプロビジョニングを実行する。  
ちなみにrun_listの部分を `recipe[mysql::client]` とかやったらエラーになった。
JSONファイルではエラーにならなかった気がしたけど、なぜだろう。

```
bundle exec vagrant provision

[Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
[Berkshelf] You should check for a newer version of vagrant-berkshelf.
[Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
[Berkshelf] You can also join the discussion in #berkshelf on Freenode.
[Berkshelf] Updating Vagrant's berkshelf: '/Users/noguchiwataru/.berkshelf/default/vagrant/berkshelf-20131109-10615-o2kbg1-default'
[Berkshelf] Using hello (0.1.0) at './site-cookbooks/hello'
[Berkshelf] Installing mysql (4.0.4) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
[Berkshelf] Installing openssl (1.1.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
[Berkshelf] Using build-essential (1.4.2)
[default] Chef 11.8.0 Omnibus package is already installed.
[default] Running provisioner: chef_solo...
Generating chef JSON and uploading...
Running chef-solo...
[2013-11-09T11:14:41+00:00] INFO: Forking chef instance to converge...
[2013-11-09T11:14:41+00:00] INFO: *** Chef 11.8.0 ***
[2013-11-09T11:14:41+00:00] INFO: Chef-client pid: 2859
[2013-11-09T11:14:41+00:00] INFO: Setting the run_list to ["hello", "mysql::client", "mysql::server"] from JSON
[2013-11-09T11:14:41+00:00] INFO: Run List is [recipe[hello], recipe[mysql::client], recipe[mysql::server]]
[2013-11-09T11:14:41+00:00] INFO: Run List expands to [hello, mysql::client, mysql::server]
[2013-11-09T11:14:41+00:00] INFO: Starting Chef Run for localhost
[2013-11-09T11:14:41+00:00] INFO: Running start handlers
[2013-11-09T11:14:41+00:00] INFO: Start handlers complete.
[2013-11-09T11:14:41+00:00] WARN: Cloning resource attributes for directory[/var/lib/mysql] from prior resource (CHEF-3694)
[2013-11-09T11:14:41+00:00] WARN: Previous directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/_server_rhel.rb:11:in `block in from_file'
[2013-11-09T11:14:41+00:00] WARN: Current  directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/_server_rhel.rb:20:in `from_file'
[2013-11-09T11:14:41+00:00] INFO: Hello, Vagrant and Chef solo!
[2013-11-09T11:14:58+00:00] INFO: package[mysql] installing mysql-5.1.69-1.el6_4 from updates repository
[2013-11-09T11:15:07+00:00] INFO: package[mysql-devel] installing mysql-devel-5.1.69-1.el6_4 from updates repository
[2013-11-09T11:15:13+00:00] INFO: package[mysql-server] installing mysql-server-5.1.69-1.el6_4 from updates repository
[2013-11-09T11:15:22+00:00] INFO: directory[/var/run/mysqld] mode changed to 775
[2013-11-09T11:15:22+00:00] INFO: directory[/var/lib/mysql] mode changed to 775
[2013-11-09T11:15:22+00:00] INFO: directory[/var/log/mysql] created directory /var/log/mysql
[2013-11-09T11:15:22+00:00] INFO: directory[/var/log/mysql] owner changed to 27
[2013-11-09T11:15:22+00:00] INFO: directory[/var/log/mysql] group changed to 27
[2013-11-09T11:15:22+00:00] INFO: directory[/var/log/mysql] mode changed to 775
[2013-11-09T11:15:22+00:00] INFO: directory[/etc/mysql/conf.d] created directory /etc/mysql/conf.d
[2013-11-09T11:15:22+00:00] INFO: directory[/etc/mysql/conf.d] owner changed to 27
[2013-11-09T11:15:22+00:00] INFO: directory[/etc/mysql/conf.d] group changed to 27
[2013-11-09T11:15:22+00:00] INFO: directory[/etc/mysql/conf.d] mode changed to 775
[2013-11-09T11:15:22+00:00] INFO: template[initial-my.cnf] backed up to /var/chef/backup/etc/my.cnf.chef-20131109111522.902091
[2013-11-09T11:15:22+00:00] INFO: template[initial-my.cnf] updated file contents /etc/my.cnf
[2013-11-09T11:15:22+00:00] INFO: template[initial-my.cnf] sending start action to service[mysql-start] (immediate)
[2013-11-09T11:15:24+00:00] INFO: service[mysql-start] started
[2013-11-09T11:15:24+00:00] INFO: execute[assign-root-password] ran successfully
[2013-11-09T11:15:24+00:00] INFO: template[/etc/mysql_grants.sql] created file /etc/mysql_grants.sql
[2013-11-09T11:15:24+00:00] INFO: template[/etc/mysql_grants.sql] updated file contents /etc/mysql_grants.sql
[2013-11-09T11:15:24+00:00] INFO: template[/etc/mysql_grants.sql] owner changed to 0
[2013-11-09T11:15:24+00:00] INFO: template[/etc/mysql_grants.sql] group changed to 0
[2013-11-09T11:15:24+00:00] INFO: template[/etc/mysql_grants.sql] mode changed to 600
[2013-11-09T11:15:24+00:00] INFO: template[/etc/mysql_grants.sql] sending run action to execute[install-grants] (immediate)
[2013-11-09T11:15:24+00:00] INFO: execute[install-grants] ran successfully
[2013-11-09T11:15:25+00:00] INFO: service[mysql] enabled
[2013-11-09T11:15:25+00:00] INFO: Chef Run complete in 43.800555894 seconds
[2013-11-09T11:15:25+00:00] INFO: Running report handlers
[2013-11-09T11:15:25+00:00] INFO: Report handlers complete
[2013-11-09T11:14:41+00:00] INFO: Forking chef instance to converge...
```

実際にインストールできたかつないでみる

```
Macintosh:vagrant3 noguchiwataru$ vagrant ssh
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$ mysql -u root -ptesttest
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.1.69-log Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

おーすげー。

## References

- [今っぽい Vagrant + Chef Solo チュートリアル - Qiita [キータ]](http://qiita.com/taiki45/items/b46a2f32248720ec2bae)
