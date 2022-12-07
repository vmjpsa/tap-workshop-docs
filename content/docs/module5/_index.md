---
weight: 5
title: "Git を使ったデプロイ"
---

## Git とは

Git はいわゆる分散型バージョン管理システムです。アプリケーションのソースコードを適切にバージョン管理したり、変更を追跡したりするための仕組みの1 つです。また、その分散型という仕組みから、それぞれの開発者がローカルリポジトリで開発を進め、変更を確定（コミット）し、その変更点をリモートリポジトリ（例えばGithub）に送信（プッシュ）するという開発形態をとることができ、チーム開発において様々なメリットを提供します。

{{< figure src="git.png" width="100%">}}


## Git を使ったデプロイ

さて、先ほどはLive Update の機能でアプリケーションをデプロイしました。これは個人で開発を進めるためには強力な機能ですが、チームで開発する場合はやはりGit のリモートリポジトリを活用したいです。そこで、今回はGit のリモートリポジトリへのコミットをトリガとしてアプリケーションのデプロイをしてみましょう。

まず、現時点でカレントディレクトリはnode-hello-tanzu のはずですが、そうでない場合は下記コマンドでカレントディレクトリを変更してください。

```shell
cd $HOME/node-hello-tanzu
```

次にgit の設定をします。ユーザーのメールアドレスと名前を下記コマンドで設定しますが、特に指定はないため何でも構いません（下記はサンプルです）。**ご自身のe-mail アドレスを入力する必要はございません。**

```shell
git config --global user.email tap-workshop@test.lab
git config --global user.name tap-workshop
```

先のLive Update で、index.js の「Hello, Tanzu!」を「Hello, Tanzu!!!」等に書き換えました。そこでこの変更をローカルリポジトリにコミットします（「Hello, Tanzu!」のままだと変更がないとみなされるため、git add の前に任意に変更してください）。

```shell
git add -A
git commit -m "My first commit!"
```

変更をリモートリポジトリにプッシュします。

```shell
git push https://$GIT_USER:$GIT_PASSWORD@$GIT_URL/tap-workshop/node-hello-tanzu-$SESSION_NUMBER main
```

これで準備ができたので、下記コマンドでアプリケーションをデプロイしましょう。

```shell
tanzu apps workload create node-hello-tanzu --git-repo https://$GIT_URL/tap-workshop/node-hello-tanzu-$SESSION_NUMBER.git --git-branch main --label app.kubernetes.io/part-of=tanzu-node-hello-tanzu --type web --annotation autoscaling.knative.dev/minScale=1 -y
```
このコマンドは少々複雑ですが、オプションとしてgit リポジトリやブランチなどを指定しています。 --annotation autoscaling.knative.dev/minScale=1 の意味は、Knative Serving の最小インスタンス数を1 以上に設定し、時間経過でPod の数が0 になることを防止しています。

--type web はワークロードの種類になります。TAP1.3 ではweb, server, worker タイプをサポートしています。詳細は下記ドキュメントをご参照ください。

[https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.3/tap/GUID-workloads-workload-types.html](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.3/tap/GUID-workloads-workload-types.html)

tanzu コマンドの結果として、下記のようなログが出力されるはずです。

```
coder@code-server-69dd88bff7-d9x22:~/node-hello-tanzu$ tanzu apps workload create node-hello-tanzu --git-repo https://$GIT_URL/tap-workshop/node-hello-tanzu-$SESSION_NUMBER.git --git-branch main --label app.kubernetes.io/part-of=tanzu-node-hello-tanzu --type web --annotation autoscaling.knative.dev/minScale=1 -y
Create workload:
      1 + |---
      2 + |apiVersion: carto.run/v1alpha1
      3 + |kind: Workload
      4 + |metadata:
      5 + |  labels:
      6 + |    app.kubernetes.io/part-of: tanzu-node-hello-tanzu
      7 + |    apps.tanzu.vmware.com/workload-type: web
      8 + |  name: node-hello-tanzu
      9 + |  namespace: tap-workshop-01
     10 + |spec:
     11 + |  params:
     12 + |  - name: annotations
     13 + |    value:
     14 + |      autoscaling.knative.dev/minScale: "1"
     15 + |  source:
     16 + |    git:
     17 + |      ref:
     18 + |        branch: main
     19 + |      url: https://$GIT_URL/tap-workshop/node-hello-tanzu-01.git

Created workload "node-hello-tanzu"

To see logs:   "tanzu apps workload tail node-hello-tanzu"
To get status: "tanzu apps workload get node-hello-tanzu"
```

指示通り、デプロイの状態を確認してみましょう。

```shell
tanzu apps workload get node-hello-tanzu 
```

```
coder@code-server-696f98d647-vx77m:~/node-hello-tanzu$ tanzu apps workload get node-hello-tanzu
📡 Overview
   name:   node-hello-tanzu
   type:   web

💾 Source
   type:     git
   url:      https://$GIT_URL/tap-workshop/node-hello-tanzu-01.git
   branch:   main

📦 Supply Chain
   name:   source-to-url

   RESOURCE           READY     HEALTHY   TIME   OUTPUT
   source-provider    Unknown   Unknown   3s     GitRepository/node-hello-tanzu
   image-provider     False     Unknown   3s     not found
   config-provider    False     Unknown   3s     not found
   app-config         False     True      3s     not found
   service-bindings   False     True      3s     not found
   api-descriptors    False     True      3s     not found
   config-writer      False     Unknown   3s     not found

🚚 Delivery

   Delivery resources not found.

💬 Messages
   Workload [MissingValueAtPath]:     waiting to read value [.status.artifact.url] from resource [gitrepository.source.toolkit.fluxcd.io/node-hello-tanzu] in namespace [tap-workshop-01]
   Workload [HealthyConditionRule]:   condition with type [Ready] not found on resource status

🛶 Pods
   NAME                                 READY   STATUS    RESTARTS   AGE
   node-hello-tanzu-build-1-build-pod   0/1     Pending   0          0s

To see logs: "tanzu apps workload tail node-hello-tanzu"

```

デプロイが完了するまでしばらく待ちましょう。下記コマンドでデプロイまでのログを確認できます（中断はCtrl+c）。

```shell
tanzu apps workload tail node-hello-tanzu
```

アプリケーションのデプロイが完了すると、下記のように表示されますので、🚢 Knative Services の欄で出力されたURL にアクセスするとLive Update と同様index.js の文字列が出力されるはずです（URL をCtrl+クリックで別タブで開くことができます）。

```
coder@code-server-696f98d647-vx77m:~/node-hello-tanzu$ tanzu apps workload get node-hello-tanzu
📡 Overview
   name:   node-hello-tanzu
   type:   web

💾 Source
   type:     git
   url:      https://$GIT_URL/tap-workshop/node-hello-tanzu-01.git
   branch:   main

📦 Supply Chain
   name:   source-to-url

   RESOURCE           READY   HEALTHY   TIME   OUTPUT
   source-provider    True    True      83s    GitRepository/node-hello-tanzu
   image-provider     True    True      63s    Image/node-hello-tanzu
   config-provider    True    True      56s    PodIntent/node-hello-tanzu
   app-config         True    True      56s    ConfigMap/node-hello-tanzu
   service-bindings   True    True      56s    ConfigMap/node-hello-tanzu-with-claims
   api-descriptors    True    True      56s    ConfigMap/node-hello-tanzu-with-api-descriptors
   config-writer      True    True      49s    Runnable/node-hello-tanzu-config-writer

🚚 Delivery
   name:   delivery-basic

   RESOURCE          READY   HEALTHY   TIME   OUTPUT
   source-provider   True    True      23s    ImageRepository/node-hello-tanzu-delivery
   deployer          True    True      16s    App/node-hello-tanzu

💬 Messages
   No messages found.

🛶 Pods
   NAME                                                 READY   STATUS      RESTARTS   AGE
   node-hello-tanzu-00001-deployment-54bcc768bb-sgrnp   2/2     Running     0          23s
   node-hello-tanzu-build-1-build-pod                   0/1     Completed   0          87s
   node-hello-tanzu-config-writer-ts27k-pod             0/1     Completed   0          57s

🚢 Knative Services
   NAME               READY   URL
   node-hello-tanzu   Ready   http://node-hello-tanzu.tap-workshop-01.***

To see logs: "tanzu apps workload tail node-hello-tanzu"
```

{{< figure src="hello-tanzu.png" width="100%">}}

アプリケーションのデプロイが確認出来たら、ソースコードを変更してみます。index.js の「Hello Tanzu!!!」を「Hello Git Tanzu!!!」と変更します（Code Server 上でのファイルの変更は自動で保存されるためCtrl+s などは必要ありません）。

{{< figure src="hello-git-tanzu-edit.png" width="100%">}}

ここでは、Live Update と異なりアプリケーションの更新は発生しません。先ほどのtanzu apps workload create コマンドでは、リモートリポジトリへの変更を追跡し、その変更をトリガに再ビルドをさせようとしていますので、再びコードをコミットし、リモートリポジトリにプッシュします。

```shell
git add -A
git commit -m "My Second commit!"
git push https://$GIT_USER:$GIT_PASSWORD@$GIT_URL/tap-workshop/node-hello-tanzu-$SESSION_NUMBER main
```

変更が反映されるまでしばらく待ってください。下記コマンドでアプリケーションの再ビルドが走ることを追跡することもできます。

```shell
tanzu apps workload get node-hello-tanzu 
```

```shell
tanzu apps workload tail node-hello-tanzu
```

先と同じように、アプリケーションのURL が発行されたらアクセスし、表示が「Hello Git Tanzu!!!」に変わっていることを確認します。


{{< figure src="hello-git-tanzu.png" width="100%">}}




