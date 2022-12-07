---
weight: 3
title: "Application Accelerator"
---

## Application Accelerator

今回は、アプリケーション開発者であるあなたのストーリーを基にラボを進めます。あなたの会社では現在 Tanzu Application Platform が稼働しており、あなたはこのプラットフォームを使ってコンテナ化したアプリケーションの配信をしたいと考えています。

あなたは、コンテナインフラストラクチャの専門家ではありません。あくまで開発者であり、人気のある言語やフレームワークを使ってビジネスアプリケーションを書くことに関しての専門家です。

あなたは、開発プロジェクトを容易に始めるためのテンプレートを必要としています。あなたは、開発プロジェクトの編集やビルド、実行に必要なすべてのコンポーネントを含む、厳選されたテンプレートをダウンロードしたいと思っています。そのようなテンプレートがあればオンボーディングがシンプルだからです。

また、あなたはDockerfile、Kubernetesリソース、およびその他のインフラストラクチャのアーティファクトを自分で記述し、維持する責任をなるべく負いたくありません。このようなインフラストラクチャの準備や運用によって、開発者はアプリケーションロジックを書いたり、ストーリーを提供したりする時間が少なくなってしまいます。

このような開発者の要望に対して、Tanzu Application Platform では、あなたが会社で使用が承認されたテンプレートを閲覧し、利用するための簡単な方法を提供します。これは Application Accelerator と呼ばれ、Tanzu Application Platform の Web インターフェース（TAP GUI）でホストされています。

エンタープライズアーキテクトはApplication Accelerator を使って、組織内の開発者にコードや設定のためのエンタープライズに準拠したテンプレートを提供します。開発者はApplication Accelerator を使用して、企業標準に準拠したプロジェクトを作成したり、それにアクセスしたりできます。

さて、さっそくApplication Accelerator を確認してみましょう。
TAPGUI のURL は下記コマンドで確認することができます。**新しいブラウザタブを開いて**TAP GUIにアクセスします。左上の>> をクリックし、Create を選択します。

```shell
echo $TAP_GUI_URL
```

{{< figure src="tap-gui-top.png" width="100%">}}

{{< figure src="tap-gui-create.png" width="100%">}}

これで、あなたは自分が取り組みたいプログラミング言語(Java やNodeなど)とアプリケーションの種類（Web, メッセージ駆動型, 関数など）に適したテンプレートを見つけることができます。

{{< figure src="tap-gui-catalog.png" width="100%">}}

## Accelerator Templates
Tanzu Java Web App テンプレートで作業してみましょう。Tanzu Java Web App のタイルにある choose ボタンを押します。UI でプロジェクトテンプレートをカスタマイズするためのオプションパラメータが提供されていますが、デフォルトのままNextを押してください。選択した内容を確認するプロンプトが表示されるので、Generate Acceleratorをクリックします。

{{< figure src="tap-gui-app1.png" width="100%">}}

{{< figure src="tap-gui-app2.png" width="100%">}}

{{< figure src="tap-gui-app3.png" width="100%">}}

その後、EXPLORE ZIP FILE をクリックして、テンプレートフォルダの中身を見てみましょう。

{{< figure src="tap-gui-app4.png" width="100%">}}

{{< figure src="tap-gui-app5.png" width="100%">}}


ファイルを見てみると、Java やSpring を知っている方であればおなじみのディレクトリ構造を確認できます。あなたがスターターアプリケーションで使用するソースコードファイルと、Tanzu Application Platform でアプリケーションを操作するためのいくつかの追加のファイルで構成されていることがわかります。これらの追加ファイルについては後のセクションで詳しく見ていきます。確認が終了したら右下のCLOSE をクリックして終了します。

{{<hint info>}}
このテンプレートはtanzu CLI でもダウンロードできます。
{{</hint>}}

{{<hint info>}}
ここではこのテンプレートは使用しないので、ファイルをダウンロードする必要はありません。
{{</hint>}}


