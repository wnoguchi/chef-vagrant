# Note1

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

## References

- [今っぽい Vagrant + Chef Solo チュートリアル - Qiita [キータ]](http://qiita.com/taiki45/items/b46a2f32248720ec2bae)
