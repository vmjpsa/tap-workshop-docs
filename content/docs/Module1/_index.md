---
weight: 1
title: "ワークショップ環境"
---

## ワークショップ環境の概要

本ワークショップではCode Server と呼ばれるWeb サービスとし提供されるVS Code 環境を利用します。アクセスURL とパスワードに関しては講師からお伝えします。

ワークショップを提供する環境は以下のようになっています。

{{< figure src="workshop-architecture.png" width="100%">}}
TAP は既にインストール済みなので、ワークショップ参加者がTAP のインストールや構成変更をする必要はありません。参加者ごとに提供されるCode Server と名前空間の中でアプリケーションのビルドやデプロイを実施します。

## Code Server の使い方
さっそくCode Server にアクセスしてみましょう。下記のような画面が表示された場合は、Yes, I trust the authors を選択してください。

{{< figure src="coder-trust.png" width="100%">}}

すると下記のような画面になるはずです。
{{< figure src="coder-top.png" width="100%">}}

ここで、Ctrl + J キー、または下記の画像のように、ハンバーガーボタン→ Terminal → New Terminal で、ターミナルを表示させます。

{{< figure src="coder-terminal.png" width="100%">}}

画面構成が下記のようになったら、ワークショップを進める準備は完了です。基本的なコマンド操作はターミナル上で行い、一部ファイル編集はエディタ上で行います。

{{< figure src="coder-howtouse.png" width="100%">}}

## 本マニュアルの使い方
マニュアルと講師の指示通り操作します。ターミナルでの操作は下記のように表現しています。右上の📋（コピーボタン）をクリックすれば、クリップボードにコマンドをコピーできます。試しにやってみましょう。

```shell
ls
```

コマンドの貼り付けは右クリックまたはCtrl+v キーで行います。Code Server のターミナルでは右クリックができない場合がありますが、その場合はCtrl+v で行ってください。

{{< figure src="coder-ls.png" width="100%">}}

この時、ブラウザでクリップボードへのアクセス許可に関する通知が出た場合は許可をしてください。

{{< figure src="chrome-clipboard.png" width="60%">}}

一方で、単なるログの共有等には下記のような書き方をしていることに注意してください（コピーボタン📋はありません）。この場合はコマンドではないのでターミナルで入力する必要はないです。

```
coder@code-server-696f98d647-vx77m:~$ ls
install-from-tanzunet.sh  kubeconfig
```

Code Server の使い方に慣れたら、次に進みましょう。