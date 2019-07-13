# 機密情報の保護

パスワード、ユーザー名、トークン、秘密鍵などの機密情報をファームウェアリリースイメージにハードコードしてはいけません。これにはディスクに書き込まれる機密データの保管も含まれます。ハードウェアセキュリティエレメント (SE) や Trusted Execution Environment (TEE) が利用可能な場合は、そのような機能を機密データの保存に使用することを推奨します。もしくは、データを保護するために強力な暗号化の使用を評価すべきです。

可能であれば、平文のすべての機密データは本質的に一過性であり、揮発性メモリにのみ存在すべきです。

**非準拠** [**ハードコードされたパスワード**](https://www.owasp.org/index.php/Use_of_hard-coded_password) **例:**

```c
int VerifyAdmin(char *password) {

  if (strcmp(password, "Mew!")) {
    printf("Incorrect Password!\n");
    return 0;
  }

  printf("Entering Diagnostic Mode\n");
  return 1;
}
```

**非準拠** [**機密データをディスクに保存**](https://www.securecoding.cert.org/confluence/display/c/MEM06-C.+Ensure+that+sensitive+data+is+not+written+out+to+disk) **例:**

この非準拠コード例では、機密情報はおそらく動的に割り当てられたバッファ secret に格納され、処理され、最終的に `memset_s()` の呼び出しによりクリアされます。secret を含むメモリページがディスクにスワップアウトされる可能性があります。`memset_s()` の呼び出しが完了する前にプログラムがクラッシュすると、secret に格納されている情報はコアダンプに保存される可能性があります。

```c
char *secret;

secret = (char *)malloc(size+1);
if (!secret) {
  /* Handle error */
}

/* Perform operations using secret... */

memset_s(secret, '\0', size+1);
free(secret);
secret = NULL;
```

情報がコアダンプに書き込まれることを防ぐには、プログラムが生成するコアダンプのサイズを `setrlimit()` を使用して 0 に設定すべきです。

```c
#include <sys/resource.h>
/* ... */
struct rlimit limit;
limit.rlim_cur = 0;
limit.rlim_max = 0;
if (setrlimit(RLIMIT_CORE, &limit) != 0) {
    /* Handle error */
}

char *secret;

secret = (char *)malloc(size+1);
if (!secret) {
  /* Handle error */
}

/* Perform operations using secret... */

memset_s(secret, '\0', size+1);
free(secret);
secret = NULL;
```

あるいは、`mlock()` を使用してメモリを所定の位置にロックすることによりページングを防ぐことができます。この準拠策はコアファイルの作成を無効化するだけでなく、バッファがハードディスクにスワップされないようにします。

```c
#include <sys/resource.h>
/* ... */
struct rlimit limit;
limit.rlim_cur = 0;
limit.rlim_max = 0;
if (setrlimit(RLIMIT_CORE, &limit) != 0) {
    /* Handle error */
}

long pagesize = sysconf(_SC_PAGESIZE);
if (pagesize == -1) {
  /* Handle error */
}

char *secret_buf;
char *secret;

secret_buf = (char *)malloc(size+1+pagesize);
if (!secret_buf) {
  /* Handle error */
}

/* mlock() may require that address be a multiple of PAGESIZE */
secret = (char *)((((intptr_t)secret_buf + pagesize - 1) / pagesize) * pagesize);

if (mlock(secret, size+1) != 0) {
    /* Handle error */
}

/* Perform operations using secret... */

memset_s(secret_buf, '\0', size+1+pagesize);
if (munlock(secret, size+1) != 0) {
    /* Handle error */
}
secret = NULL;

free(secret_buf);
secret_buf = NULL;
```

**機密データの保存, 非準拠** [**例**](https://www.securecoding.cert.org/confluence/display/c/MEM03-C.+Clear+sensitive+information+stored+in+reusable+resources): この例では、secret により参照される動的に割り当てられたメモリに格納された機密情報が動的に割り当てられたバッファ `new_secret` にコピーされ、処理され、最終的に `free()` の呼び出しにより割り当て解除されます。メモリがクリアされていないため、プログラムの別のセクションに再割り当てされる可能性があり、`new_secret` に格納された情報は意図せず漏洩する可能性があります。

```c
char *secret;

/* Initialize secret */

char *new_secret;
size_t size = strlen(secret);
if (size == SIZE_MAX) {
  /* Handle error */
}

new_secret = (char *)malloc(size+1);
if (!new_secret) {
  /* Handle error */
}
strcpy(new_secret, secret);

/* Process new_secret... */

free(new_secret);
new_secret = NULL;
```

**機密データの保存, 準拠例**: 情報漏洩を防ぐには、機密情報を含む動的メモリは解放される前にサニタイズされるべきです。サニタイズは通常割り当てられたスペースをクリアすることにより実行されます (つまり、スペースを '\0' 文字で埋めます) 。

```c
char *secret;

/* Initialize secret */

char *new_secret;
size_t size = strlen(secret);
if (size == SIZE_MAX) {
  /* Handle error */
}

/* Use calloc() to zero-out allocated space */
new_secret = (char *)calloc(size+1, sizeof(char));
if (!new_secret) {
  /* Handle error */
}
strcpy(new_secret, secret);

/* Process new_secret... */

/* Sanitize memory  */
memset_s(new_secret, '\0', size);
free(new_secret);
new_secret = NULL;
```

**検討事項:**

* 製品ライン間で証明書をハードコードしてはいけません。
* 製品ライン間でパスワードをハードコードしてはいけません。
* 保護されていない保管場所や EEPROM やフラッシュなどの外部ストレージに機密情報を格納してはいけません。

## その他の参考情報 <a id="additional-references"></a>

* [https://cwe.mitre.org/data/definitions/259.html](https://cwe.mitre.org/data/definitions/259.html)
* [https://cwe.mitre.org/data/definitions/798.html](https://cwe.mitre.org/data/definitions/798.html)
* [https://cwe.mitre.org/data/definitions/321.html](https://cwe.mitre.org/data/definitions/321.html)
* [CWE-321: Use of Hard-coded Cryptographic Key - CVE-2013-6952](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-6952)
  * [http://www.ioactive.com/pdfs/IOActive_Belkin-advisory-lite.pdf](http://www.ioactive.com/pdfs/IOActive_Belkin-advisory-lite.pdf)
