# バッファおよびスタックオーバーフローの保護

ファームウェア内のメモリ破損の脆弱性から保護するために、既知の危険な関数や API の使用を防止します。( 例 [安全でない C 関数](https://www.securecoding.cert.org/confluence/display/c/VOID+MSC34-C.+Do+not+use+deprecated+and+obsolete+functions) の使用 - [strcat, strcpy, sprintf, scanf](http://cwe.mitre.org/data/definitions/676.html#Demonstrative_Examples) )。バッファオーバーフローなどのメモリ破損の脆弱性はスタックのオーバーフロー ( [スタックオーバーフロー](https://en.wikipedia.org/wiki/Stack_buffer_overflow) ) やヒープのオーバーフロー ( [ヒープオーバーフロー](https://en.wikipedia.org/wiki/Heap_overflow) ) からなります。わかりやすくするために、本書ではこの二種類の脆弱性を区別しません。バッファオーバーフローが検出され、攻撃者により悪用された場合、命令ポインタレジスタは上書きされ、攻撃者により提供された任意の悪質なコードが実行されます。

**ソースコードの脆弱な C 関数を見つけます。例： "C" リポジトリ内で下記の "find" コマンドを使用して、ソースコードの "strncpy" や "strlen" などの脆弱な C 関数を見つけます。**

```
find . -type f -name '*.c' -print0|xargs -0 grep -e 'strncpy.*strlen'|wc -l
```

**また grep 式は以下の式のように利用できます。**

```
$ grep -E '(strcpy|strcat|strncat|sprintf|strlen|memcpy|fopen|gets)' fuzzgoat.c
   memcpy (&state.settings, settings, sizeof (json_settings));
            {  sprintf (error, "Unexpected EOF in string (at %d:%d)", line_and_col);
                        sprintf (error, "Invalid character value `%c` (at %d:%d)", b, line_and_col);
                            sprintf (error, "Invalid character value `%c` (at %d:%d)", b, line_and_col);
                  {  sprintf (error, "%d:%d: Unexpected EOF in block comment", line_and_col);
               {  sprintf (error, "%d:%d: Comment not allowed here", line_and_col);
               {  sprintf (error, "%d:%d: EOF unexpected", line_and_col);
                     sprintf (error, "%d:%d: Unexpected `%c` in comment opening sequence", line_and_col, b);
                  sprintf (error, "%d:%d: Trailing garbage: `%c`",
                  {  sprintf (error, "%d:%d: Unexpected ]", line_and_col);
                        sprintf (error, "%d:%d: Expected , before %c",
                        sprintf (error, "%d:%d: Expected : before %c",
                        {  sprintf (error, "%d:%d: Unexpected %c when seeking value", line_and_col, b);
                     {  sprintf (error, "%d:%d: Expected , before \"", line_and_col);
                     sprintf (error, "%d:%d: Unexpected `%c` in object", line_and_col, b);
                        {  sprintf (error, "%d:%d: Unexpected `0` before `%c`", line_and_col, b);
                  {  sprintf (error, "%d:%d: Expected digit before `.`", line_and_col);
                     {  sprintf (error, "%d:%d: Expected digit after `.`", line_and_col);
                  {  sprintf (error, "%d:%d: Expected digit after `e`", line_and_col);
   sprintf (error, "%d:%d: Unknown value", line_and_col);
   strcpy (error, "Memory allocation failure");
   sprintf (error, "%d:%d: Too long (caught overflow)", line_and_col);
         strcpy (error_buf, error);
         strcpy (error_buf, "Unknown error");
```

**以下に、C ソースコードに対して実行した flawfinder の出力例を示します。**

```
$ flawfinder fuzzgoat.c 
Flawfinder version 1.31, (C) 2001-2014 David A. Wheeler.
Number of rules (primarily dangerous function names) in C/C++ ruleset: 169
Examining fuzzgoat.c

FINAL RESULTS:

fuzzgoat.c:1049:  [4] (buffer) strcpy:
  Does not check for buffer overflows when copying to destination (CWE-120).
  Consider using strcpy_s, strncpy, or strlcpy (warning, strncpy is easily
  misused).
fuzzgoat.c:368:  [2] (buffer) memcpy:
  Does not check for buffer overflows when copying to destination (CWE-120).
  Make sure destination can always hold the source data.
fuzzgoat.c:401:  [2] (buffer) sprintf:
  Does not check for buffer overflows (CWE-120). Use sprintf_s, snprintf, or
  vsnprintf. Risk is low because the source has a constant maximum length.
<SNIP>
fuzzgoat.c:1036:  [2] (buffer) strcpy:
  Does not check for buffer overflows when copying to destination (CWE-120).
  Consider using strcpy_s, strncpy, or strlcpy (warning, strncpy is easily
  misused). Risk is low because the source is a constant string.
fuzzgoat.c:1041:  [2] (buffer) sprintf:
  Does not check for buffer overflows (CWE-120). Use sprintf_s, snprintf, or
  vsnprintf. Risk is low because the source has a constant maximum length.
fuzzgoat.c:1051:  [2] (buffer) strcpy:
  Does not check for buffer overflows when copying to destination (CWE-120).
  Consider using strcpy_s, strncpy, or strlcpy (warning, strncpy is easily
  misused). Risk is low because the source is a constant string.
ANALYSIS SUMMARY:

Hits = 24
Lines analyzed = 1082 in approximately 0.02 seconds (59316 lines/second)
Physical Source Lines of Code (SLOC) = 765
Hits@level = [0]   0 [1]   0 [2]  23 [3]   0 [4]   1 [5]   0
Hits@level+ = [0+]  24 [1+]  24 [2+]  24 [3+]   1 [4+]   1 [5+]   0
Hits/KSLOC@level+ = [0+] 31.3725 [1+] 31.3725 [2+] 31.3725 [3+] 1.30719 [4+] 1.30719 [5+]   0
Minimum risk level = 1
Not every hit is necessarily a security vulnerability.
There may be other security vulnerabilities; review your code!
See 'Secure Programming for Linux and Unix HOWTO'
(http://www.dwheeler.com/secure-programs) for more information.
```

非推奨関数の使用, [**非準拠コードの例**](https://www.securecoding.cert.org/confluence/display/c/VOID+STR35-C.+Do+not+copy+data+from+an+unbounded+source+to+a+fixed-length+array): この非準拠コード例では gets() が stdin から BUFSIZ - 1 文字以上を読み込まないと仮定しています。これは無効な仮定であり、操作の結果としてバッファオーバーフローを引き起こす可能性があります。なお BUFSIZ は stdio.h で定義されているマクロ整数定数であり、 setvbuf() への推奨引数を表していますが、このような入力バッファの最大値とするものではないことに注意します。

gets() 関数は、ファイルの終わりに到達するか改行文字が読み込まれるまで、stdin から出力先配列に文字を読み込みます。改行文字は破棄され、配列に読み込まれた最後の文字の直後にヌル文字が書き込まれます。

```c
#include <stdio.h>

void func(void) {
  char buf[BUFSIZ];
  if (gets(buf) == NULL) {
    /* Handle error */
  }
}
```

**準拠例**: fgets() 関数は指定された数より少ない文字をストリームから配列に読み込みます。標準入力から buf にコピーされるバイト数は確保されたメモリを超えることができないため、このソリューションは準拠しています。

```c
#include <stdio.h>
#include <string.h>

enum { BUFFERSIZE = 32 };

void func(void) {
  char buf[BUFFERSIZE];
  int ch;

  if (fgets(buf, sizeof(buf), stdin)) {
    /* fgets succeeds; scan for newline character */
    char *p = strchr(buf, '\n');
    if (p) {
      *p = '\0';
    } else {
      /* Newline not found; flush stdin to end of line */
      while (((ch = getchar()) != '\n')
            && !feof(stdin)
            && !ferror(stdin))
        ;
    }
  } else {
    /* fgets failed; handle error */
  }
}
```

strncat() は strcat() ライブラリ関数をオリジナルとするバリエーションです。いずれもヌル終端された C 文字列を別の文字列に付け加えるために使用されます。オリジナルの strcat() の危険性は、呼出元が受信バッファに収まるよりも多くのデータを提供し、それによりオーバーランニングする可能性があるというものでした。これの最も一般的な結果はセグメンテーション違反です。もっと悪い結果は、メモリ内の受信バッファにどんなものが続いてもサイレントで検出されない破損です。

strncat() にはユーザーがコピーする最大バイト数を指定できる追加のパラメータが加わりました。これはコピーするデータ量ではありません。これはソースデータのサイズではありません。これはコピーするデータ量の制限であり、通常は受信バッファのサイズに設定されます。

**"strncat" を使用した準拠例:**

```c
char buffer[SOME_SIZE];

strncat( buffer, SOME_DATA, sizeof(buffer)-1);
```

**"strncat" を使用した非準拠例:**

```c
strncat( buffer, SOME_DATA, strlen( SOME_DATA ));
```

以下のスクリーンショットは buildroot を使用してファームウェアイメージを構築する際にスタック保護サポートが有効化されていることを示しています。
![](/assets/embedSec1.png)

**検討事項:**

* バッファの種類とその場所は何ですか: 物理メモリ、論理メモリ、仮想メモリ。
* バッファが解放されたとき、または LRU に退出したときに何のデータが残りますか。
* 古いバッファがデータをリークしないようにするための戦略は何ですか。(例：使用後にバッファをクリアします。)
* バッファを割り当て時に既知の値に初期化します。
* 変数が格納されている場所を考慮します: スタック、静的、割り当てられた構造体。
* 実行時にバッファや一時ファイルに保存された機密情報は、それらが不要になった後、廃棄しセキュアに消し去ります。(例えば、バッファを解放する前に個人識別情報 (PII) がある場所のバッファをきれいにします。)
* 明示的に変数を初期化します。
* 各ファームウェアのビルド時にセキュアなコンパイラフラグやスイッチが使用されていることを確認します。(例えば、GCC では -fPIE, -fstack-protector-all, -Wl,-z,noexecstack, -Wl,-z,noexecheap など。詳細はその他の参考情報セクションを参照してください。)
* (以下の非網羅的リストにあるような) 既知の脆弱な関数には安全な同等の関数を利用します。
  * `gets() -> fgets()`
  * `strcpy() -> strncpy()`
  * `strcat() -> strncat()`
  * `sprintf() -> snprintf()`
* 安全な同等物がない関数は書き直して安全なチェックを実装する必要があります。
* FreeRTOS OS を利用する場合は、開発およびテストフェーズではフック関数で "configCHECK_FOR_STACK_OVERFLOW" に "1" を設定し、製品ビルドでは削除することを検討します。

## その他の参考情報 <a href="#additional-references" id="additional-references"></a>

* OSS (オープンソースソフトウェア) 静的解析ツール
  * C 用 [flawfinder](http://www.dwheeler.com/flawfinder/) および [PMD](https://pmd.github.io/) の使用
  * [C++](https://github.com/struct/mms/blob/master/Modern\_Memory\_Safety\_In\_C\_CPP.pdf) 用 [cppcheck](http://cppcheck.sourceforge.net/) の使用
  * Clang Static Analysis を使用した C, C++, iOS 用の [Codechecker](https://github.com/Ericsson/codechecker) および [Infer](https://fbinfer.com/) の検討
* [http://www.dwheeler.com/secure-programs/Secure-Programs-HOWTO/library-c.html](http://www.dwheeler.com/secure-programs/Secure-Programs-HOWTO/library-c.html)
* [https://www.owasp.org/index.php/C-Based\_Toolchain\_Hardening\#GCC.2FBinutils](https://www.owasp.org/index.php/C-Based\_Toolchain\_Hardening#GCC.2FBinutils)
* [https://www.owasp.org/index.php/Buffer\_overflow\_attack](https://www.owasp.org/index.php/Buffer\_overflow\_attack)
* [https://www.owasp.org/images/2/2e/OWASP\_Code\_Review\_Guide-V1\_1.pdf](https://www.owasp.org/images/2/2e/OWASP\_Code\_Review\_Guide-V1\_1.pdf) (Page 113-114)
* [University of Pittsburgh - Secure Coding C/C++: String Vulnerabilities (PDF)](http://www.sis.pitt.edu/jjoshi/courses/IS2620/Spring07/Lecture3.pdf)
* [Intel Open Source Technology Center SDL Banned Functions](https://github.com/01org/safestringlib/wiki/SDL-List-of-Banned-Functions)
* [RTOS Stack Overflow Checking](http://www.freertos.org/Stacks-and-stack-overflow-checking.html)
