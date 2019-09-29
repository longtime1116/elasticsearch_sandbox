# 1章
* Elasticsearch で作成したインデックスるが利用できるのは、1つ先のメジャーリリースまで
  * 5.x 系で作ったインデックスは 6.x 系でも使えるが、7.x 系では使えない

* 特徴4つ(1-2-2)
  * 分散配置による高速化と高可用性の実現
    * シャードの数を増やす => 分散配置による高速化(デフォルトは5)
    * レプリカを増やす => データロスト防止(デフォルトは1)
    * [わかりやすい資料を見つけた](https://www.slideshare.net/snuffkin/elasticsearch-as-a-distributed-system)
  * シンプルな REST API によるアクセス
  * JSONフォーマットに対応した柔軟性の高いドキュメント志向データベース
    * 事前にデータ型をスキーマとして登録しなくても、Elasticsearchは自動的にデータの型を推論してくれる(明示的に指定することも可能)
    * Elasticsearch の文脈において レコードのことをドキュメントとよぶ
    * これにより、スキーマ定義の修正対応が必要！みたいなことがなく、容易にフィールド追加もできる、らしい。
  * ログ収集・可視化などの多様な関連ソフトウェアとの連携
    * Elastic Stack と呼ばれるツール群
    * 可視化 => Kibana
    * ログ収集 => Logstash
    * サーバからのメトリクスデータ収集 => Beats
    * 監視・モニタリング、機械学習による異常検知などの有償アドオン => X-Pack

* Apatch Solr との違い(1-2-3)
  * シャード数、Elasticsearch ではインデックス作成後は変更不可！！
  * Elasticsearch ではシャードの自動リバランスをしてくれる
  * 調停方式、Elasticsearch は内臓の ZEN Discovery、Solr は Zookeeper
    * 調停方式って何？

* Elastic Stack について(1-3)
  * Elasticsearch に加えて Kibana, Logstash, Beats などの周辺ソフトウェアも含めた総称
  * Elasticsearch のバージョンは 2.4 => 5.0 とジャンプしている(2016年)が、これはその Elastic Stack でのバージョンの統一をするために行われたもの(このタイミングで Beats が加わった)

  * Kibana
    * Elasticsearch に格納されたデータを分析する際に、直接クエリを投げてもよいが、Kibana の UI を使うことでいい感じに確認することができる。
    * Tableau みたいなものか。
  * Logstash
    * input -> filter -> output
    * syslog や twitter とかからデータを取ってきて、Elasticsearch に出力する、みたいなことができる
  * Beats
    * メトリクスデータを収集・転送するための軽量データシッパー。
    * インストールされたサーバにできるだけ負荷を与えないよう、収集したデータを Elasticsearch や Logstash に転送してくれる



# 2章
* 論理的な概念(2-1-1)
  * ドキュメント
    * RDSでいうテーブルに格納される1行のレコード。JSONオブジェクト。
    * 明示的に指定しなければ一意な id は自動で採番される
  * フィールド
    * key-value の組
    * Elasticsearch における転置インデックスは、フィールドごとに作成・管理されている
    * [公式のデータ型説明](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)
    * 具体例
      * 文字列
        * text
          * 文字列からなる文章を格納
          * 格納時にアナライザによって文章を構成する各単語に分割されて、分割された単語ごとに転置インデックスが構成される
          * 部分一致用
        * keyword
          * 完全一致用の文字列格納
      * 数値
        * long、short、integer、float など
      * 日付(date)
        * フォーマットを指定して登録する
        * フォーマット例
          * "2019-09-29"
          * "2019-09-29T20:00:00"
          * "1569754800" (UNIXエポックからのミリ秒数)
      * 真偽値(boolean)
        * "true" or "false"
      * オブジェクト(object)
        * JSONオブジェクトをネストにするとき、Elasticsearch では object 型を用いるとデータ型として扱える
        * `{"book": {"title": "Elasticsearch guide", "price": "500"}}`
      * 配列(array)
        * 配列の各要素は同じデータ型をとる
        * `{"ids": [1,2,3]}`
        * `{"vms": [{"host": "web01", "flavor": "small"}, {"host": "db01", "flavor": "large"}]}`
    * マルチフィールド型という概念
      * 一つのフィールドに同時に複数のデータ型を定義できる
      * 完全一致検索と部分一致検索の両方を望むのであれば、text と keyword 両方のデータ型を定義する
      * Elasticsearch5.0からは、指定しなければ文字列はそのようにマルチフィールドとして定義されるようになっている
      * その必要がないのであれば指定すると良い


# 疑問
* シャード数変えられないので、12にした理由
* Kibana 以外の Elastic Stack どうしているのだろうか？Beats必須では？
