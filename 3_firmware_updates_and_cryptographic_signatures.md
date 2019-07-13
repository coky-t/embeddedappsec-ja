# ファームウェア更新と暗号化署名

堅牢な更新メカニズムが、ダウンロード時および適用可能な場合はサードパーティソフトウェアに関する機能を更新するために、暗号署名されたファームウェアイメージを使用するようにします。暗号署名を使用すると、開発者が作成および署名してからファイルが改変やあるいは改竄されていないかを検証できます。署名および検証プロセスは公開鍵暗号を使用し、最初に秘密鍵にアクセスすることなくデジタル署名 (PGP 署名など) を偽造することは困難です。秘密鍵が危殆化された場合、ソフトウェアの開発者は危殆化された鍵を無効にしなければならず、新しい鍵で以前のすべてのファームウェアリリースに再署名する必要があります。

**カーネルイメージの署名を検証する例:**

カーネルイメージをダウンロードします

```bash
wget [https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.6.6.tar.xz](https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.6.6.tar.xz)

wget [https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.6.6.tar.sign](https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.6.6.tar.sign)
```

**署名を検証するために PGP 鍵サーバーから公開鍵をダウンロードします。**

```bash
# gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 38DBBDC86092693E
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 38DBBDC86092693E: public key "Greg Kroah-Hartman (Linux kernel stable release signing key) <greg@kroah.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1
```

**.tar ファームウェアイメージを展開して署名を検証します:**

```bash
# xz -cd linux-4.6.6.tar.xz | gpg2 --verify linux-4.6.6.tar.sign -
gpg: Signature made Wed 10 Aug 2016 06:55:15 AM EDT
gpg:                using RSA key 38DBBDC86092693E
gpg: Good signature from "Greg Kroah-Hartman (Linux kernel stable release signing key) <greg@kroah.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 647F 2865 4894 E3BD 4571  99BE 38DB BDC8 6092 693E
```

WARNING: This key is not certified with a trusted signature! (警告: この鍵は信頼できる署名で認証されていません) に注意します。ここでアーカイブに署名するために使用された鍵が本当に所有者 (この例では Greg Kroah-Hartman) に属していることを検証する必要があります。これを行うにはいくつかの方法があります。

1. Kernel.org の web of trust を使用します。これはあなたがまずあなたの地域で kernel.org のメンバーを見つけて彼らの鍵に署名することを要求するでしょう。実生活で PGP 鍵の実際の所有者に合うことを除けば、これは PGP 鍵署名の有効性を検証するための最善の選択肢です。
2. `gpg --list-sigs` を使用して開発者の鍵にある署名のリストをレビューします。鍵に署名したできるだけ多くの人に、できれば異なる組織 (もしくは少なくとも異なるドメイン) に電子メールを送ります。問題の鍵に署名したことを確認するように依頼します。あなたが (もし何かを受け取るのであれば) このようなやり方で受け取るレスポンスにせいぜい marginal trust を付け加えるべきです。
3. 次のサイト pgp.cs.uu.nl を使用して、Linux Torvalds の鍵から tarball の署名に使用されている鍵までの信頼パスを確認します。Linus の鍵を "from" フィールドに入れ、上記の出力で得た鍵を "to" フィールドに入れます。通常、Linus または Linus の直接の署名を持つ人々だけがカーネルのリリースを担当します。

"BAD signature" の場合
`gpg --verify` で "BAD signature" が出力される場合は、まず以下をチェックしてください。

1. 圧縮された (.tar.xz) バージョンではなく、.tar バージョンのアーカイブに対して署名を検証していることを確認します。
2. ダウンロードしたファイルが正しいこと、切り捨てられていないこと、破損していないことを確認します。

**上記 #1 のデモ、署名の正しくない検証例**:

```bash
# gpg --verify linux-4.6.6.tar.sign linux-4.6.6.tar.xz 
gpg: Signature made Wed 10 Aug 2016 06:55:15 AM EDT
gpg:                using RSA key 38DBBDC86092693E
gpg: BAD signature from "Greg Kroah-Hartman (Linux kernel stable release signing key) <greg@kroah.com>" [unknown]
```

**署名の正しい検証例**:

```bash
# gpg --verify linux-4.6.6.tar.sign linux-4.6.6.tar
gpg: Signature made Wed 10 Aug 2016 06:55:15 AM EDT
gpg:                using RSA key 38DBBDC86092693E
gpg: Good signature from "Greg Kroah-Hartman (Linux kernel stable release signing key) <greg@kroah.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 647F 2865 4894 E3BD 4571  99BE 38DB BDC8 6092 693E
```

**検討事項:**

* 堅牢な更新メカニズムが更新機能のために暗号的に署名されたファームウェアイメージを利用することを確認します。
  * GPG \([https://github.com/romanz/trezor-agent/blob/master/README-GPG.md](https://github.com/romanz/trezor-agent/blob/master/README-GPG.md)\)
* 更新が可能な限り最新のセキュアな TLS バージョンを介してダウンロードされることを確認します。 (執筆時点で、これは TLS1.3 です) 
  * 更新が更新サーバーの公開鍵と証明書チェーンを妥当性確認することを確認します。
* 事前定義されたスケジュールで自動ファームウェアアップデートを利用する機能を含んでいること。
  * 非常に脆弱なユースケースで更新を強制します。
  * 医療機器など特定の機器では、強制アップデートにより問題が発生しないように、スケジュールされたプッシュアップデートを考慮する必要があります。
* ファームウェアバージョンが明確に表示されていることを確認します。
* ファームウェアアップデートにはセキュリティ関連の脆弱性を含む変更履歴が含まれていることを確認します。
* デバイスを脆弱なバージョンに戻すことができないように、ダウングレード防止機能 (アンチロールバック) メカニズムが採用されていることを確認します。
* Extended Verification Module (EVM) がファイル属性 (拡張属性を含む) をチェックする中で stored/calculate ハッシュ (label と呼ばれる) に対して妥当性確認することにより、ファイルが変更されていないことをカーネルに確認させる [Integrity Measurement Architecture (IMA)](https://sourceforge.net/p/linux-ima/wiki/Home/) の実装を検討します。
  * 二種類のラベルが利用できます。
    * 不変 (immutable) および署名済み (signed)
    * シンプル (Simple)
* [読み取り専用のルートファイルシステム](http://docs.automotivelinux.org/docs/architecture/en/dev/reference/security/05-security-concepts.html#read-only-root-file-system) の実装を検討します。ローカルパーシステンスを必要とするディレクトリ用に作成できるオーバーレイを使用します。
  * 永続的なストレージロケーションにデータを書き込むことができる承認プロセスに対して適切なコントロールと監視を適用します。
* 証明書失効サーバーの照会には信頼できるクロックソースが利用可能であることを確認します。



## その他の参考情報 <a id="additional-references"></a>

* [https://www.kernel.org/signature.html](https://www.kernel.org/signature.html)
* [https://www.owasp.org/index.php/Key\_Management\_Cheat\_Sheet](https://www.owasp.org/index.php/Key_Management_Cheat_Sheet)
* [http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt1r4.pdf](http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt1r4.pdf)
* [https://www.math.utah.edu/~beebe/PGP-notes.html](https://www.math.utah.edu/~beebe/PGP-notes.html)
* [CWE-321: Use of Hard-coded Cryptographic Key - CVE-2013-6952](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-6952)
  * [http://www.ioactive.com/pdfs/IOActive\_Belkin-advisory-lite.pdf](http://www.ioactive.com/pdfs/IOActive_Belkin-advisory-lite.pdf)
* [Implementing secure remote firmware updates](https://www.allegrosoft.com/wp-content/uploads/Secure-Firmware-Updates-Paper.pdf)
* [Code Integrity during execution by Automotive Grade Linux](http://docs.automotivelinux.org/docs/architecture/en/dev/reference/security/05-security-concepts.html#code-integrity-during-execution)
* [Securing Software Updates for Automobiles](https://uptane.github.io/)
