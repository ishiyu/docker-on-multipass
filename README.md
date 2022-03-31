# docker-on-multipass

## Install

```sh
brew install --cask multipass
```

## Launch Image

結構時間はかかります。

```sh
multipass launch --name docker-vm --cloud-init multipass-launch-init.yml docker
```

## Alias を設定する

mac から multipass のインスタンスに対してコマンドを発行する際に `multipass exec docker-vm docker` と毎回打つのはしんどいので、alias を設定する

https://multipass.run/docs/using-aliases

```sh
# ex> multipass alias instance:command [name]
multipass alias docker-vm:docker md

# 出力されたPATHを .zshrc に入れる
echo PATH="$PATH:/Users/<user-name>/Library/Application Support/multipass/bin" >> ~/.zshrc
```


## マウント（MacとUbuntsuの共有フォルダを設定する）

立ち上げるたびに毎回必要。

```sh
multipass mount ./share docker-vm:~/share
```

## apt-get の更新とライブラリのインストール

```sh
multipass shell docker-vm

# ここからは ubuntsu 内で実行
sudo apt-get update
sudo apt-get install -y unzip

# docker-compose
sudo apt-get install docker-compose

# aws-rotate-iam-keys
sudo add-apt-repository ppa:rhyeal/aws-rotate-iam-keys -y
sudo apt-get update
sudo apt-get install aws-rotate-iam-keys
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

https://zenn.dev/s_ryuuki/articles/db7eb23dde7084
https://discourse.ubuntu.com/t/running-a-container-with-the-docker-workflow-in-multipass/26806
