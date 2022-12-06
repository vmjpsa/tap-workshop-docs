---
weight: 4
title: "Live Update"
---

# Live Update
Application Accelerator では、Web UI を使用して、プロジェクトテンプレートを .zip ファイルとしてローカルマシンにダウンロードすることができます。今回は、このようなテンプレートがgit 上でホストされているケースを想定し、皆さんがおなじみのgit コマンドを使ってテンプレートをダウンロードし、アプリケーションの開発を開始します。

さっそくテンプレートをダウンロードしましょう。

```shell
git clone https://github.com/vmjpsa/node-hello-tanzu.git
cd node-hello-tanzu
```

今回はHello Tanzu! と画面に表示するだけの非常に単純なWeb アプリケーションで、アプリケーション開発を試してみます。


TAP ではLive Update と呼ばれる便利な機能を提供しています。Live Update では、Tilt の機能を活用して、より高速な反復開発を実現することができます。フォルダの中にTiltfile_sample というファイルがあることに注目してください。Tiltfile は Application Accelerator によってプロジェクトテンプレートの一部として作られたファイルです。Tiltfile スクリプトは、Tanzu Application Platform を利用して、3つの主要なタスクを実行します。
* コンテナイメージの作成
* アプリケーションの実行と、アプリケーションへアクセスするための Kubernetes リソースを作成
* ソースコードの編集に伴うアプリケーションのLive Update を有効にする

これらの各ステップを詳しく見ていきましょう。

### コンテナイメージの作成

Tanzu Application Plaform の最も強力な機能の1つは、**Tanzu Build Service** です。これは開発者が提供するアプリケーションのソースコードからランタイムコンテナを自動生成します。これらのコンテナイメージを作成するために、CNCFプロジェクトの**Cloud Native Buildpacks**を活用しています。Tanzu は、最新の言語ランタイムの依存性を提供しながら、セキュリティとパフォーマンスのためにコンテナイメージを最適化するためのビルドパックを提供します。Tanzu のビルドパックは、Java、.NET Core、Node、Go、Python、PHPなど、最も一般的なプログラミング言語に対応しています。また、その他の言語のニーズがある場合は、オープンソースコミュニティが他の多くの言語用のビルドパックを提供しています。

開発者のTaro は、Tanzu Build Service のメリットを活用することで、自分で Dockerfile を作成したり、メンテナンスしたりする必要がなく、コンテナの安全性やパッチの適用を保証するための作業に追われることもなくなります。彼は、コンテナランタイムの生成ではなく、ソースコードを書く本来の作業に集中することができます。Tanzu Build Service の他の利点については後ほど見ていきます。

### Kubernetes リソースの作成

開発環境が Kubernetes クラスタの場合、Tanzu Application Platform はコンテナイメージをデプロイ・実行するために必要な Kubernetes リソースを作成し、ローカルマシンから Kubernetes にデプロイされたアプリケーションにアクセスできるようにします。この環境では、Tanzu Application Platform に組み込まれているオープンソースプロジェクト Knative をランタイムに使用しています。

### Live Update

Tilt では、開発環境にアプリケーションをデプロイするもので、初回実行時には完了まで数分かかりますが、次回以降の時間は短縮されるため、実行中のアプリケーションを数秒で更新することができます。どのように動作するか見てみましょう。

まずは、自身の環境から、作成したコンテナイメージをプッシュできるようにコンテナレジストリにログインをします。

```shell
docker login $REPO_URL -u $REPO_USER -p $REPO_PASSWORD
```

次に、Tiltfile を作成します。先にみたように、本来Tiltfile はテンプレートに含むべきですが、このワークショップでは環境の都合上ユーザーごとにTiltfile を編集する必要があるため、下記コマンドで作成します。少し長い入力ですが、実行しているのはTiltfile_sample をベースとしたTiltfile の作成です。


```shell
cat <<EOF > Tiltfile
SOURCE_IMAGE = os.getenv("SOURCE_IMAGE", default='${REPO_URL}/tap-workshop/node-hello-tanzu-${SESSION_NUMBER}') # REPO_URL like harbor.tanzu.lab/tap-workshop/node-hello-tanzu
LOCAL_PATH = os.getenv("LOCAL_PATH", default='.')
NAMESPACE = os.getenv("NAMESPACE", default='')
K8S_TEST_CONTEXT = os.getenv("K8S_TEST_CONTEXT", default='')

allow_k8s_contexts(K8S_TEST_CONTEXT)

k8s_custom_deploy(
    'node-hello-tanzu',
    apply_cmd="tanzu apps workload apply -f config/workload.yaml --live-update" +
        " --local-path " + LOCAL_PATH +
        " --source-image " + SOURCE_IMAGE +
        " --namespace " + NAMESPACE +
        " --yes >/dev/null" +
        " && kubectl get workload node-hello-tanzu --namespace " + NAMESPACE + " -o yaml",
    delete_cmd="tanzu apps workload delete -f config/workload.yaml --namespace " + NAMESPACE + " --yes" ,
    deps=['.'],
    container_selector='workload',
    live_update=[
        fall_back_on(['package.json']),
        sync('.', '/workspace')
]
)

k8s_resource('node-hello-tanzu', port_forwards=["8080:8080"],
    extra_pod_selectors=[{'carto.run/workload-name': 'node-hello-tanzu', 'app.kubernetes.io/component': 'run'}])

allow_k8s_contexts('${CLUSTER_NAME}')

EOF
```

Tiltfile ができたら、いよいよLive Update の機能を試します。下記のtanzu コマンドを入力します。このコマンドではgit clone したローカルのファイルをベースにワークロードの作成の準備をします。

{{/* TODO コマンド要確認 */}}
```shell
tanzu apps workload apply node-hello-tanzu --live-update --local-path . -s $REPO_URL/tap-workshop/node-hello-tanzu-${SESSION_NUMBER} -y
```

下記のようなログが出力されるはずです。

```
Publishing source in "." to "$REPO_URL/tap-workshop/node-hello-tanzu-${SESSION_NUMBER}"...
Published source
Create workload:
      1 + |---
      2 + |apiVersion: carto.run/v1alpha1
      3 + |kind: Workload
      4 + |metadata:
      5 + |  name: node-hello-tanzu
      6 + |  namespace: tap-workshop-01
      7 + |spec:
      8 + |  source:
      9 + |    image: $REPO_URL/tap-workshop/node-hello-tanzu-${SESSION_NUMBER}:latest@sha256:abcdef...
```

tanzu コマンド入力後、tilt up でワークロードがデプロイされます。コマンド入力後、**s キーを入力してアプリケーションのデプロイまでのログを確認します。**

```shell
tilt up
```

ログは下記のように出力されます。先述したように、初回のビルドは少し時間がかかります。

```
coder@code-server-696f98d647-vx77m:~/node-hello-tanzu$ tilt up
Tilt started on http://localhost:10350/
v0.30.11, built 2022-11-07

(space) to open the browser
(s) to stream logs (--stream=true)
(t) to open legacy terminal mode (--legacy=true)
(ctrl-c) to exit
Tilt started on http://localhost:10350/
v0.30.11, built 2022-11-07

Initial Build
Loading Tiltfile at: /home/coder/node-hello-tanzu/Tiltfile
Successfully loaded Tiltfile (1.415443ms)
ERROR: [] "msg"="Reconciler error" "error"="Failed to update API server: create configmaps/node-hello-tanzu-disable: configmaps.tilt.dev \"node-hello-tanzu-disable\" already exists" "controller"="tiltfile" "controllerGroup"="tilt.dev" "controllerKind"="Tiltfile" "Tiltfile"={"name":"(Tiltfile)"} "namespace"="" "name"="(Tiltfile)" "reconcileID"="caf681ca-5e03-4d6f-9e76-c6e0468da938"
WARNING: Writing Kubernetes config: could not create any of the following paths: /run/user/1000/tilt-dev/tilt-default/cluster
node-hello-t… │ 
node-hello-t… │ Initial Build
node-hello-t… │ STEP 1/1 — Deploying
node-hello-t… │      Running cmd: tanzu apps workload apply -f config/workload.yaml --live-update --local-path . --source-image $REPO_URL/tap-workshop/node-hello-tanzu-01 --namespace tap-workshop-01 --yes >/dev/null && kubectl get workload node-hello-tanzu --namespace tap-workshop-01 -o yaml
node-hello-t… │      Objects applied to cluster:
node-hello-t… │        → node-hello-tanzu:workload
node-hello-t… │ 
node-hello-t… │      Step 1 - 1.88s (Deploying)
node-hello-t… │      DONE IN: 1.88s 
node-hello-t… │ 
node-hello-t… │ 
node-hello-t… │ Tracking new pod rollout (node-hello-tanzu-00001-deployment-66df8f56b6-2bp5q):
node-hello-t… │      ┊ Scheduled       - <1s
node-hello-t… │      ┊ Initialized     - (…) Pending
node-hello-t… │      ┊ Ready           - (…) Pending
node-hello-t… │ [queue-proxy] {"severity":"INFO","timestamp":"2022-12-05T11:37:01.427999526Z","logger":"queueproxy","caller":"queue/main.go:197","message":"Starting queue-proxy","commit":"3666ce7","knative.dev/key":"tap-workshop-01/node-hello-tanzu-00001","knative.dev/pod":"node-hello-tanzu-00001-deployment-66df8f56b6-2bp5q"}
```

デプロイが完了したら、アプリケーションにアクセスしてみましょう。アプリケーションは今Kubernetes 上にデプロイされていますが、下記スクリーンショットのように、Code Server ポートフォワーディングの機能でローカル端末からアクセスできます。

## 画像

Ctrl + クリック、またはURL を直接コピーし、別タブでアクセスします。このタブは開いたままにします。

次に、アプリケーションのコードの中心であるindex.js ファイルを開き、**Hello Tanzu!** を**Hello Tanzu!!!** などと任意に変更してみてください。すると、tilt の出力が下記のように更新されるはずです。

```
......
node-hello-t… │ 1 File Changed: [index.js]
node-hello-t… │ Will copy 1 file(s) to container: [node-hello-tanzu-00002-deployment-5ffd767b87-78m56/workload]
node-hello-t… │ - '/home/coder/node-hello-tanzu/index.js' --> '/workspace/index.js'
node-hello-t… │   → Container node-hello-tanzu-00002-deployment-5ffd767b87-78m56/workload updated!
```

再びアプリケーションのタブを確認し、画面を更新すると、上記の変更が即座に反映されているはずです。

## 画像

ターミナルにてCtrl+C を入力し、LiveUpdate を中断しましょう。

今回は単純なNodeJS のアプリでしたが、例えばSpring （Java） のアプリ等でも同様な機能を利用することができます。この際、Dockerfile はどこにも登場しなかったことに注意してください。TAP のコンポーネントであるTanzu Build Service によって、自動的にNodeJS のアプリと判定され、ベストプラクティスに沿った形でコンテナ化され、それがKubernetes 上に展開されています。開発者としては、アプリケーションの本質的な部分により時間を割くことができます。

また、今回はLive Update を利用するためにtanzu とtilt の2 つのコマンドを使いましたが、TAP では**VS Code やIntelliJ といったエディタの拡張機能としてもLive Update の機能を提供しており、これらのコマンドを入力することなく利用することができます**（今回はラボの制約のためCLI 入力としました）。

## 画像


## 削除必要？？

これでLive Update の機能を確認できました。最後に、ログでターミナルが汚れていると思いますので、確認し終わったらclear してしまいましょう。

```shell
clear
```













Tanzu コマンド ラインを使用して、最初のデプロイの準備ができたことを確認します。

```execute-2 
tanzu apps workload get spring-sensors
```

これは、関連する Pod や Knative Services とともに、サプライチェーンを通じて進行するワークロードの状態を報告します。
デプロイの準備が完了すると、下部にこのような作業用の URL が表示されます。もし Knative Services の READY の欄に Ready と表示されない場合は、完了するまでコマンドを何度か繰り返して確認してください。
```
Knative Services
NAME             READY   URL
spring-sensors   Ready   http://spring-sensors-tap-demos-w07-s003.tap.corby.cc
```
ターミナルウィンドウで URL をクリックすると、アプリケーションが表示されます。

**Ready になる前に次に進まないでください。**

では、ここからはアプリケーションのコードを変更してみましょう。現在、バナーのテキストは「Spring Sensors」と表示されています。バナーを他のものに変更してみましょう。

```editor:select-matching-text
file: spring-sensors/src/main/java/org/tanzu/demo/DemoController.java
text: "Spring Sensors"
```

選択したテキストは、コードエディタで入力して置き換えるか、以下をクリックして自動的に文字列の置換を適用することができます。

```editor:replace-text-selection
file: spring-sensors/src/main/java/org/tanzu/demo/DemoController.java
text: Hot New Banner
```

このコード変更をすると、実行中のコンテナに自動的にパッチが適用されます。10秒以内にターミナルウィンドウでアプリケーションが再起動するのが確認できます。アプリケーションが動作しているブラウザのタブに移動して、リフレッシュしてください。
コードの変更が自動的に適用されていることがわかります。

これで、Cody は集中してコード開発を続けることができます。彼はコード変更を反映させる作業を都度することなく、次の機能のコーディングを開始し、実行中のコンテナですぐに段階的な結果を確認しながら開発を続けることができます。Live Update の詳細は[ドキュメント](https://docs.tilt.dev/live_update_reference.html)をご参照ください。


では、他に Tanzu Application Platform を使ってできる機能を引き続き見ていきましょう。






