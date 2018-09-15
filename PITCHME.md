
## builderscon 2018 Tokyo
勉強会資料  


by GitPitch

+++

@color[orange](次世代通信プロトコルにおけるセキュリティ・プライバシー保護・パフォーマンス)

スピーカー:@kazuho oku

+++

セッションが人気だったので  
内容を整理してみました。

+++
#### @color[orange](内容について)

* QUICとTLS1.3についての内容
* パフォーマンスは時間の都合上割愛

---

#### @color[orange](インターネットプロトコル変化の時代)
なぜ変わろうとしているのか
  
* 2013年頃に発生したスノーデン事件を皮切り
* そのネットワークは信用できるのか？
* ネットワークへの信頼が揺らいでしまった

+++

技術での公平な暗号化が必要

+++

* 結果、通信の完全な暗号化の動きが出てきた

now | tommorrow
--- | --- |
DNS → | DNS over HTTPS
TCP + TLS  →| QUIC + TLS1.3

---

#### @color[orange](アジェンダ)
* TLS1.3
* Security of a transport
* QUIC
 * handshake
 * PN encryption
* Encryptd SNI

---

### TLS1.3

* メジャーバージョンアップと言われる程の改修
* 0-1 RTT ハンドシェイク
* AEAD ciphers
* forward securityが要件として入った
* セッションチケットを使いまわさない

+++

### 0-1 RTT ハンドシェイク

+++

![Alt Text](https://www.internetsociety.org/wp-content/uploads/2017/06/Screenshot-2017-06-20-22.15.45.png)

+++

* TLS1.2
 * Client Helloを起点にパラメータ交換を実施
 * 鍵の交換を経て通信暗号化

* TLS1.3
 * Client Helloから公開鍵暗号で暗号化

+++

### TLS1.3において発生した過去の問題

* 企業向けFWがTLS1.3をうまく理解しなかった
* 証明書が暗号化される事を不審な挙動と判定してしまう

従来のFWでは証明書の中身を見てどこと通信しているか見ていたのだが、それが暗号化されてしまっていたのが理由

---

### セキュリティ3原則

* 機密性(confideniality)
* 完全性(integrity)
* 可用性(availability)

TLSは機密性と完全性は担保するが、可用性はTCPに依存している

ファイアーウォール等がTCPレベルで切断ができてしまう

---

on-path attack
man-on-the-side attack
off-path attack

現状のTLSではman-on-the-side attackが容易にできてしまう
DTLSやIPsecなどは耐性がある

---

## QUICプロトコル


---

## QUICの特徴
* TLS1.3を使った暗号化通信
* 0-1RTハンドシェイク
QUIC Cryptoという独自の暗号化方式があったが、TLS1.3に置き換えられている

* 複数のストリームを1コネクションにまとめる multiplexing streams 
* HTTP2(というよりTCP)の問題であるパケットの順番が保証されていないと読み込めない配信方式が改善されて高速な通信が見込める
→ヘッドオブラインブロッキングが解消されている
* モビリティ
** 通信毎に固有のIDを保持
** 無線通信やモバイルで強い
** QUICでは4タプルに依存しないコネクションが確保される

---

QUIC+TLSで2重の暗号化がかかってしまう

各々、フローが異なるので信頼するタイミングがズレがあるので
暗号化通信をいつ始めていいかわからない、対応が難しい

TLS上は暗号化されていても、QUICは暗号化されていなくて平文なので
平文で通信している間は当然ながらパケット挿入可能(ACK、RST)
→TLSプロトコルを変更した

TLSのレイヤで行っていた暗号化の仕事を全てQUIC上で行う。
QUIC+TLSで5層構造
→TLS recordレイヤがなくなり、4層構造になった
QUIC packetレイヤで処理する

TLSは鍵交換と相手の認証
QUICが暗号化

結果、1RTTで暗号化が完結する流れとなった
→注入攻撃に対応

---

コネクションID？
パケット番号？

コネクションIDが切り替わった瞬間のパケット番号がわかるとその瞬間がバレてしまう

パケット番号も暗号化する事に(PNE)

パケット番号の暗号化：
AES_GCM(PN, payload)→ciphertext+AEAD_TAG, AES_CTR(ciphertext, PN)→PNE
※AES_CTRのnonceにchipertextの先頭使う

明示的に決められた情報だけが平文で通信される 

---

QUICはTCPを使用しないので逆順でも再送要求が発生しない
そもそも、パケット番号が暗号化されているのでそもそも再送要求ができない

TCPの場合、携帯電話の基地局はパケットの順序がずれたときのために再送を減らすために通信をバッファリングしている。
効果は5％くらいある。

ただし、QUICではパケット番号を暗号化するのでバッファリングの意味がなくなる。

---
QUICではTLSがハンドシェイク、QUICが暗号化
ほとんどすべての通信が暗号化される(完全にすべてではない))



---

Encrypted SNI
SNIの暗号化





---