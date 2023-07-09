# サードパーティコードとコンポーネント

ツールチェーンのセットアップ後は、カーネル、ソフトウェアパッケージ、サードパーティライブラリを確実に更新して、公にされている既知の脆弱性から保護することが重要です。Rompager などのソフトウェアや Buildroot などの組込みビルドツールは、更新が必要かどうかを判断するために脆弱性データベースや ChangeLog をチェックすべきです。組込みシステムを更新するとそれらのシステムの動作に問題が生じる可能性があるため、このプロセスはリリースビルドの前に開発者や QA チームによりテストされるべきであることに注意することが重要です。

組込みプロジェクトはそのファームウェアイメージに含まれるサードパーティソフトウェアとオープンソースソフトウェアの「部品表 (Bill of Materials)」を保守する必要があります。この部品表をチェックして、含まれているサードパーティソフトウェアにパッチが当てられていない脆弱性がないことも確認する必要があります。National Vulnerability Database や Open Hub を通じて、最新の脆弱性情報が見つかる可能性があります。

サードパーティソフトウェアのカタログ化と監査にはいくつかのソリューションがあります。以下のような多くのソリューションがビルド環境に組み込まれています。

* C / C++
  * `Makefile`
* Go
  * 公式の `dep` [ツール](https://github.com/golang/dep) を使用します。
* Node
  * `npm list`
* Python
  * `pip freeze`
* Ruby
  * `gem dependency`
* Lua
  * `rockspec file` を参照します。
* Java
  * `mvn dependency:tree`
  * `gradle app:dependencies`
* Yocto
  * `buildhistory`
* Buildroot (フリー)
  * `make legal-info`
* パッケージマネージャ (フリー)
*
  * `dpkg --list`
  * `rpm -qa`
  * `yum list`
  * `apt list --installed`
* Javascript プロジェクトのための RetireJS (フリー)

**BOM の例を以下に示します:**

| **Component** | Version | Vulnerabilities - CVEs | Notes       |
| ------------- | ------- | ---------------------- | ----------- |
| jQuery        | 1.4.4   | CVE-2011-4969          |             |
| libxml2       | 2.9.4   | CVE-2016-5131          | To be fixed |

ソフトウェア BOM にはコンポーネントの機能や特定のバージョンを使用することの正当性に関するライセンスおよび文脈情報も含んでいます。

**JavaScript プロジェクトディレクトリ内の Retirejs 例:**

```bash
$ retire .
Loading from cache: https://raw.githubusercontent.com/RetireJS/retire.js/master/repository/jsrepository.json
Loading from cache: https://raw.githubusercontent.com/RetireJS/retire.js/master/repository/npmrepository.json
/js/jquery-1.4.4.min.js
 ↳ jquery 1.4.4.min has known vulnerabilities: severity: medium; CVE: CVE-2011-4969; http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-4969 http://research.insecurelabs.org/jquery/test/ severity: medium; bug: 11290, summary: Selector interpreted as HTML; http://bugs.jquery.com/ticket/11290 http://research.insecurelabs.org/jquery/test/ severity: medium; issue: 2432, summary: 3rd party CORS request may execute; https://github.com/jquery/jquery/issues/2432 http://blog.jquery.com/2016/01/08/jquery-2-2-and-1-12-released/
/js/jquery-1.4.4.min.js
 ↳ jquery 1.4.4.min has known vulnerabilities: severity: medium; CVE: CVE-2011-4969; http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-4969 http://research.insecurelabs.org/jquery/test/ severity: medium; bug: 11290, summary: Selector interpreted as HTML; http://bugs.jquery.com/ticket/11290 http://research.insecurelabs.org/jquery/test/ severity: medium; issue: 2432, summary: 3rd party CORS request may execute; https://github.com/jquery/jquery/issues/2432 http://blog.jquery.com/2016/01/08/jquery-2-2-and-1-12-released/
/javascript/vendor/jquery-1.9.1.min.js
 ↳ jquery 1.9.1.min has known vulnerabilities: severity: medium; issue: 2432, summary: 3rd party CORS request may execute; https://github.com/jquery/jquery/issues/2432 http://blog.jquery.com/2016/01/08/jquery-2-2-and-1-12-released/
/javascript/vendor/jquery-migrate-1.1.1.min.js
 ↳ jquery-migrate 1.1.1.min has known vulnerabilities: severity: medium; release: jQuery Migrate 1.2.0 Released, summary: cross-site-scripting; http://blog.jquery.com/2013/05/01/jquery-migrate-1-2-0-released/ severity: medium; bug: 11290, summary: Selector interpreted as HTML; http://bugs.jquery.com/ticket/11290 http://research.insecurelabs.org/jquery/test/
/javascript/vendor/moment.min.js
 ↳ moment.js 2.10.6 has known vulnerabilities: severity: low; summary: reDOS - regular expression denial of service; https://github.com/moment/moment/issues/2936
```



yocto ビルドから installed-packages.txt を見つけます。詳細については次を参照します: [http://www.yoctoproject.org/docs/current/ref-manual/ref-manual.html\#understanding-what-the-build-history-contains](http://www.yoctoproject.org/docs/current/ref-manual/ref-manual.html#understanding-what-the-build-history-contains)

**Yocto 2.2 Morty 以降、ビルドインの** `cve-check` [**BitBake クラス**](https://git.yoctoproject.org/cgit/cgit.cgi/poky/tree/meta/classes/cve-check.bbclass) **がビルド時に公開 CVE に対してレシピを自動チェックするために追加されました。詳細については次の Yocto ページを参照してください。** [**https://docs.yoctoproject.org/dev/dev-manual/vulnerabilities.html**](https://docs.yoctoproject.org/dev/dev-manual/vulnerabilities.html)



**検討事項 (免責: 以下のリストは完全なものではありません):**

* JavaScript ライブラリに [retire.js](https://github.com/RetireJS/retire.js) を使用します。
  * NodeJS パッケージに `npm audit` を利用します。
* [OWASP DependencyCheck](https://github.com/jeremylong/DependencyCheck) を使用して、アプリケーションの [依存関係およびファイルの種類](https://jeremylong.github.io/DependencyCheck/analyzers/index.html) における一般に公開された脆弱性を検出します。
* Python 関連パッケージの既知の脆弱性をスキャンするために [`safety check`](https://github.com/pyupio/safety) を使用します。
* ウェブアプリケーションテストに [OWASP ZAP](https://github.com/zaproxy/zaproxy/wiki/Downloads) を使用します。
* 基本的なカーネル監査と提案を強化するために [Lynis](https://raw.githubusercontent.com/CISOfy/lynis/master/lynis) などのツールを利用します。
  * `wget --no-check-certificate  https://github.com/CISOfy/lynis/archive/master.zip && unzip master.zip && cd lynis-master/ && bash lynis audit system`
  * `/var/log/lynis.log` のレポートをレビューします。
  * **注意**: Linux カーネルを使用していない場合、Lynis はカーネルチェックを行いません。次のメッセージがログに記録されます。 "Skipped test KRNL-5695 (Determine Linux kernel version and release number) Reason to skip: Incorrect guest OS (Linux only)"
  * ストレージが制限されている (すなわち php などの不要なプラグインを削除している) 場合、Lynis はそれに応じて修正すべきです。
* [LibScanner](https://github.com/scriptingxss/LibScanner) などのフリーのライブラリスキャナを利用して、プロジェクトの依存関係を検索し、NVD と相互参照して、yocto ビルド環境の既知の CVE を探します。
  * このツールはチームが継続的統合テストのためにそのような機能を利用できるようにする XML を出力します。
* ツールチェーン内のその他のライブラリにはパッケージマネージャ (opkg, ipkg など) やカスタムアップデートメカニズムを利用します。
* ツールチェーン、ソフトウェアパッケージ、ライブラリの更新履歴を確認して、アップデートが必要かどうかを判断します。
* Yocto や Buildroot などの組込みビルドシステムの実装が、含まれているすべてのパッケージのアップデートを可能にするようにセットアップされていることを確認します。

## その他の参考情報 <a href="#additional-references" id="additional-references"></a>

* [https://www.kb.cert.org/vuls/id/922681](https://www.kb.cert.org/vuls/id/922681)
* [https://www.kb.cert.org/vuls/id/561444](https://www.kb.cert.org/vuls/id/561444)
* [https://buildroot.org/downloads/manual/manual.html#faq-no-binary-packages](https://buildroot.org/downloads/manual/manual.html#faq-no-binary-packages)
* [https://wiki.yoctoproject.org/wiki/Security](https://wiki.yoctoproject.org/wiki/Security)
* [https://nvd.nist.gov/](https://nvd.nist.gov/)
* [https://www.openhub.net/](https://www.openhub.net/)
* [Improving Your Embedded Linux Security Posture with Yocto](https://legacy.gitbook.com/book/scriptingxss/embedded-appsec-best-practices/edit)
