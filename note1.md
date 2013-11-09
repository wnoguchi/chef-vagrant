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
```

## References

- [今っぽい Vagrant + Chef Solo チュートリアル - Qiita [キータ]](http://qiita.com/taiki45/items/b46a2f32248720ec2bae)
