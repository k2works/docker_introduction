Docker入門
===
# 目的
Dockerの習得を目的とする。

# 前提
| ソフトウェア     | バージョン    | 備考         |
|:---------------|:-------------|:------------|
| OS X           |10.8.5        |             |
| boot2docker　　 |1.0.0        |             |
| vagrant   　　 |1.6.0        |             |

+ [Vagrant入門](https://github.com/k2works/vagrant_introduction)

# 構成
+ [Dockerとは](#1)
+ [インストール](#2)
+ [Docker Hubはじめに](#3)
+ [ドカライズアプリケーション](#4)
+ [コンテナでの作業](#5)
+ [Tips](#9)

# 詳細
## <a name="1">Dockerとは</a>
>Develop, Ship and Run Any Application, Anywhere  
>開発・配置そしてどこでもどんなアプリケーションでも実行する

Dockerは開発者と開発のためのシステム管理者がアプリケーションを配置そして実行するためのプラットフォームです。

なぜDockerか？

+ アプリケーションをよりはやく届ける
+ デプロイとスケールをもっと簡単に
+ 迅速な開発は管理をもっと楽にする

Dockerの構成

+ Dockerエンジン・・・アプリケーションの構築・コンテナのワークフローを統合する軽量かつパワフルなオープンソース仮想コンテナ技術
+ [Docker Hub](https://hub.docker.com/)・・・アプリケーションスタックを共有・管理するためのSaasサービス

インスールガイド
+ [こっち](http://docs.docker.com/installation/#installation)

ユーザーガイド
+ [こっち](http://docs.docker.com/userguide/)

## <a name="2">インストール</a>
### [MacOS X](http://docs.docker.com/installation/mac/)の場合

#### Dockerセットアップ  
[Docker for OSX Installer](https://github.com/boot2docker/osx-installer/releases)をダウンロードした後以下のコマンド実行。
```bash
$ boot2docker init
$ boot2docker start
$ export DOCKER_HOST=tcp://$(boot2docker ip 2>/dev/null):2375
```
`bootdocker init`コマンドを実行するとSSHキーパスフレーズを聞いてくる。  
これは`boot2docker ssh`コマンド実行の際に使われる。  
仮想マシンのセットアップが完了したら`boot2docker stop`と`bot2docker start`コマンドでコントロールする。

#### Dockerアップグレード  
既存仮想マシン環境をアップグレードするには以下のコマンドを実行する。
```bash
$ boot2docker stop
$ boot2docker download
$ boot2docker start
```

#### Docker実行
```bash
$ docker run ubuntu echo hello world
hello world
```

### [Ubuntu Trusty14.04(LTS)](http://docs.docker.com/installation/ubuntulinux/#ubuntu-raring-1304-and-saucy-1310-64-bit)の場合
#### Vagrantの準備
[ここ](https://github.com/phusion/open-vagrant-boxes)のイメージを使う

```bash
$ vagrant init phusion/ubuntu-14.04-amd64
```

_Vagrantfile_のネームサーバ対応
```
・・・
$script = <<SCRIPT
  echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null
SCRIPT

config.vm.provision "shell", inline: $script
・・・
```
#### Dockerセットアップ
```bash
$ vagrant ssh
$ sudo apt-get update
$ sudo apt-get install docker.io
$ sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
$ sudo sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io
```

#### Docker実行
```bash
$ sudo docker run -i -t ubuntu /bin/bash
Unable to find image 'ubuntu' locally
Pulling repository ubuntu
e54ca5efa2e9: Download complete
3db9c44f4520: Download complete
195eb90b5349: Download complete
c5881f11ded9: Download complete
463ff6be4238: Download complete
ebe4be4dd427: Download complete
511136ea3c5a: Download complete
6cfa4d1f33fb: Download complete
d7ac5e4f1812: Download complete
4d289a435341: Download complete
bac448df371d: Download complete
3af9d794ad07: Download complete
f127542f0b61: Download complete
dfaad36d8984: Download complete
fae16849ebe2: Download complete
994db1cb2425: Download complete
b7c6da90134e: Download complete
5796a7edb16b: Download complete
0f4aac48388f: Download complete
f86a812b1308: Download complete
47dd6d11a49f: Download complete
209ea56fda6d: Download complete
0b628db0b664: Download complete
2f4b4d6a4a06: Download complete
83ff768040a0: Download complete
6c37f792ddac: Download complete
root@6ab798be1467:/#
```
`ubuntu`イメージをダウンロードしてコンテナ内の`bash`が実行されれば成功。

#### 追加設定
```
$ exit
$ vagrant plugin install sahara
$ vagrant sandbox on
$ vagrant ssh
```

## <a name="3">Docker Hubはじめに</a>
### [Docker Hub](https://hub.docker.com/)とは
+ イメージのホスティング
+ ユーザー認証
+ ビルドトリガーやwebフックのようなワークフローとイメージビルドの自動化
+ GitHubとBitBucketとの統合

### ログイン
```bash
$ docker login
```
## <a name="4">ドカライズアプリケーション</a>
### Hello World!
```bash
$ sudo docker run ubuntu:14.04 /bin/echo 'Hello World'
Hello World
```
1. `docker run`で実行するコンテナを関連付ける。
1. `ubuntu:14.04`でイメージを特定する。これは実効するソースコンテナでこの場合はUbuntu 14.04のオペレーティングシステムイメージを使っている。
1. 指定したイメージがない場合Dockerは最初のあなたがホストしているイメージを探す。なければ[Docker Hub](https://hub.docker.com/)から公開イメージをダウンロードする。
1. `/bin/echo 'Hello World'`ではDcokerで新たに作られたUbuntu 14.04環境のコンテナで`/bin/echo`コマンドを内部的に実行して`Hello World!`を出力する。

### インタラクティブコンテナ
```bash
$ sudo docker run -t -i ubuntu:14.04 /bin/bash
root@9f2980a49b89:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@9f2980a49b89:/# pwd
/
root@9f2980a49b89:/#
root@9f2980a49b89:/# exit
exit
```
ここでは再び`ubuntu:14.04`イメージを使い`docker run`を実行している。`-t`フラグはコンテナ内で擬似ターミナルまたはターミナルを割り当てるオプションで`-i`フラグはコンテナ内で標準出力を介してインタラクティブに操作することを許すオプションです。  
ここではコンテナ内の`/bin/bash`を実行するように指定しています。この結果Bashシェルがコンテナ内で立ち上がります。

### Hello World!のデーモン化
```bash
$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
a6445476982ab88450bc631153ea44510f8baa700d3ae1fd1fda3768d0e6dae4
```
`-d`フラグはコンテナを実行してデーモン化するためバックグランド実行するオプション。
`/bin/sh -c "while true; do echo hello world; sleep 1; done`を実行することで`hello world`を出力し続けるショボイデーモンの出来上がりです。

しかし、なぜ`hello world`が出力されていないのかその代わりに長い文字列が出力されています。
`a6445476982ab88450bc631153ea44510f8baa700d3ae1fd1fda3768d0e6dae4`これはコンテナIDと呼ばれるもので作業する対象コンテナをユニークに特定します。

コンテナが実行中か確認します。確認には`docker ps`コマンドを使います。
```bash
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
a6445476982a        ubuntu:14.04        /bin/sh -c while tru   3 minutes ago       Up 3 minutes                            naughty_heisenberg
```
`docker ps`コマンドの結果短縮されたコンテナID`a6445476982a`とともに有益な情報を提供してくれます。ここから`ubuntu:14.04`イメージを使っていることコマンドが実行中であることステータスと自動的に割り当てられた名前`naughty_heisenberg`が分かります。

実行中のデーモンを確認するには`docker logs`コマンドを使います。
```
$ sudo docker logs naughty_heisenberg
hello world
hello world
hello world
・・・
```
デーモンを終了させるには`docker stop`コマンドを使います。
```bash
$ sudo docker stop naughty_heisenberg
naughty_heisenberg
```
正しく終了したかを`docker ps`コマンドで確認します。
```bash
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
稼働中のコンテナがなければ正しく停止されています。

## <a name="5">コンテナでの作業</a>

### Dockerクライアントが何ができるか確認

オプション無しで`docker`コマンドを実行すれば実行可能なコマンドが一覧表示されます。
```bash
$ sudo docker
Usage: docker [OPTIONS] COMMAND [arg...]
 -H=[unix:///var/run/docker.sock]: tcp://host:port to bind/connect to or unix://path/to/socket to use

A self-sufficient runtime for linux containers.

Commands:
    attach    Attach to a running container
    build     Build a container from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from the containers filesystem to the host path
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    export    Stream the contents of a container as a tar archive
    ・・・
```
### Dcokerコマンドの使い方を確認する
さらに特定のコマンドの詳細な使い方を確認することができます。
```bash
$ sudo docker attach

Usage: docker attach [OPTIONS] CONTAINER

Attach to a running container

  --no-stdin=false: Do not attach stdin
  --sig-proxy=true: Proxify all received signal to the process (even in non-tty mode)
```
または`--help`オプションでもできます。
```bash
$ sudo docker images --help

Usage: docker images [OPTIONS] [NAME]

List images

  -a, --all=false: Show all images (by default filter out the intermediate images used to build)
  --no-trunc=false: Don't truncate output
  -q, --quiet=false: Only show numeric IDs
  -t, --tree=false: Output graph in tree format
  -v, --viz=false: Output graph in graphviz format
```
### DockerでWebアプリケーションを実行する
```bash
sudo docker run -d -P training/webapp python app.py
```
`-d`フラグはバックグランド実行。`-P`フラグはDcokerに我々のホスト内のコンテナで必要なポートをマッピングするオプション。

`traing/webapp`イメージは予め作成された簡単なPython Flaskアプリケーションです。

最後にコマンドで`python app.py`を指定してコンテナ内で実行させています。これでwebアプリケーションが起動します。

### Webアプリケーションを確認する
```bash
$ sudo docker ps -l
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
177ccc8c5565        training/webapp:latest   python app.py       6 minutes ago       Up 6 minutes        0.0.0.0:49153->5000/tcp   sleepy_bardeen
```
`-l`オプションは最後に実行したコンテナの詳細を返します。  
`docker ps`コマンドは実行中のコンテナだけ表示します。もし停止中のコンテナも表示させたい場合は`-a`フラグを使ってください。

```bash
PORTS
0.0.0.0:49155->5000/tcp
```
`docker run`コマンドの`-P`フラグは我々のホスト内のイメージに他のポートを割り当てるオプションです。
このケースではポート5000(Pytho Flaskデフォルトポート)から49155に変更している。

Dockerではネットワークポートの割り当てを柔軟にできます。最後のケースでは`-P`フラグはコンテナ内のポート5000をローカルのDockerホストの上位ポート(49000 - 49900)に割り当てる`-p 5000`にショートカットオプションです。
また、`-p`フラグを使って特定のポートを指定した以下の様なDockerコンテナを作ることができます。
```bash
$ sudo docker run -d -p 5000:5000 training/webapp python app.py
```
これはローカルホスト内のポート5000にポート5000を割り当てます。なぜ１対１の割り当てを使わないかというと例えばローカルホスト内の複数のコンテナで同じPythonアプリを実行する場合それぞれにポートが割り当てていないと１つづつしか実行できないからです。

### ネットワークポートショートカット
```bash
$ sudo docker port sleepy_bardeen 5000
0.0.0.0:49153
```
このコマンドでポート5000にマッピングされているポート一覧を表示させることができます。

### Webアプリケーションログを確認する
```
$ sudo docker logs -f sleepy_bardeen
 * Running on http://0.0.0.0:5000/
```
`-f`フラグで`tail -f`みたいにログが表示できます。

### Webアプリケーションコンテナプロセスを確認する
```bash
$ sudo docker top sleepy_bardeen
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                1643                1577                0                   01:18               ?                   00:00:02            python app.py
```

### Webアプリケーションコンテナを調べる
```bash
$ sudo docker inspect sleepy_bardeen
[{
    "ID": "177ccc8c556526c62254eed3e994f17387831e0ba8bb7f9bf1aa45681a2f92df",
    "Created": "2014-06-23T01:18:09.0791517Z",
    "Path": "python",
    "Args": [
        "app.py"
    ],
    "Config": {
        "Hostname": "177ccc8c5565",
        "Domainname": "",
        "User": "",
        "Memory": 0,
        "MemorySwap": 0,
        "CpuShares": 0,
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "PortSpecs": null,
        "ExposedPorts": {
            "5000/tcp": {}
        },
        ・・・
```
また特定の要素を確認することもできます。
```bash
$ sudo docker inspect -f '{{ .NetworkSettings.IPAddress }}' sleepy_bardeen
172.17.0.2
```

### Webアプリケーションを止める
```bash
$ sudo docker stop sleepy_bardeen
sleepy_bardeen
```

### Webアプリケーションを再起動する
```bash
$ sudo docker start sleepy_bardeen
sleepy_bardeen
```

### Webアプリケーションコンテナを削除する
```bash
$ sudo docker stop sleepy_bardeen
sleepy_bardeen
$ sudo docker rm sleepy_bardeen
sleepy_bardeen
```

## Tips
### Dockerにつながらない場合
[no answer from server](https://github.com/dotcloud/docker/issues/3603)  
以下の作業をしてネームサーバを更新したあとDockerデーモンを再起動する。
```bash
$ vagrant provision
$ vagrant ssh
$ sudo service docker.io restart
```

# 参照
+ [Docker](http://www.docker.com/)
+ [Documentation](http://docs.docker.com/)
+ [Docker-friendly Vagrant base boxes](https://github.com/phusion/open-vagrant-boxes)
