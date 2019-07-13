# 組込みフレームワークとCベースツールチェーンの堅牢化

BusyBox, 組込みフレームワーク, ツールチェーンをファームウェアビルドの構成時に使用されるライブラリおよび関数のみに制限します。Buildroot, Yocto などの組込み Linux ビルドシステムは通常このタスクを実行します。Telnet など既知のセキュアではないライブラリやプロトコルを削除することで、ファームウェアビルドにおける攻撃エントリポイントを最小限に抑えるだけでなく、潜在的なセキュリティ上の脅威を防ぐことに取り組み、ソフトウェアを構築するためのセキュアバイデザイン (secure-by-design) アプローチも提供します。

**ライブラリの堅牢化** [**例**](https://www.owasp.org/index.php/C-Based_Toolchain_Hardening)**:** [圧縮はセキュアではありません](http://arstechnica.com/security/2012/09/crime-hijacks-https-sessions/) (特に), [SSLv2 はセキュアではありません](http://www.schneier.com/paper-ssl-revised.pdf), [SSLv3 はセキュアではありません](http://www.yaksman.org/~lweith/ssl.pdf), また TLS の初期バージョンも同様であることが知られています。さらに、ハードウェアやエンジンを使用せず、静的リンクのみを許可するとします。知識と仕様を考慮して、OpenSSL ライブラリを以下のように構築します。

```bash
$ Configure darwin64-x86_64-cc -no-hw -no-engine -no-comp -no-shared -no-dso -no-ssl2 -no-ssl3 --openssldir=
```

**一つのシェルを選択する例**: buildroot を使用して、以下のスクリーンショットでは bssh 一つだけのシェルが有効にされていることを示しています。 (注意: buildroot の例が以下に示されていますが、他の組込み Linux ビルドシステムにも同じ構成を達成するほかの方法があります。)


![](/assets/embedSec3.png)

**サービスの堅牢化例**: 以下のスクリーンショットは openssh が有効であることを示していますが、FTP デーモンの proftpd と pure-ftpd は無効になっています。TLS を利用する場合にのみ FTP を有効にします。例えば、proftpd と pureftpd は TLS を使用するためにカスタムコンパイルを必要とします。 proftpd には mod_tls を使用し、pureftpd には `./configure --with-tls` を渡します。

![](/assets/embedSec2.png)

**U-boot の堅牢化例: ** 多くの場合、組込みデバイスへの物理的なアクセスにより攻撃パスがブートローダー設定を変更することを可能にします。以下では、 `uboot_config` のベストプラクティス設定例が提供されています。注意: `uboot_config` ファイルは一般的にビルド環境と特定のボードに応じて自動生成されます。

U-Boot 2013.07 以降のバージョンでは "Verified Boot" (セキュアブート) を設定します。 Verified Boot はデフォルトでは有効になっておらず、少なくとも以下の構成でのボードサポートが必要です。

`CONFIG_ENABLE_VBOOT=y #Enables Verified Boot`

`CONFIG_FIT_SIGNATURE=y #Enables signature verification of FIT images.`

`CONFIG_RSA=y #Enables RSA algorithm used for FIT image verification`

`CONFIG_OF_SEPARATE=y #Enables separate build of u-Boot from the device tree.`

`CONFIG_FIT=y #Enables support for Flat Image Tree (FIT) uImage format.`

`CONFIG_OF_CONTROL=y #Enables Flattened Device Tree (FDT) configuration.`

`CONFIG_OF_LIBFDT=y`

`CONFIG_DEFAULT_DEVICE_TREE=y #Specifies the default Device Tree used for the run-time configuration of U-Boot.`

それから、Verified Boot を構成するために一連の手順が必要になります。[Beaglebone black ボード用の Verified Boot](https://github.com/siemens/u-boot/blob/master/doc/uImage.FIT/beaglebone_vboot.txt) 構築の概要例は以下の通りです。

1. Verified Boot オプションを有効にして、ボードの U-Boot を構築します。
2. 適切な Linux カーネル (できれば最新のもの) を入手します。
3. カーネルをどのようにパッケージ化、圧縮、署名したいかを記述した Image Tree Source (ITS) ファイルを作成します。
4. RSA2048 で RSA 鍵ペアを作成し、認証に SHA256 ハッシュアルゴリズムを使用します。 (秘密鍵を安全な場所に保管します。ファームウェアにハードコードしてはいけません。)
5. カーネルに署名します。
6. **公開鍵** を U-Boot のイメージに入れます。
7. U-Boot とカーネルをボードに入れます。
8. イメージとブート設定をテストします。

上記に加えて、適切な構成を組込みデバイスのコンテキストに合わせて有効にします。以下は注目すべき構成です。

`CONFIG_BOOTDELAY -2. #Prevents access to u-boot's console when auto boot is used`

`CONFIG_CMD_USB=n #Disables basic USB support and the usb command`

`CONFIG_USB_UHCI: defines the lowlevel part.`

`CONFIG_USB_KEYBOARD: enables the USB Keyboard`

`CONFIG_USB_STORAGE: enables the USB storage devices`

`CONFIG_USB_HOST_ETHER: enables USB ethernet adapter support`

以下の構成マクロで U-Boot のシリアルコンソール出力を無効にします。

`CONFIG_SILENT_CONSOLE`

`CONFIG_SYS_DEVICE_NULLDEV`

`CONFIG_SILENT_CONSOLE_UPDATE_ON_RELOC`

不変 (immutable) の U-Boot 環境編集を有効にして許可されていない変更 (bootargs の変更、Verified Boot 公開鍵の更新など) やファームウェアのサイドローディングを防止するには、以下のように不揮発性メモリ設定を削除します。

`#define CONFIG_ENV_IS_IN_MMC`

`#define CONFIG_ENV_IS_IN_NAND`

`#define CONFIG_ENV_IS_IN_NVRAM`

`#define CONFIG_ENV_IS_IN_SPI_FLASH`

`#define CONFIG_ENV_IS_IN_REMOTE`

`#define CONFIG_ENV_IS_IN_EEPROM`

`#define CONFIG_ENV_IS_IN_FLASH`

`#define CONFIG_ENV_IS_IN_DATAFLASH`

`#define CONFIG_ENV_IS_IN_MMC`

`#define CONFIG_ENV_IS_IN_FAT`

`#define CONFIG_ENV_IS_IN_ONENAND`

`#define CONFIG_ENV_IS_IN_UBI`

**検討事項 (免責: 以下のリストは完全なものではありません):**

* SSH などのサービスにセキュアなパスワードが作成されていることを確認します。
* perl, python, lua など未使用の言語インタプリタを削除します。
* 未使用ライブラリ関数からデッドコードを削除します。
* ash, dash, zsh などの未使用のシェルインタプリタを削除します。
  * `/etc/shell` をレビューします。
* 旧来のセキュアではないデーモンを削除します。以下を含みますがこれに限定されません。
  * telnetd
  * ftpd
  * ftpget
  * ftpput
  * tftp
  * rlogind
  * rshd
  * rexd
  * rcmd
  * rhosts
  * rexecd
  * rwalld
  * rbootd
  * rusersd
  * rquotad
  * rstatd
  * nfs
* 以下のような未使用／不要なユーティリティを削除します。

  * sed, wget, curl, awk, cut, df, dmesg, echo, fdisk, grep, mkdir, mount (vfat), printf, tail, tee, test (directory), test (file), head, cat

  [Automotive Grade Linux (AGL)](http://docs.automotivelinux.org/docs/architecture/en/dev/reference/security/07-system-hardening.html#removal-or-non-inclusion-of-utilities) はデバック環境または製品環境 (ビルド) での一般的なユーティリティとその使用法のサンプル表を開発しました。

| ユーティリティ名 | 場所 | デバッグ環境 | 製品環境 |
| :--- | :--- | :--- | :--- |
| Strace | /bin/trace | INCLUDE | EXCLUDE |
| Klogd | /sbin/klogd | INCLUDE | EXCLUDE |
| Syslogd\(logger\) | /bin/logger | INCLUDE | EXCLUDE |
| Gdbserver | /bin/gdbserver | INCLUDE | EXCLUDE |
| Dropbear | Remove “dropbear” from ‘/etc/init.d/rcs’ | EXCLUDE | EXCLUDE |
| SSH | NA | INCLUDE | EXCLUDE |
| Editors \(vi\) | /bin/vi | INCLUDE | EXCLUDE |
| Dmesg | /bin/dmesg | INCLUDE | EXCLUDE |
| UART | /proc/tty/driver/ | INCLUDE | EXCLUDE |
| Hexdump | /bin/hexdump | INCLUDE | EXCLUDE |
| Dnsdomainname | /bin/dnsdomainname | EXCLUDE | EXCLUDE |
| Hostname | /bin/hostname | INCLUDE | EXCLUDE |
| Pmap | /bin/pmap | INCLUDE | EXCLUDE |
| su | /bin/su | INCLUDE | EXCLUDE |
| Which | /bin/which | INCLUDE | EXCLUDE |
| Who and whoami | /bin/whoami | INCLUDE | EXCLUDE |
| ps | /bin/ps | INCLUDE | EXCLUDE |
| lsmod | /sbin/lsmod | INCLUDE | EXCLUDE |
| install | /bin/install | INCLUDE | EXCLUDE |
| logger | /bin/logger | INCLUDE | EXCLUDE |
| ps | /bin/ps | INCLUDE | EXCLUDE |
| rpm | /bin/rpm | INCLUDE | EXCLUDE |
| Iostat | /bin/iostat | INCLUDE | EXCLUDE |
| find | /bin/find | INCLUDE | EXCLUDE |
| Chgrp | /bin/chgrp | INCLUDE | EXCLUDE |
| Chmod | /bin/chmod | INCLUDE | EXCLUDE |
| Chown | /bin/chown | INCLUDE | EXCLUDE |
| killall | /bin/killall | INCLUDE | EXCLUDE |
| top | /bin/top | INCLUDE | EXCLUDE |
| stbhotplug | /sbin/stbhotplug | INCLUDE | EXCLUDE |

* 監査や提案を強化するために [Lynis](https://raw.githubusercontent.com/CISOfy/lynis/master/lynis) などのツールを利用します。 `wget --no-check-certificate https://github.com/CISOfy/lynis/archive/master.zip && unzip master.zip && cd lynis-master/ && bash lynis audit system`
  * `/var/log/lynis.log` のレポートをレビューします。
* 組込みデバイスで実行されているソフトウェアについて、開発者および関係者との間で繰り返し脅威モデル演習を実行します。

## その他の参考情報 <a id="additional-references"></a>

* [https://www.owasp.org/index.php/C-Based\_Toolchain\_Hardening](https://www.owasp.org/index.php/C-Based_Toolchain_Hardening)
* [https://www.bulkorder.ftc.gov/system/files/publications/pdf0199-carefulconnections-buildingsecurityinternetofthings.pdf](https://www.bulkorder.ftc.gov/system/files/publications/pdf0199-carefulconnections-buildingsecurityinternetofthings.pdf)
* [http://isa99.isa.org/Public/Documents/ISA-62443-4-1-WD.pdf](http://isa99.isa.org/Public/Documents/ISA-62443-4-1-WD.pdf) (page 34-38)
* [https://events.linuxfoundation.org/sites/events/files/slides/belloni-petazzoni-buildroot-oe\_0.pdf](https://events.linuxfoundation.org/sites/events/files/slides/belloni-petazzoni-buildroot-oe_0.pdf) - Details on buildroot and yocto
* [http://elinux.org/Toolchains](http://elinux.org/Toolchains)
* [https://download.pureftpd.org/pub/pure-ftpd/doc/README.TLS](https://download.pureftpd.org/pub/pure-ftpd/doc/README.TLS)
* [http://www.proftpd.org/docs/howto/TLS.html](http://www.proftpd.org/docs/howto/TLS.html)
* [https://www.owasp.org/index.php/Application\_Threat\_Modeling](https://www.owasp.org/index.php/Application_Threat_Modeling)
* [GNU C Library Vulnerability in Industrial Products](http://www.siemens.com/cert/pool/cert/siemens_security_advisory_ssa-301706.pdf)
* [Linux Exploit Quick Listing](http://www.kmbl.us/les/working.php)
* [Hardened U-boot](http://docs.automotivelinux.org/docs/architecture/en/dev/reference/security/07-system-hardening.html#hardened-boot)
  * [Verified boot](https://lwn.net/Articles/571031/)
* [Improving Your Embedded Linux Security Posture with Yocto](https://www.nccgroup.trust/globalassets/our-research/us/whitepapers/2018/improving-embedded-linux-security-yocto3.pdf)
