---
title: "次世代型DB TiDB採用の前に知っておきたい基礎知識"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tidb", "database", "mysql"]
published: false
publication_name: "gemcook"
---

こんにちは。
最近 TiDB Serverless の実証を行う機会があったので、その前に調べた TiDB の基礎知識をまとめてみました。私は普段 DynamoDB を使うことが多いのですが、設計面の難しさや柔軟な検索機能の弱さなどに辛さを感じることがそれなりにあります。

そのためサーバレスで MySQL 互換データベースの TiDB Serverless には非常に期待&興味をもってます！

リポジトリ、公式サイトはこちら

https://github.com/pingcap/tidb

https://pingcap.co.jp/tidb/

## TiDB Serverless のいいところ

TiDB や NewSQL の説明の前に、なぜ TiDB Serverless を使おうと思ったのか・使うメリットはなんなのかを先に述べておきます！

- MySQL 互換
  - NoSQL DB に比べて柔軟なクエリが可能。検索の自由度が高い
  - ACID
- サーバレスであること
  - 高速なオートスケーリング
  - 従量課金制
    - 無料枠があるのも嬉しい（PoC やローカルでの実験にも使いやすい）
- VPC に対応

後は筆者が単純に MySQL に慣れているので、そこも大きな理由ですね。この用途では AWS の Aurora Serverless も候補に上がるのですが、比較的コストが高いと感じているので TiDB Serverless にはそのあたりも期待しています。

## TiDB とは

TiDB は MySQL と互換性（完全ではない）を持つ分散型データベース、つまり NewSQL です。互換性がない機能は[こちら](https://docs.pingcap.com/ja/tidbcloud/mysql-compatibility#unsupported-features)。
少し触ってみた感触だと MySQL に比べてほぼ不自由無い印象です。既存の MySQL から移行する場合は検証をしっかりするなど注意が必要なぐらいでしょうか。

採用事例もかなり増えており、目にする機会も大分増えてきました。

## NewSQL とは

先ほど TiDB とは NewSQL と書きましたが、NewSQL ってそもそも何やねん？？という方に向けて簡単な説明をします。

NewSQL は簡単に言うと RDBMS と NoSQL のいいとこ取りをしたデータベースのことです。

RDBMS のようにトランザクション処理ができて、NoSQL のようにスケーラビリティが高いのが特徴です。NoSQL もサービスによってはトランザクション処理をサポートしているものもありますが、NewSQL は検索の自由度が高かったり、より RDBMS の強みをもっています。

これはまさしく次世代の夢の DB システム！

### NewSQL の歴史

NewSQL 自体は Google が 2012 年に Spanner という分散型 DB システムを発表したことから始まり、その後 2017 年に Google Cloud Spanner として一般提供されました。TiDB は 2017 年に OSS として GA。その後 2020 年に TiDB Cloud がリリース。

採用例は最近増えてきましたが、個人的な感覚として以前から存在する技術だったことに驚いています（ここ 2~3 年ぐらいの技術のイメージだったので）

### NewSQL の代表例

- [TiDB](https://github.com/pingcap/tidb)
  - MySQL 互換
  - 元は OSS
    - セルフホスト可能
  - TiDB Cloud&TiDB Serverless でマネージドサービスとしても提供
- [Google Spanner](https://cloud.google.com/spanner?hl=ja)
  - PostgreSQL 互換
  - マネージドサービス
- [CockroachDB](https://www.cockroachlabs.com/)
  - PostgreSQL 互換
  - OSS
  - マネージドサービス

代表的な NewSQL としてはこれらがあります。
OSS として提供しながら、クラウドでのマネージドサービスとしても提供しているものが多いですね。

## TiDB の構成

TiDB の構成自体は公式ドキュメントや PingCAP（TiDB 開発元）の方が記事にしてくれているので、ここでは割愛。
以下の記事がわかりやすかったですね。

https://docs.pingcap.com/ja/tidb/stable/tidb-architecture

https://zenn.dev/koiping/articles/f4b6579f3e6a68

https://zenn.dev/koiping/articles/fc54076d2e6845

## TiDB の使い方

TiDB にはセルフホストが可能な OSS 版、VPC 内に構築する TiDB Cloud、サーバレスの TiDB Serverless があります。
私は PoC のために触ってみたかったので、TiDB Serverless で試しました。セルフホストなら Kubernetes 環境を用意して動かすのが一般的でしょう。
コンソールから環境を簡単に作れるのもいいところ（そのうち cdktf に記述を移行したい）

### ローカル環境で TiDB を動かす

ローカルで TiDB を動かす場合は公式ドキュメントに従って動かすのが一番楽です。
インストールすれば数分で試せるので、この手軽さはかなりありがたい！

https://docs.pingcap.com/tidb/stable/quick-start-with-tidb

## おわりに

今までの RDBMS ではスケーラビリティの問題に悩まされ、NoSQL では設計面と複雑な構成を組むことの難しさに悩まされていました。それらを解決するための NewSQL, TiDB には非常に期待が持てます。これでコスト面の折り合いが付けば言う事なしですね。
現在進行系で実証を行っているので、コスト感も含めてその結果もまた記事にしていきたいと思います！

良い DB ライフを！
