# docker-on-multipass

## Install

```sh
brew install --cask multipass
brew install docker docker-compose
```

## SSH ファイルを作成する

`hogefuga@example.com` のメールアドレスは適宜変えてください。

```sh
ssh-keygen -t rsa -b 4096 -C "hogefuga@example.com" -f ~/.ssh/multipass_docker
```

## multipass の構成ファイルを作成する

```sh
# 作成した.pubファイルを変数に読み込む
AUTHORIZED_KEYS=$(cat ~/.ssh/multipass_docker.pub )
# Docker Compose Version
# version はここから確認 https://github.com/docker/compose/releases
DOCKER_COMPOSE_VER=v2.3.4
# ヒアドキュメントでファイルを作成する
cat > ./multipass_docker.yml << _EOF_
---
locale: en_US.UTF8
timezone: Asia/Tokyo
package_upgrade: true
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh-authorized-keys:
      - ${AUTHORIZED_KEYS}

packages:
  - ca-certificates
  - curl
  - gnupg
  - lsb-release
  - docker-ce
  - docker-ce-cli
  - containerd.io
  - software-properties-common
  - avahi-daemon

runcmd:
  - sudo curl -fsSL https://get.docker.com | sudo bash
  - sudo systemctl enable docker
  - sudo systemctl enable -s HUP ssh
  - sudo groupadd docker
  - sudo usermod -aG docker ubuntu
  - sudo apt-get install docker-compose -y
  # aws-rotate-iam-keys
  - sudo add-apt-repository ppa:rhyeal/aws-rotate-iam-keys -y
  - sudo apt-get update
  - sudo apt-get install aws-rotate-iam-keys -y

_EOF_
```

## Launch Image

結構時間はかかります。
（CPU/メモリ/ストレージ容量は任意で変えてください）

```sh
# multipass launch -c CPU -m メモリ量 -d ストレージ容量 -n インスタンス名 --cloud-init ./multipass_docker.yml
multipass launch -c 4 -m 8GB -d 40GB -n docker --cloud-init ./multipass_docker.yml
```

## ここまでの確認

### SSH接続
```sh
# 念の為、以下を実行してログイン記録を削除。
ssh-keygen -R docker.local
# sshファイルを使用してMultipassのインスタンスにログイン
ssh -i ~/.ssh/multipass_docker ubuntu@docker.local

# ...
# instance につながるはず
# ...

# docker のインストール確認
docker info
# docker-compose のインストール確認
docker-compose -v
```

## Alias を設定する

mac から multipass のインスタンスに対してコマンドを発行する際に `multipass exec docker-vm docker` と毎回打つのはしんどいので、alias を設定する

https://multipass.run/docs/using-aliases

```sh
# ex> multipass alias instance:command [name]
multipass alias docker:docker md

# 出力されたPATHを .zshrc に入れる
echo PATH="$PATH:/Users/<user-name>/Library/Application Support/multipass/bin" >> ~/.zshrc
```


## マウント（MacとUbuntsuの共有フォルダを設定する）

立ち上げるたびに毎回必要。

```sh
multipass mount ./share docker:~/share
```

## 

## FAQ

### launch failed: Operation canceled が出る

https://github.com/canonical/multipass/issues/2288

以下のコマンドで multipass を再起動する
```sh
sudo launchctl stop com.canonical.multipassd
sudo launchctl start com.canonical.multipassd
```

## Mac から docker を実行するコマンド

```sh
# multipass exec <multipass instance name> docker
multipass exec docker-vm docker
multipass exec docker-vm docker-compose
```


## 参考リンク

https://qiita.com/mm_sys/items/c64086a57e5643c102d2
https://zenn.dev/s_ryuuki/articles/db7eb23dde7084
https://discourse.ubuntu.com/t/running-a-container-with-the-docker-workflow-in-multipass/26806
