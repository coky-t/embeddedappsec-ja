### トランスポート層セキュリティ(TLS) {#8-transport-layer-security}


すべての通信手法が [TLS](https://www.securecoding.cert.org/confluence/display/c/API10-C.+APIs+should+have+security+options+enabled+by+default) の業界標準の暗号化設定利用していることを確認します。TLS を使用することで、すべてのデータは機密のままであり、転送中に改竄されないことを確実にします。組込みデバイスがドメイン名を使用する場合には [Let's Encrypt](https://letsencrypt.org/) などのフリーの認証局サービスを利用します。

[**例**](http://fm4dd.com/openssl/certverify.htm) **OpenSSL ライブラリ関数を使用して、ルート認証局に対して基本的な証明書検証を実行する方法:**

```c
#include <openssl/bio.h>
#include <openssl/err.h>
#include <openssl/pem.h>
#include <openssl/x509.h>
#include <openssl/x509_vfy.h>

int main() {

  const char ca_bundlestr[] = "./ca-bundle.pem";
  const char cert_filestr[] = "./cert-file.pem";

  BIO              *certbio = NULL;
  BIO               *outbio = NULL;
  X509          *error_cert = NULL;
  X509                *cert = NULL;
  X509_NAME    *certsubject = NULL;
  X509_STORE         *store = NULL;
  X509_STORE_CTX  *vrfy_ctx = NULL;
  int ret;

  /* ---------------------------------------------------------- *
   * These function calls initialize openssl for correct work.  *
   * ---------------------------------------------------------- */
  OpenSSL_add_all_algorithms();
  ERR_load_BIO_strings();
  ERR_load_crypto_strings();

  /* ---------------------------------------------------------- *
   * Create the Input/Output BIO's.                             *
   * ---------------------------------------------------------- */
  certbio = BIO_new(BIO_s_file());
  outbio  = BIO_new_fp(stdout, BIO_NOCLOSE);

  /* ---------------------------------------------------------- *
   * Initialize the global certificate validation store object. *
   * ---------------------------------------------------------- */
  if (!(store=X509_STORE_new()))
     BIO_printf(outbio, "Error creating X509_STORE_CTX object\n");

  /* ---------------------------------------------------------- *
   * Create the context structure for the validation operation. *
   * ---------------------------------------------------------- */
  vrfy_ctx = X509_STORE_CTX_new();

  /* ---------------------------------------------------------- *
   * Load the certificate and cacert chain from file (PEM).     *
   * ---------------------------------------------------------- */
  ret = BIO_read_filename(certbio, cert_filestr);
  if (! (cert = PEM_read_bio_X509(certbio, NULL, 0, NULL))) {
    BIO_printf(outbio, "Error loading cert into memory\n");
    exit(-1);
  }

  ret = X509_STORE_load_locations(store, ca_bundlestr, NULL);
  if (ret != 1)
    BIO_printf(outbio, "Error loading CA cert or chain file\n");

  /* ---------------------------------------------------------- *
   * Initialize the ctx structure for a verification operation: *
   * Set the trusted cert store, the unvalidated cert, and any  *
   * potential certs that could be needed (here we set it NULL) *
   * ---------------------------------------------------------- */
  X509_STORE_CTX_init(vrfy_ctx, store, cert, NULL);

  /* ---------------------------------------------------------- *
   * Check the complete cert chain can be build and validated.  *
   * Returns 1 on success, 0 on verification failures, and -1   *
   * for trouble with the ctx object (i.e. missing certificate) *
   * ---------------------------------------------------------- */
  ret = X509_verify_cert(vrfy_ctx);
  BIO_printf(outbio, "Verification return code: %d\n", ret);

  if(ret == 0 || ret == 1)
  BIO_printf(outbio, "Verification result text: %s\n",
             X509_verify_cert_error_string(vrfy_ctx->error));

  /* ---------------------------------------------------------- *
   * The error handling below shows how to get failure details  *
   * from the offending certificate.                            *
   * ---------------------------------------------------------- */
  if(ret == 0) {
    /*  get the offending certificate causing the failure */
    error_cert  = X509_STORE_CTX_get_current_cert(vrfy_ctx);
    certsubject = X509_NAME_new();
    certsubject = X509_get_subject_name(error_cert);
    BIO_printf(outbio, "Verification failed cert:\n");
    X509_NAME_print_ex(outbio, certsubject, 0, XN_FLAG_MULTILINE);
    BIO_printf(outbio, "\n");
  }

  /* ---------------------------------------------------------- *
   * Free up all structures                                     *
   * ---------------------------------------------------------- */
  X509_STORE_CTX_free(vrfy_ctx);
  X509_STORE_free(store);
  X509_free(cert);
  BIO_free_all(certbio);
  BIO_free_all(outbio);
  exit(0);
}
```

**検討事項 (免責事項: 以下のリストは網羅していません):**

* 新しい製品には最新の TLS バージョンを使用します (執筆時では、これは TLS 1.2 です)
* 許可されたクライアントの制限されたグループから TLS 接続を受け入れるファームウェアに対して、TLS 双方向認証を実装することを検討します。
* 可能であれば、相互認証を使用して両方のエンドポイントを認証することを検討します。
* 証明書の公開鍵、ホスト名、[チェーン](http://fm4dd.com/openssl/certverify.htm) を検証します。
* 証明書およびそのチェーンは署名に SHA256 を使用していることを確認します。
* 非推奨の SSL および初期の TLS バージョンを無効化します。
* 非推奨の NULL および脆弱な暗号スイートを無効化します。
* 秘密鍵および証明書は、Secure Environment や Trusted Execution Environment、または強力な暗号化を使用して保護するなど、セキュアに保存されていることを確認します。
* 最新のセキュアな設定で証明書を更新します。
* 有効期限が切れると適切な証明書更新機能が利用可能になることを確認します。
* [ssllabs.com](https://www.ssllabs.com), `--script ssl-enum-ciphers.nse` を使用した nmap, TestSSLServer.jar, sslscan, sslyze などのサービスを利用して TLS 設定を検証します。

**その他の例:**

TLS を利用するには、OpenSSL 以外の選択肢があります。非網羅的なリストは以下の通りです。

以前 PolarSSL と呼ばれていた mbed TLS を使用するプロジェクトのリストは以下にあります。

* [https://tls.mbed.org/kb/generic/projects-using-mbedtls](https://tls.mbed.org/kb/generic/projects-using-mbedtls)
* [https://tls.mbed.org/](https://tls.mbed.org/)

実装例は以下にあります。

* [https://tls.mbed.org/kb/how-to/mbedtls-tutorial](https://tls.mbed.org/kb/how-to/mbedtls-tutorial)

以前 CyaSSL と呼ばれていた wolfSSL 、および wolfSSL を使用するプロジェクトのリストは以下にあります。

* [https://www.wolfssl.com/wolfSSL/wolfssl-embedded-ssl-case-studies.html](https://www.wolfssl.com/wolfSSL/wolfssl-embedded-ssl-case-studies.html)
* [https://www.wolfssl.com/wolfSSL/Home.html](https://www.wolfssl.com/wolfSSL/Home.html)

実装例は以下にあります。

* [https://github.com/wolfSSL/wolfssl-examples](https://github.com/wolfSSL/wolfssl-examples)

#### その他の参考情報 {#additional-references}

* [https://letsencrypt.org/](https://letsencrypt.org/)
* [https://community.letsencrypt.org/t/certificate-for-embedded-device-without-a-domain-name/2372](https://community.letsencrypt.org/t/certificate-for-embedded-device-without-a-domain-name/2372)
* [http://fm4dd.com/openssl/](http://fm4dd.com/openssl/)
* [https://www.engadget.com/2015/04/19/wink-home-automation-hub-bricked/](https://www.engadget.com/2015/04/19/wink-home-automation-hub-bricked/)
