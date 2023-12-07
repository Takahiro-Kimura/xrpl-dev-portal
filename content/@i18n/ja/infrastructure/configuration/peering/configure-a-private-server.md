---
html: configure-a-private-server.html
parent: configure-peering.html
blurb: サーバーが特定の信頼できるピアのみに接続するように設定します。
labels:
  - コアサーバー
  - セキュリティ
---
# プライベートサーバーの設定

[プライベートサーバー](peer-protocol.html#プライベートピア)は、オープンなピアツーピアネットワーク内の検出されたピアに直接接続するのではなく、特定の信頼できるピアのみを通じてネットワークに接続する`rippled`サーバーです。この種の構成は、[バリデータ](run-rippled-as-a-validator.html)に一般的に推奨される任意の対策ですが、その他の特定の目的でも役立ちます。

## 前提条件

プライベートサーバーを使用するには、次の前提条件を満たしている必要があります。

- [`rippled`をインストール](install-rippled.html)して最新バージョンにアップデートし、まだ実行していない状態である必要があります。
- 自社で運用している**プロキシ**を通じて接続するか、**公開ハブ**を通じて接続するかを決める必要があります。これらの選択肢の違いについては、[ピアリング構成のメリットとデメリット](peer-protocol.html#ピア接続設定のメリットとデメリット)を参照してください。
  - プロキシを使用している場合、`rippled`がインストールされていてプロキシとして使用し実行される別のマシンが必要です。これらのサーバーは、外部のネットワークとプライベートサーバーに接続できる必要があります。
  - どちらの構成でも、接続先のピアのIPアドレスとポートを把握しておく必要があります。

## 手順

特定のサーバーをプライベートピアとして設定するには、次の手順を実行します。

1. `rippled`の構成ファイルを編集します。

        vim /etc/opt/ripple/rippled.cfg

   {% include '_snippets/conf-file-location.ja.md' %}<!--_ -->

2. プライベートピアリングを有効にします。

   構成ファイルに以下のスタンザを追加するか、コメントを解除します。

        [peer_private]
        1

3. 固定数のピアを追加します。

   構成ファイルに`[ips_fixed]`スタンザを追加するか、コメントを解除します。このスタンザの各行は、接続先のピアのホスト名またはIPアドレス、1個の空白文字、このピアがピアプロトコル接続を受け付けるポートの順に記載されている必要があります。

   例えば、**公開ハブ**を使用して接続する場合は、以下のスタンザを使用できます。

        [ips_fixed]
        r.ripple.com 51235
        zaphod.alloy.ee 51235

   サーバーが**プロキシ**を使用して接続している場合は、IPアドレスとポートが、プロキシとして使用している`rippled`サーバーの構成と一致している必要があります。これらの各サーバーについては、ポート番号が、サーバーの構成ファイルに記載されている`protocol = peer`ポート（通常は51235）と一致している必要があります。例えば、構成は次のようになります。

          [ips_fixed]
          192.168.0.1 51235
          192.168.0.2 51235

4. プロキシを使用している場合、プロキシをプライベートピアと互いを含めてクラスター化します。

   公開ハブを使用している場合は、このステップをスキップします。

   プロキシを使用している場合、プライベートピアを含む[クラスターとしてプロキシを構成](cluster-rippled-servers.html)します。クラスターの各メンバーは、クラスターの_他の_各メンバーをリストにした`[ips_fixed]`スタンザを持っている必要があります。ただし、`[peer_private]`スタンザを持つのは**プライベートサーバーのみ**とします。

   各プロキシで`rippled`を再起動します。各プロキシサーバーで、次のようにします。

        sudo service systemctl restart rippled

5. プライベートサーバーで`rippled`を起動します。

        sudo service systemctl start rippled

6. [peersメソッド][]を使用して、プライベートサーバーが自身のピアに _のみ_ 接続していることを確認します。

   応答の`peers`配列に、構成済みのピアのいずれでもない`address`を持つオブジェクトが含まれていてはなりません。含まれている場合は、構成ファイルを再度確認して、プライベートサーバーを再起動します。


## 次のステップ

追加の予防対策として、自身のピアでないサーバーからプライベートサーバーへの着信接続をブロックするようにファイアウォールを設定する必要があります。プロキシサーバーを実行している場合は、ファイヤーウォールを通じてプロキシに[ピアポートを転送](forward-ports-for-peering.html)するようにします。ただし、プライベートピアで**ない**ものに転送します。この設定方法の具体的な手順は、使用するファイアウォールによって異なります。

ファイアウォールがポート80で発信HTTP接続を**ブロックしない**ことを確認します。デフォルトの設定では、このポートを使用して**vl.ripple.com**から最新の推奨バリデータリストをダウンロードします。バリデータリストがないと、サーバーはどのバリデータを信頼してよいかわからず、ネットワークが、いつコンセンサスに至ったかを認識できません。

## 関連項目

- **コンセプト:**
  - [ピアプロトコル](peer-protocol.html)
  - [コンセンサス](consensus.html)
  - [並列ネットワーク](parallel-networks.html)
- **チュートリアル:**
  - [ピアクローラーの設定](configure-the-peer-crawler.html)
- **リファレンス:**
  - [peersメソッド][]
  - [connectメソッド][]
  - [fetch_infoメソッド][]
  - [ピアクローラー](peer-crawler.html)


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}