Docker入門
===
# 目的
Dockerの習得を目的とするため[公式サイトのドキュメント](http://docs.docker.com/)を超訳。

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
+ [Dockerイメージでの作業](#6)
+ [コンテナをリンクする](#7)
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

## <a name="6">Dockerイメージでの作業</a>
### ホストのイメージを一覧表示する
```bash
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              12.10               c5881f11ded9        4 days ago          172.2 MB
ubuntu              quantal             c5881f11ded9        4 days ago          172.2 MB
ubuntu              13.04               463ff6be4238        4 days ago          169.4 MB
ubuntu              raring              463ff6be4238        4 days ago          169.4 MB
ubuntu              13.10               195eb90b5349        4 days ago          184.7 MB
ubuntu              saucy               195eb90b5349        4 days ago          184.7 MB
ubuntu              12.04               ebe4be4dd427        4 days ago          210.6 MB
ubuntu              precise             ebe4be4dd427        4 days ago          210.6 MB
ubuntu              14.04               e54ca5efa2e9        4 days ago          276.5 MB
ubuntu              latest              e54ca5efa2e9        4 days ago          276.5 MB
ubuntu              trusty              e54ca5efa2e9        4 days ago          276.5 MB
training/webapp     latest              31fa814ba25a        3 weeks ago         278.8 MB
ubuntu              10.04               3db9c44f4520        8 weeks ago         183 MB
ubuntu              lucid               3db9c44f4520        8 weeks ago         183 MB
```
このケースでは`ubuntu`イメージは複数にバージョンを保持しているまたそれぞれのバージョンは以下のようにTAGで参照できる。
```bash
ubuntu:14.04
```
タグで指定されたバージョンのイメージを実行するには以下のようにする。
```bash
$ sudo docker run -t -i ubuntu:14.04 /bin/bash
```
Ubuntu 12.04のイメージなら
```bash
$ sudo docker run -t -i ubuntu:12.04 /bin/bash
```
もしタグを指定しない場合は`ubuntu:latest`イメージをデフォルで使います。

### 新しいイメージを取得する
```bash
$ sudo docker pull centos
Pulling repository centos
0c752394b855: Download complete
539c0211cd76: Download complete
511136ea3c5a: Download complete
34e94e67e63a: Download complete
```
それぞれのレイヤーイメージプルダウンされたことが確認できます。そしてダウンロードを待つことなくコンテナを実行できます。
```bash
$ sudo docker run -t -i centos /bin/bash
```

### イメージを探す
[Docker Hub](https://hub.docker.com/)サイトから検索できます。
または`docker search`コマンドでイメージを探すことができます。
```bash
$ sudo docker search sinatra
NAME                                   DESCRIPTION                                     STARS     OFFICIAL   TRUSTED
wiggles/sinatra                                                                        1
huahuiyang/sinatra                                                                     1
training/sinatra                                                                       1
bmorearty/handson-sinatra              handson-ruby + Sinatra for Hands on with D...   0
・・・
```
### イメージをプルする
```bash
$ sudo docker pull training/sinatra
Pulling repository training/sinatra
f0f4ab557f95: Download complete
3c59e02ddd1a: Download complete
511136ea3c5a: Download complete
5e66087f3ffe: Download complete
4d26dd3ebc1c: Download complete
d4010efcfd86: Download complete
99ec81b80c55: Download complete
1fd0d1b3b785: Download complete
08ebafdba908: Download complete
fbc9a83d93d9: Download complete
3e76c0a80540: Download complete
be88c4c27e80: Download complete
bfab314f3b76: Download complete
e809f156dc98: Download complete
ce80548340bb: Download complete
79e6bf39f993: Download complete
```
これでチームは自分たちのコンテナとしてイメージを使うことができるようになります。
```bash
$ sudo docker run -t -i training/sinatra /bin/bash
root@24fb13a7df27:/#
```
### 自分用イメージを作る
プルしたイメージは幾つか変更やアップデートが必要なのでそのままでは使えない。イメージのアップデートと作成には２つの方法があります。
1. イメージから作ったコンテナをアップデートしてその結果をコミットする。
1. `Dockerfile`を使って作成する。

### イメージのアップデートとコミット
```bash
$ sudo docker run -t -i training/sinatra /bin/bash
root@8045f93568c8:/# gem install json
Fetching: json-1.8.1.gem (100%)
Building native extensions.  This could take a while...
Successfully installed json-1.8.1
1 gem installed
Installing ri documentation for json-1.8.1...
Installing RDoc documentation for json-1.8.1...
root@8045f93568c8:/# exit
exit
```
作りたいカスタムコンテナができたので`docker commit`コマンドでコンテナイメージのコピーをコミットします。
```bash
$ sudo docker commit -m="Added json gem" -a="k2works" 8045f93568c8 k2works/sinatra:v2
dd9bd08eba288de838d924517f3c94c42f5d041c4058fdb8d6df549f811995d1
```
`-m`フラグはバージョン管理システムのコミットのようにコミットメッセージを特定するオプションです。`-a`フラグは作成者の名前です。

また、新しく作りたいイメージを`8045f93568c8`(先に記録されたID)のコンテナをから指定しイメージのターゲットを特定しています。
```
k2works/sinatra:v2
```
これは作成したイメージのユーザー`k2works`で構成されています。また、イメージ名も指定しています今回はオリジナルの名前`sinatra`をそのまま使っています。最後に`v2`イメージダグをつけています。
```bash
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
k2works/sinatra     v2                  dd9bd08eba28        20 seconds ago      451.1 MB
・・・
```
新しく作成したイメージを使うには以下の操作を実行ます。
```bash

$ sudo docker run -t -i k2works/sinatra:v2 /bin/bash
root@1bd21b2cf944:/#
```

### `Dockerfile`からイメージを作る
ディレクトリと`Dockerfile`を作成
```bash
$ cd /vagrant/
$ sudo mkdir sinatra
$ cd sinatra/
$ sudo touch Dockerfile
```
_Dockerfile_を編集
```
# This is a comment
FROM ubuntu:14.04
MAINTAINER Kate Smith <ksmith@example.com>
RUN apt-get -qq update
RUN apt-get -qqy install ruby ruby-dev
RUN gem install sinatra
```

+ `FROM`はDockerにイメージソースを伝える。
+ `MAINTAINE`はイメージをメンテナンスする人。
+ 'RUN'はイメージ内で実行するコマンド。

続いて`Dockerfile`と`docker build`コマンドを使ってイメージを作ります。
```bash
$ sudo docker build -t="k2works/sinatra:v2" .
Uploading context  2.56 kB
Uploading context
Step 0 : FROM ubuntu:14.04
 ---> e54ca5efa2e9
Step 1 : MAINTAINER Kate Smith <ksmith@example.com>
 ---> Running in c79cb06db257
 ---> 8f7f1a073054
Step 2 : RUN apt-get -qq update
 ---> Running in 299e70b68f1c
 ---> 69fab425df1a
Step 3 : RUN apt-get -qqy install ruby ruby-dev
 ---> Running in 2de88d382882
 ・・・
 Step 4 : RUN gem install sinatra
 ---> Running in 41354f79db2c
unable to convert "\xC3" to UTF-8 in conversion from ASCII-8BIT to UTF-8 to US-ASCII for README.rdoc, skipping
unable to convert "\xC3" to UTF-8 in conversion from ASCII-8BIT to UTF-8 to US-ASCII for README.rdoc, skipping
Successfully installed rack-1.5.2
Successfully installed tilt-1.4.1
Successfully installed rack-protection-1.5.3
Successfully installed sinatra-1.4.5
4 gems installed
Installing ri documentation for rack-1.5.2...
Installing ri documentation for tilt-1.4.1...
Installing ri documentation for rack-protection-1.5.3...
Installing ri documentation for sinatra-1.4.5...
Installing RDoc documentation for rack-1.5.2...
Installing RDoc documentation for tilt-1.4.1...
Installing RDoc documentation for rack-protection-1.5.3...
Installing RDoc documentation for sinatra-1.4.5...
 ---> 8a78b9fca8dc
Successfully built 8a78b9fca8dc
Removing intermediate container c79cb06db257
Removing intermediate container 299e70b68f1c
Removing intermediate container 2de88d382882
Removing intermediate container 41354f79db2c
```
`-t`フラグで`k2works`ユーザーに属することを指定します。そしてレポジトリ名を`sinatra`でタグを`v2`に指定しています。
また今回はカレントディレクトリとしてして`.`を指定した`Dockerfile`の場所を別の場所に指定することもできます。

続いて`Dockerfile`で実行されている内容をステップバイステップで見ていきます。各ステップごとに`docker commit`が実行されているように見えます。
そして最後に`8a78b9fca8dc`イメージが残され残りのイメージが削除されていることが確認できます。

さらにその新しいイメージからコンテナを作ることができます。
```bash
$ sudo docker run -t -i k2works/sinatra:v2 /bin/bash
root@4b12aad7d137:/#
```

### イメージにタグを付ける
```bash
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
k2works/sinatra     v2                  8a78b9fca8dc        16 minutes ago      385.6 MB
・・・
$ sudo docker tag 8a78b9fca8dc k2works/sinatra:devel
$ sudo docker images k2works/sinatra
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
k2works/sinatra     v2                  8a78b9fca8dc        17 minutes ago      385.6 MB
k2works/sinatra     devel               8a78b9fca8dc        17 minutes ago      385.6 MB
```

### Docker Hubにプッシュする
```bash
$ sudo docker push k2works/sinatra
The push refers to a repository [k2works/sinatra] (len: 2)
Sending image list
Pushing repository k2works/sinatra (2 tags)
511136ea3c5a: Image already pushed, skipping
d7ac5e4f1812: Image already pushed, skipping
2f4b4d6a4a06: Image already pushed, skipping
83ff768040a0: Image already pushed, skipping
6c37f792ddac: Image already pushed, skipping
e54ca5efa2e9: Image already pushed, skipping
8f7f1a073054: Image successfully pushed
69fab425df1a: Image successfully pushed
dfb6dcb9b017: Image successfully pushed
8a78b9fca8dc: Image successfully pushed
Pushing tag for rev [8a78b9fca8dc] on {https://registry-1.docker.io/v1/repositories/k2works/sinatra/tags/v2}
Pushing tag for rev [8a78b9fca8dc] on {https://registry-1.docker.io/v1/repositories/k2works/sinatra/tags/devel}
```
### ホストのイメージを削除する
```bash
$ sudo docker rmi training/sinatra
Untagged: training/sinatra:latest
```

## <a name="7">コンテナをリンクする</a>

### ネットワークポートマッピングリフレッシャー
```bash
$ sudo docker run -d -P training/webapp python app.py
$ docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
79f7862d8e9c        training/webapp:latest   python app.py       10 seconds ago      Up 7 seconds        0.0.0.0:49153->5000/tcp   sad_euclid
```
`-P`フラグは自動的に49000から49900の間の上位ポートをランダムに割り当てる。

```bash
$ docker run -d -p 5000:5000 training/webapp python app.py
```
`-p`フラグは特定のポートを割り当てる。
```bash
$ sudo docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```
`-p`フラグはまた`localhost`といった特定のインターフェースに対応づけることもできる。

```bash
$ docker run -d -p 127.0.0.1::5000 training/webapp python app.py
53b95a1b4127a9f8825f9cbe811bf60f1993e90a2673497e3f2922eefb29f708
$ docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                       NAMES
53b95a1b4127        training/webapp:latest   python app.py       10 seconds ago      Up 8 seconds        127.0.0.1:49153->5000/tcp   distracted_davinci
5838eee2f776        training/webapp:latest   python app.py       4 minutes ago       Up 3 minutes        127.0.0.1:5000->5000/tcp    romantic_fermi
```
また、`localhost`上でもポート動的割り当てができる。

```bash
$ docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
$ docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                                NAMES
642116e5a966        training/webapp:latest   python app.py       2 minutes ago       Up 7 seconds        5000/tcp, 127.0.0.1:5000->5000/udp   cocky_morse
```
また、UDPポートにも割り当てができる。

```bash
$ docker port distracted_davinci 5000
127.0.0.1:49153
```
`docker port`ショートカットで現在どのポートが割り当てられているか確認できる。

### Dockerコンテナリンキング
Dockerには複数のコンテナのリンクを許可してそれぞれの情報を共有する機能を持っています。
Dockerリンキング機能は親コンテナが子コンテナに関する情報を選択できる親子関係を作ることができます。

### コンテナ名
コンテナ名を指定できる。この機能は２つの利便性を提供します。
1. webアプリケーションに`web`というように管理しやすくなる。
1. `web`コンテナが`db`コンテナにリンクするといったようにDockerに参照ポイントを提供する。

`--name`フラグでコンテナに名前をつけることができます。
```bash
$ docker run -d -P --name web training/webapp python app.py
$ docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
f03664bf7c6d        training/webapp:latest   python app.py       3 minutes ago       Up About a minute   0.0.0.0:49154->5000/tcp   web
```
また、`docker inspect`でコンテナ名を取得できます。
```bash
$ docker inspect -f "{{ .Name }}" f03664bf7c6d
/web
```

### コンテナリンク
リンクはコンテナに相互の安全なやりとりを許します。リンクを使うには`--link`フラグを使います。

```bash
$ docker run -d --name db training/postgres
$ docker ps
CONTAINER ID        IMAGE                      COMMAND                CREATED             STATUS              PORTS               NAMES
235f0a3d463e        training/postgres:latest   su postgres -c '/usr   3 minutes ago       Up About a minute   5432/tcp            db
```
まず、`db`の名前でPostgreSQLデータベースのコンテナを作ります。
```bash
$ docker run -d -P --name web --link db:db training/webapp python app.py
```
続いて`db`にリンクする`web`コンテナを作ります。
`--link`フラグのフォーマットは以下。
```
--link name:alias
```
`name`はコンテナ名で`alias`は名前にリンクした別名です。

`docker ps`でコンテナのリンク状況を確認してみましょう。
```bash
$ docker ps
CONTAINER ID        IMAGE                      COMMAND                CREATED             STATUS              PORTS                     NAMES
97de7cebf622        training/webapp:latest     python app.py          2 minutes ago       Up 24 seconds       0.0.0.0:49154->5000/tcp   web
235f0a3d463e        training/postgres:latest   su postgres -c '/usr   7 minutes ago       Up 4 minutes        5432/tcp                  db,web/db
```
ここで`NAMES`コラムの`db/web`の部分が`web`コンテナが`db`コンテナにリンクされていることを表しています。
ここでは親の`db`コンテナが子の`web`コンテナにアクセスすることができます。それを実現するためにDockerは外部ポートを開放することなく安全なトンネルを作ります。
リンクが貼られている限り特別な操作をする必要はありません。Dockerは相互接続情報を以下の方法でやりとりします。

+ 環境編集
+ `/etc/host`ファイルの更新

Dockerが最初の設定する環境変数
```
$ docker run --rm --name web2 --link db:db training/webapp env
HOME=/
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=0ceeec30ad75
DB_PORT=tcp://172.17.0.11:5432
DB_PORT_5432_TCP=tcp://172.17.0.11:5432
DB_PORT_5432_TCP_ADDR=172.17.0.11
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_PROTO=tcp
DB_NAME=/web2/db
DB_ENV_PG_VERSION=9.3
```
それぞれの環境変数は最初に指定した`alias`である`DB`がプリフィックスされています。もし`alias`を`db1`としたなら環境変数は`DB1_`でプリフィックスされます。
これらの環境変数を`db`コンテナ内のデータベースと接続するための設定に使うことができます。接続は安全です、プライベートかつ`web`コンテナだけとリンクしているから。

加えて、Dockerはリンクしている親コンテナへのエントリーを`/etc/hosts`ファイルに追加します。
```bash
$ docker run -t -i --link db:db  training/webapp:latest /bin/bash
root@49a13556ccef:/opt/webapp# cat /etc/hosts
172.17.0.26     49a13556ccef
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.11     db
```
dbエイリアスで参照されたIPアドレスを確認することができます。
```bash
root@49a13556ccef:/opt/webapp# apt-get install -yqq inetutils-ping
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package netbase.
(Reading database ... 8961 files and directories currently installed.)
Unpacking netbase (from .../netbase_4.47ubuntu1_all.deb) ...
Selecting previously unselected package inetutils-ping.
Unpacking inetutils-ping (from .../inetutils-ping_2%3a1.8-6_amd64.deb) ...
Setting up netbase (4.47ubuntu1) ...
Setting up inetutils-ping (2:1.8-6) ...
root@49a13556ccef:/opt/webapp# ping db
PING db (172.17.0.11): 48 data bytes
56 bytes from 172.17.0.11: icmp_seq=0 ttl=64 time=0.197 ms
56 bytes from 172.17.0.11: icmp_seq=1 ttl=64 time=0.415 ms
56 bytes from 172.17.0.11: icmp_seq=2 ttl=64 time=0.250 ms
```
ホストエントリーの`172.17.0.5`を使って`db`コンテナと`ping`コマンドで通信できたことが確認できます。

## Tips
### Dockerにつながらない場合
[no answer from server](https://github.com/dotcloud/docker/issues/3603)  
以下の作業をしてネームサーバを更新したあとDockerデーモンを再起動する。
```bash
$ vagrant provision
$ vagrant ssh
$ sudo service docker.io restart
```
#### boot2dockerを使っていてconnection refusedになる場合
ネームサーバ8.8.8.8を[追加](https://ml-wiki.sys.affrc.go.jp/help/settings/dns/client/macosx-leopard)する。
MacOS Xの_/etc/resolv.conf_に以下が追加されているか確認する。
```
nameserver 8.8.8.8
```
そして再起動
```bash
$ boot2docker restart
```


# 参照
+ [Docker](http://www.docker.com/)
+ [Documentation](http://docs.docker.com/)
+ [Docker-friendly Vagrant base boxes](https://github.com/phusion/open-vagrant-boxes)
