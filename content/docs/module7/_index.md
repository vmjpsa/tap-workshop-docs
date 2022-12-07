---
weight: 7
title: "セキュアなサプライチェーン"
---

# セキュアなサプライチェーン

node-hello-tanzu と同様、リモートリポジトリにあるファイルを入力値として、tanzu コマンドでtanzu-java-web-app アプリケーションをデプロイします。これはJava Spring アプリケーションです。

```shell
tanzu apps workload create tanzu-java-web-app --git-repo https://github.com/vmware-tanzu/application-accelerator-samples --sub-path tanzu-java-web-app --git-branch main --type web --label apps.tanzu.vmware.com/has-tests=true --label app.kubernetes.io/part-of=tanzu-java-web-app --annotation autoscaling.knative.dev/minScale=1 -y
```
```
coder@code-server-696f98d647-vx77m:~/node-hello-tanzu$ tanzu apps workload create tanzu-java-web-app --git-repo https://github.com/vmware-tanzu/application-accelerator-samples --sub-path tanzu-java-web-app --git-branch main --type web --label apps.tanzu.vmware.com/has-tests=true --label app.kubernetes.io/part-of=tanzu-java-web-app --annotation autoscaling.knative.dev/minScale=1 -y
Create workload:
      1 + |---
      2 + |apiVersion: carto.run/v1alpha1
      3 + |kind: Workload
      4 + |metadata:
      5 + |  labels:
      6 + |    app.kubernetes.io/part-of: tanzu-java-web-app
      7 + |    apps.tanzu.vmware.com/has-tests: "true"
      8 + |    apps.tanzu.vmware.com/workload-type: web
      9 + |  name: tanzu-java-web-app
     10 + |  namespace: tap-workshop-01
     11 + |spec:
     12 + |  params:
     13 + |  - name: annotations
     14 + |    value:
     15 + |      autoscaling.knative.dev/minScale: "1"
     16 + |  source:
     17 + |    git:
     18 + |      ref:
     19 + |        branch: main
     20 + |      url: https://github.com/vmware-tanzu/application-accelerator-samples
     21 + |    subPath: tanzu-java-web-app

Created workload "tanzu-java-web-app"

To see logs:   "tanzu apps workload tail tanzu-java-web-app"
To get status: "tanzu apps workload get tanzu-java-web-app"

```

node-hello-tanzu をデプロイした時とコマンドが似ていますが、ここで最も重要なのは、--label apps.tanzu.vmware.com/has-tests=true オプションです。これは単なるラベルにすぎませんが、どのサプライチェーンを使うかをラベルで表現します。TAP では、ワークロードオブジェクトにつけられたラベルを見て、どのサプライチェーンが使われるかが決まります。ここではapps.tanzu.vmware.com/has-tests=true というラベルを付けていますが、これに対応するサプライチェーンが、この後確認するsource-test-scan-to-url になります。

ワークロードの状態を確認し、Ready になるまで数分待ちます。node-hello-tanzu よりもビルドに多少時間がかかります。

```shell
tanzu apps workload get tanzu-java-web-app
```

その間にサプライチェーンがどのような状態になっているか、TAP GUI を確認してみましょう。

{{< figure src="supply-chain-secure.png" width="100%">}}

source-to-url サプライチェーンと異なり、いくつか追加のステップがあることが分かります。今回はsource-test-scan-to-url サプライチェーンを使いましたが、これは[Tekton](https://tekton.dev/) によるテストや[Grype](https://github.com/anchore/grype) によるイメージスキャンが追加のステップとして定義されている組み込みのサプライチェーンになります。今回はTekton のテストは空ですが、当然カスタマイズすることができます。

今回デプロイしたアプリケーションは脆弱性を多く含みますが、危険度が大きい脆弱性を検知した場合にデプロイを自動的に中止することができます（ワークショップの都合上、全ての脆弱性をこのサプライチェーンにおいては無視しました）。その後、ソースコードを編集し、対象の脆弱性を含むようなライブラリの利用を避ける、バージョンを上げるなどの対処をすることで、そのようなセキュリティリスクを含むアプリケーションが顧客に提供されることを防ぐことができます。

アプリケーションの準備ができ、URL が発行されたらアクセスし、Greetings from Spring Boot + Tanzu! と表示されることを確認します。

{{< figure src="tanzu-java-web-app.png" width="100%">}}

最後に、デプロイしたアプリケーションを削除します。

```shell
tanzu apps workload delete tanzu-java-web-app -y
```





