---
weight: 8
title: "TAP のセキュリティ"
---

# TAP のセキュリティ

TAP のセキュリティとして「Shift Left」の発想を取り入りています。一言でいえば**ソフトウェア開発のなるべく早い段階でテストをしていくという考え方です。** 

{{< figure src="shift-left.png" width="100%">}}

[https://www.launchableinc.com/blog/shift-left-automated-tests-with-launchable/](https://www.launchableinc.com/blog/shift-left-automated-tests-with-launchable/)

「セキュリティ = アンチウィルスソフト」のように、エージェントをインストールするようなイメージをもっている方も多いと思います。しかし、このアプローチですが、「いじめの問題を学校中に監視カメラを配置する」という解決策に似ています。以下のような問題が発生します。

* 監視カメラの目が届かない場所がうまれるかもしれない
  * セキュリティツールが万能でない可能性がある
* 監視カメラが学校生活を息苦しい場所にして転校が多くなるかもしれない
  * 開発者がセキュリティ設定が緩やかな環境に逃げていく可能性がある
* 本質的ないじめ問題を解決していない
  * 本質的なセキュリティ問題を解決していない

このような背景から、セキュリティの問題を根本的に発生しにくくし、監視カメラもしくはエージェントすらいらないレベルまでセキュリティが当たり前に設定されている手法が重要になってきます。その上で開発生産性を落とさないアプローチとして、「 Shift Left 」のアプローチが注目されています。TAP では、セキュリティを開発の流れ、すなわち TAP のサプライチェーン に組み込んだ形で提供しています。

{{< figure src="security.png" width="100%">}}

この サプライチェーン の過程ですが、主に、3つの機能で表現することができます。

* Scan : TAP では、二段階のスキャンが実施されます。ソースコードそのもののスキャン、作成されたコンテナのスキャンの二段階で実施されます。二段階のスキャンにより、脆弱なコードが環境にデプロイされることを防ぎます。
* Sign : イメージの署名を行います。厳格な環境の場合、署名を受けたイメージ以外のデプロイを防ぐといったことを実施します。
* Store： Scan のフェーズでそのスキャニングの結果をデータベースに保管をして、高速な検索を実現します。
ここから、この機能について紐解いていこうと思います。なお、順番は Scan > Store > Sign の順に紹介します。

## Sign (イメージの署名)

Kubernetes では当たり前ですが、コンテナのフォーマットに従っていれば、なんでも起動ができてしまいます。ところが、セキュリティを考えた場合、  TAP の SupplyChain のプロセス以外で作成されるイメージを許可しないなどの制限を設ける必要性がでてきます。この「 TAP の SupplyChain のプロセス以外で作成されるイメージ」の判別の方法として使われるのが、イメージの署名、Sign 機能です。TAP では、 OSS である cosign を使った署名機能を使います。

cosign 自体は基本的な機能は以下の Gif で紹介されていますが、秘密鍵によるイメージの署名、そして公開鍵によるイメージの認証をおこなうというシンプルなものです。

{{< figure src="sign-gif.gif" width="100%">}}

今回のワークショップ環境は単一クラスタ構成ですが、TAP ではマルチクラスタ構成をとることができます。例えば、

* TAP をビルドクラスタ、Staging クラスタ、Production クラスタといった形でコンポーネントを分散させる
* ビルドクラスタの役目は TAP を使いセキュアなイメージをつくること
* ビルドクラスタに秘密鍵を登録して、イメージの署名
* 実際のワークロードは Staging クラスタ、Production クラスタにて稼働
* Staging クラスタ、Production クラスタには公開鍵を登録して、認証以外のイメージを許容しないようにする

{{< figure src="multi-cluster.png" width="100%">}}

## Store (SBoM の活用)

Scan 機能で脆弱性の検査が行われますが、その検索結果は SBoM (Software Bills of Materials)の形で metadata store とよばれるデータベースに保管されます。SBoM とは、パッケージの情報やバージョンをまとめたメタ情報です。通常の脆弱性検査に比べて、メタ情報のみの検索を行うため非常に高速に脆弱性検査ができることがメリットです。この「高速」は環境が小さい時こそ恩恵は感じにくいですが、環境が大きくなっていくにつれて、脆弱検査のスピードが重要になっていきます。

{{< figure src="sbom.png" width="100%">}}

TAP では専用の CLI である insight コマンドもしくは API を直接使い metadata store に保管された SBoM の検索が行えます。ここでの SBoM は先ほど紹介された Grype によって生成されています。以下 TAP によって作成されたイメージの検索結果ですが、そのイメージに含まれるパッケージ、バージョン、それにともなった脆弱性の照合までを瞬時で結果として得られます。metadata store を「脆弱性が報告された時に該当するイメージを瞬時にリストをしていく」などの機能ができるようになり、すばやく対応につなげていくことができます。

参考までに、先ほどデプロイしたtanzu-java-web-app のSBoM を確認してみます。これらのコマンドはワークショップ環境では利用することができませんので、下記出力は参考としてご確認ください。

今回ビルドしたコンテナイメージのダイジェストをもとに、tanzu insight コマンドでSBoM を表示させてみます。

```
[root@localhost tap2]# tanzu insight image get --digest sha256:6cd706255927f6e27d73df4a857c2e499a76725511889db9071d38aa14a5663d
ID:             8
Registry:       $REPO-URL
Image Name:     tap-workshop/supply-chain/tanzu-java-web-app-tap-workshop-01
Digest:         sha256:6cd706255927f6e27d73df4a857c2e499a76725511889db9071d38aa14a5663d
Packages:
        1. helper@3.4.0
        2. HdrHistogram@2.1.12
        3. LatencyUtils@2.0.3
        4. jackson-annotations@2.13.4
        5. jackson-core@2.13.4
        6. jackson-databind@2.13.4.2
        7. jackson-datatype-jdk8@2.13.4
        8. jackson-datatype-jsr310@2.13.4
        9. jackson-module-parameter-names@2.13.4
        10. jakarta.annotation-api@1.3.5
        11. jul-to-slf4j@1.7.36
        12. log4j-api@2.17.2
        13. log4j-to-slf4j@2.17.2
        14. logback-classic@1.2.11
        15. micrometer-core@1.9.5
        16. tomcat-embed-el@9.0.68
        17. tomcat-embed-websocket@9.0.68
        18. Spring Cloud Bindings@1.10.0
        19. adduser@3.116ubuntu1
        20. helper@9.8.0
        21. BellSoft Liberica JRE@11.0.16
        22. apt@1.6.14
        23. base-files@10.1ubuntu2.11
        24. base-passwd@3.5.44
        25. bash@4.4.18-2ubuntu1.3
        CVEs:
                1. CVE-2022-3715 (Medium)
        26. bsdutils@1:2.31.1-0.4ubuntu3.7
        27. bzip2@1.0.6-8.1ubuntu0.2
        28. ca-certificates@20211016~18.04.1
        29. coreutils@8.28-1ubuntu1
        CVEs:
                1. CVE-2016-2781 (Low)
        30. dash@0.5.8-2.10
        31. debconf@1.5.66ubuntu1
        32. debianutils@4.8.4
        33. diffutils@1:3.6-1
        34. dpkg@1.19.0.5ubuntu2.4
        35. e2fsprogs@1.44.1-1ubuntu1.4
        36. fdisk@2.31.1-0.4ubuntu3.7
        37. findutils@4.6.0+git+20170828-2
        38. gcc-8-base@8.4.0-1ubuntu1~18.04
        CVEs:
                1. CVE-2020-13844 (Medium)
        39. gpgv@2.2.4-1ubuntu1.6
        CVEs:
                1. CVE-2022-3219 (Low)
        40. grep@3.1-2build1
        41. gzip@1.6-5ubuntu1.2
        42. hostname@3.20
        43. init-system-helpers@1.51
        44. libacl1@2.2.52-3build1
        45. libapt-pkg5.0@1.6.14
        46. libattr1@1:2.4.47-2build1
        47. libaudit-common@1:2.8.2-1ubuntu1.1
        48. libaudit1@1:2.8.2-1ubuntu1.1
        49. libblkid1@2.31.1-0.4ubuntu3.7
        50. libbz2-1.0@1.0.6-8.1ubuntu0.2
        51. libc-bin@2.27-3ubuntu1.6
        CVEs:
                1. CVE-2009-5155 (Low)
                2. CVE-2015-8985 (Low)
                3. CVE-2016-20013 (Low)
        52. libc6@2.27-3ubuntu1.6
        CVEs:
                1. CVE-2009-5155 (Low)
                2. CVE-2015-8985 (Low)
                3. CVE-2016-20013 (Low)
        53. libcap-ng0@0.7.7-3.1
        54. libcom-err2@1.44.1-1ubuntu1.4
        55. libdb5.3@5.3.28-13.1ubuntu1.1
        56. libdebconfclient0@0.213ubuntu1
        57. libexpat1@2.2.5-3ubuntu0.7
        CVEs:
                1. CVE-2022-40674 (Medium)
                2. CVE-2022-43680 (Medium)
        58. libext2fs2@1.44.1-1ubuntu1.4
        59. libfdisk1@2.31.1-0.4ubuntu3.7
        60. libffi6@3.2.1-8
        61. libgcc1@1:8.4.0-1ubuntu1~18.04
        CVEs:
                1. CVE-2020-13844 (Medium)
        62. libgcrypt20@1.8.1-4ubuntu1.3
        63. libgmp10@2:6.1.2+dfsg-2
        CVEs:
                1. CVE-2021-43618 (Low)
        64. libgnutls30@3.5.18-1ubuntu1.6
        65. libgpg-error0@1.27-6
        66. libhogweed4@3.4.1-0ubuntu0.18.04.1
        67. libidn2-0@2.0.4-1.1ubuntu0.2
        68. liblz4-1@0.0~r131-2ubuntu3.1
        69. liblzma5@5.2.2-1.3ubuntu0.1
        70. libmount1@2.31.1-0.4ubuntu3.7
        71. logback-core@1.2.11
        72. slf4j-api@1.7.36
        73. snakeyaml@1.32
        CVEs:
                1. GHSA-w37g-rhq8-7m4j (Medium)
        74. spring-aop@5.3.23
        75. spring-beans@5.3.23
        76. spring-boot@2.7.5
        77. spring-boot-actuator@2.7.5
        78. spring-boot-actuator-autoconfigure@2.7.5
        79. spring-boot-autoconfigure@2.7.5
        80. spring-boot-devtools@2.7.5
        81. spring-boot-jarmode-layertools@2.7.5
        82. spring-context@5.3.23
        83. spring-core@5.3.23
        CVEs:
                1. CVE-2016-1000027 (Critical)
        84. spring-expression@5.3.23
        85. spring-jcl@5.3.23
        86. spring-web@5.3.23
        87. spring-webmvc@5.3.23
        88. tomcat-embed-core@9.0.68
        89. helper@5.19.0
        90. libncurses5@6.1-1ubuntu1.18.04
        CVEs:
                1. CVE-2019-17594 (Low)
                2. CVE-2019-17595 (Low)
                3. CVE-2021-39537 (Low)
                4. CVE-2022-29458 (Low)
        91. libncursesw5@6.1-1ubuntu1.18.04
        CVEs:
                1. CVE-2019-17594 (Low)
                2. CVE-2019-17595 (Low)
                3. CVE-2021-39537 (Low)
                4. CVE-2022-29458 (Low)
        92. libnettle6@3.4.1-0ubuntu0.18.04.1
        93. libp11-kit0@0.23.9-2ubuntu0.1
        94. libpam-modules@1.1.8-3.6ubuntu2.18.04.3
        95. libpam-modules-bin@1.1.8-3.6ubuntu2.18.04.3
        96. libpam-runtime@1.1.8-3.6ubuntu2.18.04.3
        97. libpam0g@1.1.8-3.6ubuntu2.18.04.3
        98. libpcre3@2:8.39-9ubuntu0.1
        CVEs:
                1. CVE-2017-11164 (Low)
        99. libprocps6@2:3.3.12-3ubuntu1.2
        100. libseccomp2@2.5.1-1ubuntu1~18.04.2
        101. libselinux1@2.7-2build2
        102. libsemanage-common@2.7-2build2
        103. libsemanage1@2.7-2build2
        104. libsepol1@2.7-1ubuntu0.1
        105. libsmartcols1@2.31.1-0.4ubuntu3.7
        106. libss2@1.44.1-1ubuntu1.4
        107. libssl1.1@1.1.1-1ubuntu2.1~18.04.20
        108. libstdc++6@8.4.0-1ubuntu1~18.04
        CVEs:
                1. CVE-2020-13844 (Medium)
        109. libsystemd0@237-3ubuntu10.53
        CVEs:
                1. CVE-2022-2526 (Medium)
                2. CVE-2022-3821 (Medium)
        110. util-linux@2.31.1-0.4ubuntu3.7
        111. zlib1g@1:1.2.11.dfsg-0ubuntu2.2
        CVEs:
                1. CVE-2022-42800 (Medium)
        112. github.com/BurntSushi/toml@v1.1.0
        113. github.com/BurntSushi/toml@v1.2.0
        114. github.com/Masterminds/semver/v3@v3.1.1
        115. github.com/apex/log@v1.9.0
        116. github.com/buildpacks/libcnb@v1.27.0
        117. github.com/buildpacks/lifecycle@v0.0.0-20221021172422-4659c442e411
        118. github.com/creack/pty@v1.1.18
        119. github.com/h2non/filetype@v1.1.3
        120. github.com/heroku/color@v0.0.6
        121. github.com/imdario/mergo@v0.3.13
        122. github.com/magiconair/properties@v1.8.6
        123. github.com/mattn/go-colorable@v0.1.13
        124. github.com/mattn/go-isatty@v0.0.16
        125. sysvinit-utils@2.88dsf-59.10ubuntu1
        126. tar@1.29b-2ubuntu0.3
        127. tzdata@2022c-0ubuntu0.18.04.0
        128. ubuntu-keyring@2018.09.18.1~18.04.2
        129. libtasn1-6@4.13-2
        130. libtinfo5@6.1-1ubuntu1.18.04
        CVEs:
                1. CVE-2019-17594 (Low)
                2. CVE-2019-17595 (Low)
                3. CVE-2021-39537 (Low)
                4. CVE-2022-29458 (Low)
        131. libudev1@237-3ubuntu10.53
        CVEs:
                1. CVE-2022-2526 (Medium)
                2. CVE-2022-3821 (Medium)
        132. libunistring2@0.9.9-0ubuntu2
        133. libuuid1@2.31.1-0.4ubuntu3.7
        134. libyaml-0-2@0.1.7-2ubuntu3
        135. libzstd1@1.3.3+dfsg-2ubuntu1.2
        136. locales@2.27-3ubuntu1.6
        CVEs:
                1. CVE-2009-5155 (Low)
                2. CVE-2015-8985 (Low)
                3. CVE-2016-20013 (Low)
        137. login@1:4.5-1ubuntu2.3
        CVEs:
                1. CVE-2013-4235 (Low)
        138. lsb-base@9.20170808ubuntu1
        139. mawk@1.3.3-17ubuntu3
        140. mount@2.31.1-0.4ubuntu3.7
        141. ncurses-base@6.1-1ubuntu1.18.04
        CVEs:
                1. CVE-2019-17594 (Low)
                2. CVE-2019-17595 (Low)
                3. CVE-2021-39537 (Low)
                4. CVE-2022-29458 (Low)
        142. ncurses-bin@6.1-1ubuntu1.18.04
        CVEs:
                1. CVE-2019-17594 (Low)
                2. CVE-2019-17595 (Low)
                3. CVE-2021-39537 (Low)
                4. CVE-2022-29458 (Low)
        143. netbase@5.4
        144. openssl@1.1.1-1ubuntu2.1~18.04.20
        145. passwd@1:4.5-1ubuntu2.3
        CVEs:
                1. CVE-2013-4235 (Low)
        146. perl-base@5.26.1-6ubuntu0.5
        CVEs:
                1. CVE-2020-16156 (Medium)
        147. procps@2:3.3.12-3ubuntu1.2
        148. sed@4.4-2
        149. sensible-utils@0.0.12
        150. github.com/mattn/go-shellwords@v1.0.12
        151. github.com/miekg/dns@v1.1.50
        152. github.com/mitchellh/hashstructure/v2@v2.0.2
        153. github.com/onsi/gomega@v1.20.2
        154. github.com/paketo-buildpacks/ca-certificates/v3@v0.0.0-20220901010128-3944830c68f6
        155. github.com/paketo-buildpacks/libjvm@v1.40.0
        156. github.com/paketo-buildpacks/libpak@v1.62.0
        157. github.com/paketo-buildpacks/libpak@v1.63.0
        158. github.com/paketo-buildpacks/spring-boot/v5@v0.0.0-20220921103437-77d00b359dfa
        159. github.com/pavlo-v-chernykh/keystore-go/v4@v4.4.0
        160. github.com/pelletier/go-toml@v1.9.5
        161. github.com/pkg/errors@v0.9.1
        162. github.com/xi2/xz@v0.0.0-20171230120015-48954b6210f8
        163. golang.org/x/net@v0.0.0-20220923203811-8be639271d50
        164. golang.org/x/sys@v0.0.0-20220829200755-d48e67d00261
        165. golang.org/x/sys@v0.0.0-20220915200043-7b5979e65e41
        166. golang.org/x/sys@v0.0.0-20220919091848-fb04ddd9f9c8
        167. golang.org/x/sys@v0.1.0
        168. jrt-fs@11.0.16.1
        169. spring-cloud-bindings@1.10.0
```

また、 特定の CVE によって影響を受ける依存関係を表示させることもできます。2021 年12 月に公開された脆弱性「Log4Shell」はまだ記憶に新しいところではありますが、あまりに色々なソフトウェアにあたり前に利用されていたため、その影響範囲を調べることが困難でした。SBoM はソフトウェアがどのような材料で作られているかを示す表のようなものですので、脆弱性が発生したときにその影響範囲を明確にし、より適切な対応を取ることができる指針として活用できます。

```
[root@localhost tap2]# tanzu insight vulnerabilities get --cveid CVE-2022-3821
1. CVE-2022-3821 (Medium)
Packages:
        1. libsystemd0@237-3ubuntu10.53
        2. libudev1@237-3ubuntu10.53
```