# MacをiPadから使う
自宅にMacがあり、iPadのみを持って外出している場合でも、iPadからMacに遠隔接続し、操作することを可能にします。

## 前提
1. MacにHomebrewをインストールしていること
2. Macは常時起動していること
3. MacとiPadはインターネットに接続していること

## 1 Tailscaleをインストールする
TailscaleとはVPNを提供するアプリです。MacとiPadが同一VPNに接続することにより仮想LANを構築し、安全性を確保します。
```mermaid
graph TD
  subgraph home[家]
  mac(Mac)
  end
  subgraph out[外出先]
  ipad(iPad)
  end
  vpn(Tailscale) --- home
  vpn --- out
```
まずはMacにTailscaleをインストールします。
```zsh
brew install --cask tailscale
brew install tailscale
```
GUI版とCUI版の両方をインストールしています。
インストールしたGUI版を開き、アカウントの新規作成/ログインをして下さい。

次にiPadにもApp Store経由でTailscaleをインストールします。Mac版と同じアカウントでログインして下さい。

TailscaleでMacとiPadの両方がリストされていれば仮想LANは構築できています。

## 2 hbbs/hbbrサーバーを構築する
いずれも遠隔接続に必要なサーバーです。
Dockerコンテナとして提供されているので、まずはDockerをインストールします。
```zsh
brew install --cask docker
```
Docker Desktopを起動して下さい。dockerdというデーモンが起動し、コンテナを動かせるようになります。

次にサーバー情報を保存するためのディレクトリを作成します。
```zsh
mkdir ~/Documents/rustdesk-server
```

hbbsサーバーを構築します。
```zsh
docker run -d --name hbbs --restart unless-stopped \
-v "$HOME/Documents/rustdesk-server:/root" \
-p 21115:21115 \
-p 21116:21116 \
-p 21116:21116/udp \
rustdesk/rustdesk-server hbbs
```

hbbrサーバーを構築します。
```zsh
docker run -d --name hbbr --restart unless-stopped \
  -v "$HOME/Documents/rustdesk-server:/root" \
  -p 21117:21117 \
  -p 21118:21118 \
  -p 21119:21119 \
  rustdesk/rustdesk-server hbbr
```

docker runコマンドにより、rustdeskサーバーのhbbsとhbbrを起動しています。-dオプションは、サーバーをバックグラウンドで動かすためのものです。--restart unless-stoppedを付けたので、明示的に停止させない限りは自動的にサーバーが再起動します。

## 3 接続に必要な情報を得る
1. 仮想LANにおけるMacのIPアドレス（＝hbbs/hbbrサーバーのIPアドレス）
2. hbbsサーバーが発行する公開鍵

の2つの情報が必要です。

### 3.1 IPアドレス
MacのIPアドレスはTailscaleから知ることができます。GUIでMacの情報を見ても良いですし、
```zsh
tailscale ip -4
```
の値を見ても良いです。

### 3.2 公開鍵
公開鍵は2章で作成したディレクトリに出力されています。`id_ed25519.pub`というファイルをテキストエディタで開いてください。

## 4 Rustdeskをインストールする

## 5 接続する
