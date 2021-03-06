# ホストマシンで起動しているDockerコンテナにクライアントのVSCodeから接続

[part1](Docker%20part1.%20dockerで仮想環境の用意.md)ではホストマシンにDockerを用いて仮想環境を用意する手順を説明した。ここでは自分のPCなどのVisual Studio Codeから、リモートでDockerコンテナの起動しているホストマシンにアクセスして操作するための設定について解説する。

## 前提

- クライアントからホストへのssh接続はできるようにしておく
- ホスト・クライアントともにDockerがインストールされており、どちらもローカルで利用できる状態にある

ssh接続については調べたら参考になるサイトがたくさん出てくるはずなのでそれを参考に設定しよう。WindowsにDockerをインストールする際は注意する点がいくつかあるので、以下のサイトあたりを参考に。

- [WindowsでDocker環境を試してみる](https://qiita.com/fkooo/items/d2fddef9091b906675ca)
- [Docker for Windowsをインストール](https://ops.jig-saw.com/tech-cate/docker-for-windows-install)

---

## ①VSCodeに拡張機能のインストール

以下の拡張機能インストール

- [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
- [Remote SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)
- [Remote Container](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

---

## ②VSCodeの設定

以下の中から好きな方法で

[公式ガイド](https://code.visualstudio.com/docs/remote/containers-advanced#_developing-inside-a-container-on-a-remote-docker-host)

- SSHプロトコル

  ssh-agentに秘密鍵の登録

  ```bash
  ssh-add -K [鍵のパス]
  ```

  VSCodeのsettings.jsonに以下の設定を必要な箇所を訂正して追加

  ```config
  {
    "docker.host": "ssh://[リモートでのユーザ名]@[リモートマシンのアドレス]"
    // "docker.host": "ssh://tanaka@192.168.10.10"   example
  }
  ```

- TCPプロトコルの使用

  VSCodeでsshのconfigを開き、ホストへの接続設定の部分にLocalForwardの項目を追記

  以下はポート23750番を使う場合の例

  ```bash
  # 例えばもともと以下のような設定だった場合
  Host hostexample1
    HostName 192.168.10.10
    IdentityFile C:/Users/yourname/.ssh/id_rsa
    User Tanaka
    Port 22

  # LocalForward 23750 /var/run/docker.sockを追記
  Host hostexample1
    HostName 192.168.10.10
    IdentityFile C:/Users/yourname/.ssh/id_rsa
    User Tanaka
    Port 22
    LocalForward 23750 /var/run/docker.sock
  ```

  VSCodeのsettings.jsonに以下を追記

  ```json
  {
    "docker.host": "tcp://[リモートマシンのアドレス]:23750"
  }
  ```

- それ以外の方法
  
  [公式ガイド](https://code.visualstudio.com/docs/remote/containers-advanced#_developing-inside-a-container-on-a-remote-docker-host)見てくれ。

---

## ③VSCodeから接続

1. クライアント上でVSCodeを起動し左下の __><__ マークからホストマシンにssh接続
1. <font color="Red">ssh接続したVSCodeのウィンドウとは別のウィンドウで</font>左のリモートエクスプローラーから接続したいコンテナを選択 -> 右クリックで __Open__ __Folder__ __in__ __Container__

ssh接続したウィンドウは使わないものの、<font color="Red">開きっぱなしでないとコンテナとの接続が切れる</font>。これについては現状原因不明。最小化でもしておく。一応多段sshを用いることで解決する~~らしいが、そもそもコンテナ側にopenssh-serverの設定がされてない場合が多いため、Dockerイメージのビルドの段階からやり直す必要がある上にこの設定がかなり面倒~~。多段sshを用いた設定方法は[こちら](Docker%20番外編%20dockerコンテナに多段ssh接続.md)。

---
