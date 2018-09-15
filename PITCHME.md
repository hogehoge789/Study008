
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

### @color[orange](TLS1.3)

* メジャーバージョンアップと言われる程の改修
* 0-1 RTT ハンドシェイク
* AEAD ciphers
 * 古いアルゴリズムはとっぱらい
* forward security
* セッションチケットを使いまわさない

+++

### @color[orange](0-1 RTT ハンドシェイク)

+++

![Alt Text](https://applech2.com/wp-content/uploads/2017/06/TLS-1-3-Overview-improve-dfficiency-1024x576.jpg)

+++

* TLS1.2
 * Client Helloからまずはパラメータ交換
 * その後に鍵の交換を経て通信暗号化

* TLS1.3
 * Client Helloから公開鍵暗号で暗号化

+++

* 一度交換した鍵を再利用する事で0 RTTもできる
* 再利用するのでセキュリティが下がる

+++

### @color[orange](TLS1.3において発生した過去の問題)

* 企業向けFWがTLS1.3をうまく理解しなかった
* 証明書が暗号化される事を不審な挙動と判定

+++

従来のFWでは証明書の中身を見てどこと通信しているか見ていたのだが、それが暗号化されてしまっていたのが理由

---

### @color[orange](Security of a transport)

* 機密性(confideniality)
 * データを覗かれていないか
* 完全性(integrity)
 * データが改ざんされていないか
* 可用性(availability)
 * データにアクセスできるか

+++

TLSは機密性と完全性は担保するが、可用性はTCPに依存

---

現状のTLSではman-on-the-side attackが容易にできてしまう

---

### @color[orange](QUICプロトコル)

* 暗号化とハンドシェイクを組み合わせ(TLS1.3)
* 0-1 RTハンドシェイク
* コネクション内でストリームを多重化 
* (TCPの)ヘッドオブラインブロッキング解消
* ネットワークモビリティ
 * QUICでは4タプルに依存しないコネクションが確保
 * 通信毎に固有のIDを保持
 * 無線通信やモバイルで強い

+++
構造イメージ

![Alt Text](https://ma.ttias.be/wp-content/uploads/2016/07/tcp_udp_quic_http2_compared.png)

+++
* Quick UDP Internet Connections
* いわゆる4タプルがない
 * src ip/port
 * dst ip/port
* UDPですし

+++

### @color[orange](フロー制御、輻輳制御)
Rich Signaling for Congestion Control and Loss Recovery

+++

![Alt Text](https://docs.google.com/presentation/d/13LSNCCvBijabnn1S4-Bb6wRlm79gN6hnPFHByEXXptk/present?slide=id.g17a0599c4_1164)

+++ 
### @color[orange](ストリーム多重化)

![Alt Text](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT0t9nLS1ULcbzJONF9MXGXlSHviXAnu--yzVr-wBSAdWeRcX9S)

+++

### @color[orange](前方誤り訂正)

 
+++

### @color[orange](double encryption)
QUIC+TLSで2重暗号化



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