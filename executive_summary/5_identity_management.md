### ID 管理 {#5-identity-management}

組込みデバイス内のユーザーアカウントは本質的に静的であってはいけません。内部ウェブ管理、内部コンソールアクセス、リモートウェブ管理、リモートコンソールアクセスのためのユーザーアカウントの分離を可能にする機能は自動化された悪意のある攻撃を防ぐために利用可能であるべきです。

**検討事項:**

* ウェブ管理や製品ライン間の端末アクセスに使用される静的パスワードを使用すべきではありません。リリースプロセスの一環として削除すべきです。
  * 静的デフォルトパスワードを中間で使用する必要がある場合には、ユーザーがデバイスのセットアップおよびアクティベーション時にパスワード変更を強制されることを確認します。
  * すべての組込みアカウントにユーザーがパスワード、ピン、パスフレーズなどを変更するオプションがあることを確認します。
* 出荷前アカウント検証スクリプトは、適用可能である場合には、WIFI および WPS パスワードを検証するものと同様に調整されるべきです。
* ユーザーに対してリモートログインおよびローカルログインアカウント機能を実装します。
* SSH ログインと管理者ログインのユーザーを分離します。
* リモートログインは自動ブルートフォース攻撃を防ぐために一時的なアカウントロックアウト閾値を実装すべきです。
* ウェブ管理インタフェースについて
  * セッション ID は URL ではなくリクエストボディで送信されることを確認します。
  * セッション ID はユーザーがログアウトすると無効化されることを確認します。
  * セッション ID はパスワードが変更されると無効化されることを確認します。
  * セッション ID がクッキーに保存されている場合には、クッキーに HttpOnly フラグが設定されていることを確認します。
  * セッション ID はランダムであり、セッション間で変更されていることを確認します。
* ユーザー名、パスワード、セッション ID を含むクッキーはセキュアではないプロトコル (HTTP, FTP, Telnet など) で送信されないことを確認します。
* パスワードの複雑さのポリシーは "Password1" など簡単に推測できるパスワードを阻止するために強制されるべきです。複雑なパスワードは以下の属性を持つべきです。

  * 少なくとも 10 文字以上の長さ

  * 少なくとも 1 つの大文字

  * 少なくとも 1 つの数字

  * 少なくとも 1 つの小文字

  * 少なくとも 1 つの記号

* EEPROM は複雑さの要件を満たすパスワードで保護されていることを確認します。

#### その他の参考情報Additional References {#additional-references}

* [FTC Charges D-Link Put Consumers' Privacy at Risk Due to the Inadequate Security of Its Computer Routers and Cameras](https://www.ftc.gov/news-events/press-releases/2017/01/ftc-charges-d-link-put-consumers-privacy-risk-due-inadequate)
* [https://www.owasp.org/index.php/Testing_Identity_Management](https://www.owasp.org/index.php/Testing_Identity_Management)
* [https://www.owasp.org/images/6/67/OWASPApplicationSecurityVerificationStandard3.0.pdf](https://www.owasp.org/images/6/67/OWASPApplicationSecurityVerificationStandard3.0.pdf) (Page 26-31)
* [SB-327 Information privacy: connected devices](https://leginfo.legislature.ca.gov/faces/billNavClient.xhtml?bill_id=201720180SB327)
