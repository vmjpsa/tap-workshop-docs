---
weight: 2
title: "TAP の概要"
---

## TAP が求められる背景

Kubernetes（K8s） はすでに世界中で利用されており、そのユーザー数は毎年どんどん増えています。日本においてもそのユーザー数は拡大しており、単なる検証フェーズではなく、本番環境での利用を進めているユーザーも増えています。それほど多くのユーザーがどんどん流入しているのは K8s 及びコンテナが有用だからなのですが、**本格的に使い始めてから多くのユーザーが「K8s は難しい」という印象を抱きます。**

以下の図を見てください。
{{< figure src="k8s-dev.png" width="100%">}}

この図では K8s を比較的最近使い始めた入門ユーザーのアプリ開発からデプロイの流れ（上）と、エンタープライズ利用でのアプリ開発からデプロイまでの流れ（下）を記載しています。簡単に言ってしまうと、最初に K8s を使う場合は「アプリが動けばOK」で考えてしまうのですが、実際に大事なアプリを展開するには「バグったアプリにしない」「セキュリティ要件を満たす」「デプロイに失敗させない」などといった懸念事項が山のように出てきて、それを満たすための必要なアクションやツールがどんどん増えていきます。これは K8s 特有というよりも、従来の VM ベースの開発/運用のときと同じです。

この状況は K8s を使い始めた全ての組織が向き合う話ですので、これらを解決するためのツールや導入手順などが K8s の管理団体である [CNCF](https://www.cncf.io/) を中心として存在しています。これらはいわゆる「Kubernetes エコシステム」と呼ばれるものです。どういったツールがあるかは「CNCF Cloud Native Interactive Landscape」にまとめられてあり、圧倒されるほどのプロジェクト数が存在しています。そして、それらを導入する手順の手引書として「CNCF Trail Map」が公開されており、かなり長い道のりが提示されています。

これらの道具を使って K8s の開発/運用の流れを構築するのですが、その作業も毎回人力で kubectl コマンドなどを叩くのではなく、自動化などのソリューションを利用するのが一般的です。この自動化のレベルも何段階かあり、おおよそ以下の図のレベル感となります。

{{< figure src="pipeline.png" width="100%">}}

図の左が初期段階で、右に行くほど成熟してきます。一番左が自動化を全く使わずに毎回手動でコンテナをビルドして、デプロイ作業をするという段階です。K8s の基礎を学ぶだけでなく、自動化がカバーできない、たまに実施する操作や例外的な作業はマニュアルで実施します。ただ、先ほど紹介したエンタープライズな開発からデプロイまでを全てコマンドの手打ちで実施すると、面倒ですし操作ミスによるトラブルなども発生します。なにより、**K8s を適切に理解している人しか、開発からデプロイまでの作業をできなくなってしまいます。**


その次のレベルがパイプラインの導入です。図にある Jenkins などが有名ですが、マニュアルで実施していた人間の作業をそのままパイプラインのスクリプトに落とし込むことで、人間が手を動かさなくても人間相当のことをパイプラインが愚直に実施してくれます。これはマニュアル作業で K8s の開発からデプロイまでを実施できるひとが、深い知識なしに手順を自動化できるという強みがあります。なぜなら、人間が打っていたコマンドラインを単にスクリプト化すればよいだけだからです。開発するアプリやパイプラインの環境が少ない場合はこれでよいかもしれません。ただ、**K8s を本格的に導入しはじめると「数十のアプリを数十のパイプラインで自動化する」といった状況になってきます。こういった状況で原始的な手順ベースのパイプラインを作ってしまうと、各パイプラインの維持メンテナンスコストに悩まされます。**


そういったマニュアル作業や依存性の問題が辛くなってきたユーザーは、汎用パイプラインを卒業して多くの場合は K8s フレンドリーなツール群を使い始めます。有名なものは図にある Tekton や ArgoCD などです。たとえば Tekton によるビルドは K8s 方式の YAML 定義で実現できますし、ArgoCD によるデプロイは同様に Git リポジトリ上にある YAML 定義の構成ファイルに勝手に K8s クラスタをシンクさせるといった動きをします。Jenkins で人間の操作を自動化にそのまま落とし込むとパイプラインがフルオーダーメイドで調整しにくくなる一方、TektonやArgoCDなどはパラメーターをちょっと変更するセミオーダーメイドなので導入が簡単です。K8s のパワーユーザーはこういったツールを他にも大量に組み合わせてパイプラインを作ることが多いです。ただ、この方式にも問題があります。それは、「1つ1つの道具としては優秀なものの、それらを人間が手組みで組み合わせるという作業に辛さを覚える」という問題です。NHKの有名な教育番組のピタゴラスイッチというものがありますが、ちょうどあれを作るかのような感覚でパイプラインを組むようなものです。

そういったユーザーが最後にたどり着くのが K8s のパイプラインフレームワークであり、VMware 製品ではこの記事で紹介する Tanzu Application Platform （TAP）となります。パイプラインフレームワークはパイプラインを構築しやすくするための仕組みであり、TAP の場合はまるでレゴでパイプラインを組み立てるかのように「フレームワークに道具を簡単にはめこめる」「道具は変更可能」ということを実現しています。こうすることで、手で調整するピタゴラスイッチなパイプラインから、ミニ四駆のような規格化されたパーツを目的に沿って組み合わせる形のパイプラインを構築できるようになります。

**TAPを導入することで、開発者はプラットフォームの些末なことを気にする必要がなくなり、アプリケーションの本質に注力することができるようになります。そして、インフラエンジニアは、開発者に対してよりよいプラットフォームを提供することを、1から100まで全て自分たちで手配する必要がなくなり、その大部分を TAP に任せることができるようになります。**


## TAP の特徴

TAP の目指す世界感をおおまかにお伝えしたので、次はそれをどのように実現するかという具体的な話に移りたいと思います。まず、前提知識の話となりますが、開発者と運用者の生産性を高めるには、コンテナにしろレガシーなプラットフォームにしろ、以下のような開発/運用形態をとることが必要です。

{{< figure src="loop.png" width="100%">}}

先にお伝えしたように、エンタープライズでのアプリケーション開発/運用は動けばよいというものではなく、セキュリティなど様々な考慮事項があり、それらの課題をクリアする必要があります。ただ、それらの課題全てを開発者が意識して開発するとなると、それはコードを書くことが本業の開発者にたいして多くを求めすぎているかもしれません。そのようなことを求めると**一部の超優秀な開発者を除く8割の普通の開発者はついていけなくなります。** そういった状態になることを防ぐために、「開発者は深いことを考えずにコードを書くことに集中できる」ことと「運用者は開発者が開発したコードを、エンタープライズな要件を満たすかチェックし、満たせていれば運用に移り、満たせていなければ開発者に問題箇所を教える」ということを両立しなければいけません。このような「簡単さ」と「慎重さ」は相反するように思えるかもしれませんが、それは上記図のように**Inner Loop （開発者が効率よくコードを書ける仕組み）**と**Outer Loop（運用者が安全にシステムを運用できる仕組み）** の2本立てにすることで解決可能です。

Inner Loop は開発者本来の仕事を助けるために、「コードを書く。ビルドする。テストする」ということを短いサイクルで実現するための仕組みを提供します。コードにバグが含まれている場合などは、この短いサイクルでの開発と確認により、すでに問題箇所を特定して修復することが容易となります。

一方、Outer Loop はさきほどのエンタープライズなパイプラインを実現するための仕組みを提供します。開発者が開発したコードが、ただ動くだけではなく、きちんとセキュリティなどの要件を満たせているかチェックし、満たせている場合は「本番環境で動いているアプリの置き換え」という一般的には重要で難しい対応をなんなく実現するための手法を提供します。

TAPの場合は、この Inner Loop と Outer Loop を以下の図のような流れで支援しています。

{{< figure src="supplychain.png" width="100%">}}
 
そして、この支援の裏側には以下の図のような様々なカテゴリに属するツール群が用意されています。

{{< figure src="tap.png" width="100%">}}


本来はこれらのツール群は、k8s ユーザー自身が OSS として調達し、それらを組み合わせて目的を達成する必要があります。ただ、そのようなことを実現しようと思うと、膨大な数の検証と、アップグレードの際にちゃんと動くかテストをするといった作業負担が非常に大きいです。ただ、TAP の場合はそれらの作業を VMware がお客様の代わりに実施してあり、「このツールをフレームワークに沿って使えば動きます」という当たり前なものの実現することがなかなか大変な要件をいとも簡単に達成することができます。


さて、実際にTAP は開発者や運用者をどのように支援できるのでしょうか？TAP の提供する機能とは具体的にどのようなものなのでしょうか？

本ワークショップを通して、TAP を使って実際にアプリケーションの開発を体験してみることで、それらの強力な機能を学んでいきましょう！

