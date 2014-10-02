# Demo sakurraform and knife-zero with dozens, fluentd

## Setup

### Sakurraform

```
$ bundle install --path vendor/bundle --binstubs


$ ./bin/sakurraform 
Commands:
  sakurraform bs SUBCOMMAND    # Manage Sakura no Base Storage
  sakurraform help [COMMAND]   # Describe available commands or one specific command
  sakurraform init             # initiaize .sakuracloud/credentials
  sakurraform map              # open sakura cloud map!
  sakurraform plan SUBCOMMAND  # Manage plan
  sakurraform status           # show status [--sync](to update cached_state)
  sakurraform version          # show version
```


### Knife-Zero

```
$ cd chef_repo
$ bundle install --path vendor/bundle --binstubs


$ ./bin/knife zero
FATAL: Cannot find sub command for: 'zero'
Available zero subcommands: (for details, knife SUB-COMMAND --help)

** ZERO COMMANDS **
knife zero bootstrap FQDN (options)
knife zero chef_client QUERY (options)
```

#### Rake tasks(chef_repo)

```
$ cd chef_repo

$ ./bin/rake -vT
rake add_hostname    # add hosts to dozens
rake bootstrap       # bootstrap servers with knife-zero
rake data_bag        # convert credentials to data_bag
rake default         # show server_states for debug
rake flush_hostname  # clear all hosts on dozens (Caution! All record of domain will be deleted.)
```


## Steps.

このデモのプランはこんな構成になります。

- network1
    - server1 (fluent-fowarder)
    - server2 (fluent-aggregater)
- network2
    - server3 (fluent-fowarder)
    - server4 (fluent-aggregater)

※ さくらのクラウド APIキー、BASE STORAGEのAPIキーが必要です。

### プロジェクトルートで。インフラを作ります。


1. `./bin/sakurraform init`で`credencials`を作る
1. `./env.sh.sample` を元に`env.sh`を作成、ssh鍵のIDを登録し、sourceで読んでおく。
1. `./bin/sakurraform plan apply` でインフラ作成
1. `./bin/sakurraform status --sync` でリモート状態をローカルキャッシュと同期
    - 割り当てたIPアドレスなど

> `plan apply`ではAPIが結構落ちます(503)が、もう一度実行すれば続きからリソースを作成します。


### `chef_repo`ディレクトリで。サーバの構成をします。

事前にSSH-Agentにログイン用の秘密鍵を登録しておきましょう。

※ Dozens連携を試したければホストの登録が何もないドメインを用意。


1. (※option)`./env.sh.sample` を元に`env.sh`を作成、DOZENS_ID、sourceで読んでおく。
1. `rake data_bag` でBase Storageの情報入りData_Bagを作成
1. `rake bootstrap`でfluent準備OK
1. (※option)`rake add_hostname`でDozensにAレコードを作成します


### fluent=> Base Storageを確認

1. `chef_repo`ディレクトリでデータ投入
    1. `echo '{"hoge" : "piyo"}' | ./bin/fluent-cat -h {IPまたはホスト名} stdin.test` でfluent-fowarderにデータ送信。
1. 60秒くらい待機
1. プロジェクトルートでデータを表示
    1. `./bin/sakurraform bs ls`でオブジェクト一覧を確認。
    1. `./bin/sakurraform bs cat {キー名}`で中身を確認。
    1. `./bin/sakurraform bs delete {キー名}`でオブジェクトを削除。
