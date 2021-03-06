# chef-solo + Vagrant on Linux

LinuxをホストOSとしてchef-solo。  
ここではchef-solo。  
CentOSはイバラの道らしいのでUbuntu13.04で。

## Tips

chef-soloするときのGemfile。

```
# Gemfile
source 'https://rubygems.org'
gem 'chef'
gem 'sahara'
gem 'knife-solo'
```

## Getting Started

### インストール

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

または

```
# Gemfile
source 'https://rubygems.org'
gem 'chef'
```

して

```
bundle install --path vendor/bundle
```

### レポジトリの作成

```
git clone git://github.com/opscode/chef-repo.git
```

```
knife configure
```

### クックブック作成

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
Compiling Cookbooks...
Converging 1 resources
Recipe: hello::default
  * log[Hello, Chef!] action write

Chef Client finished, 1 resources updated

```

zshを入れてみる

```ruby
package "zsh" do
  action :install
end
```

さらにRubyの文法を活用してみる。  
ちなみにAmazon LinuxではなくてUbuntuでやってるのでパッケージ名はUbuntuのパッケージングリポジトリに載ってる名前でやらないとエラーになるみたい。

```ruby
%w{zsh gcc make libreadline-dev vim}.each do |pkg|
  package pkg do
    action :install
  end
end
```

出力は以下のとおり。

```
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 6 resources
Recipe: hello::default
  * log[Hello, Chef!] action write

  * package[zsh] action install
    - install version 5.0.0-2ubuntu3 of package zsh

  * package[gcc] action install (up to date)
  * package[make] action install (up to date)
  * package[libreadline-dev] action install
    - install version 6.2-9ubuntu1 of package libreadline-dev

  * package[vim] action install
    - install version 2:7.3.547-6ubuntu5 of package vim

Chef Client finished, 4 resources updated
```

## nginx

```
knife cookbook create nginx -o cookbooks 
** Creating cookbook nginx
** Creating README for cookbook: nginx
** Creating CHANGELOG for cookbook: nginx
** Creating metadata for cookbook: nginx

```

```ruby
# cookbooks/nginx/recipes/default.rb

package "nginx" do
  action :install
end

service "nginx" do
  supports :status => true, :restart => true, :reload => true
  action [ :enable, :start ]
end

template "nginx.conf" do
  path "/etc/nginx/nginx.conf"
  source "nginx.conf.erb"
  owner "root"
  owner "root"
  mode 0644
  notifies :reload, 'service[nginx]'
end
```

```
# cookbooks/nginx/templates/default/nginx.conf.erb

user nginx;
worker_processes 1;
error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  server {
    listen <%= node['nginx']['port'] %>;
    server_name localhost;
    location / {
      root /usr/share/nginx/html;
      index index.html index.htm;
    }
  }
}
```

```
// localhost.json
{
  "nginx": {
    "port": 80
  },
  "run_list": [
    "nginx"
  ]
}
```

```
tree -F
.
├── CHANGELOG.md
├── README.md
├── attributes/
├── definitions/
├── files/
│   └── default/
├── libraries/
├── metadata.rb
├── providers/
├── recipes/
│   └── default.rb
├── resources/
└── templates/
    └── default/
        └── nginx.conf.erb

10 directories, 5 files
```

```
sudo chef-solo -c solo.rb -j ./localhost.json


Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 3 resources
Recipe: nginx::default
  * package[nginx] action install
    - install version 1.2.6-1ubuntu3.2 of package nginx

  * service[nginx] action enable (up to date)
  * service[nginx] action start
    - start service service[nginx]

  * template[nginx.conf] action create
    - update content in file /etc/nginx/nginx.conf from 9492ca to 267bcf
        --- /etc/nginx/nginx.conf	2012-12-17 15:57:45.000000000 +0900
        +++ /tmp/chef-rendered-template20130905-9585-1mv06ac	2013-09-05 08:57:28.333738529 +0900
        @@ -1,95 +1,24 @@
        -user www-data;
        -worker_processes 4;
        -pid /run/nginx.pid;
        +
        +user nginx;
        +worker_processes 1;
        +error_log /var/log/nginx/error.log;
        +pid /var/run/nginx.pid
         
         events {
        -	worker_connections 768;
        -	# multi_accept on;
        +  worker_connections 1024;
         }
         
         http {
        +  include /etc/nginx/mime.types;
        +  default_type application/octet-stream;
         
        -	##
        -	# Basic Settings
        -	##
        -
        -	sendfile on;
        -	tcp_nopush on;
        -	tcp_nodelay on;
        -	keepalive_timeout 65;
        -	types_hash_max_size 2048;
        -	# server_tokens off;
        -
        -	# server_names_hash_bucket_size 64;
        -	# server_name_in_redirect off;
        -
        -	include /etc/nginx/mime.types;
        -	default_type application/octet-stream;
        -
        -	##
        -	# Logging Settings
        -	##
        -
        -	access_log /var/log/nginx/access.log;
        -	error_log /var/log/nginx/error.log;
        -
        -	##
        -	# Gzip Settings
        -	##
        -
        -	gzip on;
        -	gzip_disable "msie6";
        -
        -	# gzip_vary on;
        -	# gzip_proxied any;
        -	# gzip_comp_level 6;
        -	# gzip_buffers 16 8k;
        -	# gzip_http_version 1.1;
        -	# gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
        -
        -	##
        -	# nginx-naxsi config
        -	##
        -	# Uncomment it if you installed nginx-naxsi
        -	##
        -
        -	#include /etc/nginx/naxsi_core.rules;
        -
        -	##
        -	# nginx-passenger config
        -	##
        -	# Uncomment it if you installed nginx-passenger
        -	##
        -	
        -	#passenger_root /usr;
        -	#passenger_ruby /usr/bin/ruby;
        -
        -	##
        -	# Virtual Host Configs
        -	##
        -
        -	include /etc/nginx/conf.d/*.conf;
        -	include /etc/nginx/sites-enabled/*;
        +  server {
        +    listen 80;
        +    server_name localhost;
        +    location / {
        +      root /usr/share/nginx/html;
        +      index index.html index.htm;
        +    }
        +  }
         }
         
        -
        -#mail {
        -#	# See sample authentication script at:
        -#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
        -# 
        -#	# auth_http localhost/auth.php;
        -#	# pop3_capabilities "TOP" "USER";
        -#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
        -# 
        -#	server {
        -#		listen     localhost:110;
        -#		protocol   pop3;
        -#		proxy      on;
        -#	}
        -# 
        -#	server {
        -#		listen     localhost:143;
        -#		protocol   imap;
        -#		proxy      on;
        -#	}
        -#}

  * service[nginx] action reload
================================================================================
Error executing action `reload` on resource 'service[nginx]'
================================================================================


Mixlib::ShellOut::ShellCommandFailed
------------------------------------
Expected process to exit with [0], but received '1'
---- Begin output of /etc/init.d/nginx reload ----
STDOUT: 
STDERR: 
---- End output of /etc/init.d/nginx reload ----
Ran /etc/init.d/nginx reload returned 1


Resource Declaration:
---------------------
# In /home/wnoguchi/chef-repo/cookbooks/nginx/recipes/default.rb

 14: service "nginx" do
 15:   supports :status => true, :restart => true, :reload => true
 16:   action [ :enable, :start ]
 17: end
 18: 



Compiled Resource:
------------------
# Declared in /home/wnoguchi/chef-repo/cookbooks/nginx/recipes/default.rb:14:in `from_file'

service("nginx") do
  action [:enable, :start]
  updated true
  supports {:status=>true, :restart=>true, :reload=>true}
  retries 0
  retry_delay 2
  service_name "nginx"
  enabled true
  running true
  pattern "nginx"
  startup_type :automatic
  cookbook_name :nginx
  recipe_name "default"
end



[2013-09-05T08:57:28+09:00] ERROR: Running exception handlers
[2013-09-05T08:57:28+09:00] ERROR: Exception handlers complete
[2013-09-05T08:57:28+09:00] FATAL: Stacktrace dumped to /tmp/chef-solo/chef-stacktrace.out
Chef Client failed. 3 resources updated
[2013-09-05T08:57:28+09:00] FATAL: Chef::Exceptions::ChildConvergeError: Chef run process exited unsuccessfully (exit code 1)

```

エラー。。。  
Ubuntu仕様に書き直してみる。

```
# cookbooks/nginx/templates/default/nginx.conf.erb

user www-data;
worker_processes 4;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  server {
    listen <%= node['nginx']['port'] %>;
    server_name localhost;
    location / {
      root /usr/share/nginx/html;
      index index.html index.htm;
    }
  }
}

```

もう一度。

```
sudo chef-solo -c solo.rb -j ./localhost.json      
[sudo] password for unicast: 
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 3 resources
Recipe: nginx::default
  * package[nginx] action install (up to date)
  * service[nginx] action enable (up to date)
  * service[nginx] action start (up to date)
  * template[nginx.conf] action create
    - update content in file /etc/nginx/nginx.conf from 267bcf to c7207c
        --- /etc/nginx/nginx.conf	2013-09-05 08:57:28.333738529 +0900
        +++ /tmp/chef-rendered-template20130905-10526-3gql20	2013-09-05 12:21:47.854516784 +0900
        @@ -1,8 +1,8 @@
         
        -user nginx;
        -worker_processes 1;
        +user www-data;
        +worker_processes 4;
         error_log /var/log/nginx/error.log;
        -pid /var/run/nginx.pid
        +pid /run/nginx.pid;
         
         events {
           worker_connections 1024;

  * service[nginx] action reload
    - reload service service[nginx]

Chef Client finished, 2 resources updated

```

今度はうまく行った。  
ポートを変えてみる。

```
sudo chef-solo -c solo.rb -j ./localhost.json
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 3 resources
Recipe: nginx::default
  * package[nginx] action install (up to date)
  * service[nginx] action enable (up to date)
  * service[nginx] action start (up to date)
  * template[nginx.conf] action create
    - update content in file /etc/nginx/nginx.conf from c7207c to 14074a
        --- /etc/nginx/nginx.conf	2013-09-05 12:21:47.854516784 +0900
        +++ /tmp/chef-rendered-template20130905-10918-1r1m3qi	2013-09-05 12:23:09.728528973 +0900
        @@ -13,7 +13,7 @@
           default_type application/octet-stream;
         
           server {
        -    listen 80;
        +    listen 8080;
             server_name localhost;
             location / {
               root /usr/share/nginx/html;

  * service[nginx] action reload
    - reload service service[nginx]

Chef Client finished, 2 resources updated
```

よし。

iptablesをoffにする。

```
service 'iptables' do
  action [ :disable, :stop ]
end
```
