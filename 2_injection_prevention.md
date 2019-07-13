### インジェクションの防止 {#2-injection-prevention}

意図しないシステム実行を防止するために、すべての信頼できないデータとユーザー入力が妥当性検証、サニタイズ、出力エンコードされていることを確認します。オペレーティングシステム (OS) コマンドインジェクション、クロスサイトスクリプティング (JavaScript インジェクションなど) 、SQL インジェクション、XPath インジェクションなどのさまざまなインジェクション攻撃がアプリケーションセキュリティ内に存在します。しかし、組込みソフトウェア内のインジェクション攻撃の中で最もよくあるものは OS コマンドインジェクションに関するものです。アプリケーションが信頼できない入力やセキュアではない入力を受け入れ、それを妥当性検証や適切なエスケープなしで外部アプリケーションに (アプリケーション名自体または引数として) 渡します。

オペレーティングシステムコールを使用する [**非準拠コード例**](https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=2130132) :

この非準拠コード例では、system() 関数を使用してホスト環境で any_cmd を実行します。コマンドプロセッサの呼び出しは必要ありません。

```c
#include <string.h>
#include <stdlib.h>
#include <stdio.h>

enum { BUFFERSIZE = 512 };

void func(const char *input) {
  char cmdbuf[BUFFERSIZE];
  int len_wanted = snprintf(cmdbuf, BUFFERSIZE,
                            "any_cmd '%s'", input);
  if (len_wanted >= BUFFERSIZE) {
    /* Handle error */
  } else if (len_wanted < 0) {
    /* Handle error */
  } else if (system(cmdbuf) == -1) {
    /* Handle error */
  }
}
```

このコードがコンパイルされ、Linux システム上にて特権で実行される場合、攻撃者は次の文字列を入力することでアカウントを作成できます。`any_cmd 'happy'; useradd 'attacker'` これは以下のように解釈されます。

```c
any_cmd 'happy';
useradd 'attacker'
```

**準拠例**:この準拠策では、`system()` の呼び出しは `execve()` の呼び出しに置き換えられています。exec 系の関数は完全なシェルインタプリタを使用しませんので、非準拠コード例に示すようなコマンドインジェクション攻撃に対して脆弱ではありません。

`execlp()`, `execvp()`, および (標準的ではない) `execvP()` 関数は、指定されたファイル名の先頭にスラッシュ文字 (/) がない場合、シェルの動作を真似て実行可能ファイルを探します。結果として、先頭のスラッシュ文字 (/) なしで使用されるのは、PATH 環境変数が安全な値に設定されている場合にのみであるべきです。

`execl()`, `execle()`, `execv()`, および `execve()` 関数はパス名の置き換えを行いません。

さらに、予防措置を講じて外部実行可能ファイルが信頼できないユーザーにより改変されないようにすべきです。例えば、実行可能ファイルがユーザーにより書き込み可能でないことを保証します。この準拠策は前述の非準拠コード例とは大きく異なります。第一に、入力は args 配列に組み込まれ `execve()` に引数として渡されます。コマンド文字列の生成時にバッファオーバーフローや文字列トランケーションの問題がなくなります。第二に、この準拠策は子プロセスで `/usr/bin/any_cmd` を実行する前に新しいプロセスをフォークします。この方法は system() を呼び出すよりも複雑ですが、追加されたセキュリティは追加の労力の価値があります。

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <errno.h>
#include <stdlib.h>

void func(char *input) {
  pid_t pid;
  int status;
  pid_t ret;
  char *const args[3] = {"any_exe", input, NULL};
  char **env;
  extern char **environ;

  /* ... Sanitize arguments ... */

  pid = fork();
  if (pid == -1) {
    /* Handle error */
  } else if (pid != 0) {
    while ((ret = waitpid(pid, &status, 0)) == -1) {
      if (errno != EINTR) {
        /* Handle error */
        break;
      }
    }
    if ((ret != -1) &&
      (!WIFEXITED(status) || !WEXITSTATUS(status)) ) {
      /* Report unexpected child status */
    }
  } else {
    /* ... Initialize env as a sanitized copy of environ ... */
    if (execve("/usr/bin/any_cmd", args, env) == -1) {
      /* Handle error */
      _Exit(127);
    }
  }
}
```

# 

**検討事項:**

* シェルコマンドラッパーを起動してはいけません。これは以下に限定されません。
  * PHP:`system()` `exec()`
  * C:`system()`
  * C++:`ShellExecute()` 
  * Lua:`os.execute()`
  * Perl:`system()` `exec()`
  * Python:`os.system()` `subprocess.call()`
* 可能であれば、オペレーションシステムコマンドにユーザーデータを使用することは避けます。
  * 必要に応じて、オペレーティングシステムに渡される可能性のあるユーザー駆動文字列に番号とコマンド文字列の参照マップを使用します。
* ホワイトリストは参照マップを介したコマンドを許可することで、期待されるパラメータ値のみが処理されることを保証します。
* エンコード文字のユーザーデータをコンテキストに合わせて出力することを保証します。 (例、 HTML, JavaScript, CSS, など)

  * HTML エンティティエンコード

    * &lt; は以下のようにエンコードして出力します。

      * ```
        &lt;
        ```

    * &gt; は以下のようにエンコードして出力します。

      * `&gt;`

#### その他の参考情報: {#additional-references}

* [Multiple Netgear routers are vulnerable to arbitrary command injection](https://www.kb.cert.org/vuls/id/582384)
* [FTC Charges D-Link Put Consumers’ Privacy at Risk Due to the Inadequate Security of Its Computer Routers and Cameras](https://www.ftc.gov/news-events/press-releases/2017/01/ftc-charges-d-link-put-consumers-privacy-risk-due-inadequate)
* [https://www.owasp.org/index.php/XSS_(Cross\_Site\_Scripting)_Prevention_Cheat_Sheet](https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet)
* [https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet](https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet)
* [https://www.owasp.org/images/2/2e/OWASP\_Code\_Review_Guide-V1_1.pdf](https://www.owasp.org/images/2/2e/OWASP_Code_Review_Guide-V1_1.pdf) (Page 117)
* [https://www.owasp.org/index.php/Command_Injection](https://www.owasp.org/index.php/Command_Injection)
* [http://cwe.mitre.org/data/definitions/77.html](http://cwe.mitre.org/data/definitions/77.html)
* [http://cwe.mitre.org/data/definitions/78.html](http://cwe.mitre.org/data/definitions/78.html)
* [Bash Command Injection Vulnerability](https://ics-cert.us-cert.gov/advisories/ICSA-14-269-01A)
