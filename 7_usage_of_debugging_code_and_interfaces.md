# デバッグコードとインタフェースの使用

すべての市場セグメントにファームウェアをリリースする前に、不必要な出荷前ビルドコード、デッドコード、未使用コードがすべて削除されていることを確認することが重要です。これには設計製造業者 (ODM) やサードパーティ業者などの関係者により残された可能性のある潜在的なバックドアコードやルート特権アカウントが含まれますが、これに限定されません。通常、これは相手先ブランド名製造 (OEM) のスコープに該当し、契約書またはバイナリのリバースエンジニアリングを介して実行します。また、ODM はマスターサービス契約書 (MSA) に、バックドアコードが含まれていないこと、ソフトウェアセキュリティ脆弱性についてすべてのコードがレビューされていることを保証するための署名をする必要があります。すべてのサードパーティ開発者は市場に大量に導入されているデバイスに説明責任があります。

**検討事項 (免責事項: 以下のリストは網羅していません):**

* デバッグ、デプロイメント検証、カスタマーサポートの目的で使用されるバックドアアカウントを削除します。
* 開発、診断、デバッグ機能がリリースビルドに含まれていないことを確認します。
* コードのクリーンアップセッションを実行して、デッドコードや未使用コードがリポジトリ間で確実に削除されるようにします。
* 市場にデプロイメントする前にサードパーティライブラリおよびバイナリイメージがスタッフによりバックドアについてレビューされていることを確認します。
  * Binwalk, Firmadyne, IDA pro, radare2, Firmware Mod Toolkit (FMK), Firmware Analysis Comparison Toolkit (FACT), およびその他の参考情報 (#4) に記載されているその他のさまざまなツールをファームウェア解析に利用すべきです。

## その他の参考情報 <a id="additional-references"></a>

* [https://www.owasp.org/index.php/Leftover_Debug_Code](https://www.owasp.org/index.php/Leftover_Debug_Code)
* [https://cwe.mitre.org/data/definitions/489.html](https://cwe.mitre.org/data/definitions/489.html)
* [http://www.kb.cert.org/vuls/id/419568](http://www.kb.cert.org/vuls/id/419568)
* [Firmware and binary analysis tool index](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/#firmware-and-binary-analysis-tool-index)
