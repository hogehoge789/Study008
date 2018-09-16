
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

now | tomorrow
--- | --- |
DNS → | DNS over HTTPS
TCP + TLS  →| QUIC + TLS1.3

---

### @color[orange](TLS1.3)

* メジャーバージョンアップと言われる程の改修
* 0-1 RT ハンドシェイク
* AEAD ciphers
 * 古いアルゴリズムはとっぱらい
* forward security
* セッションチケットを使いまわさない

+++

### @color[orange](0-1 RT ハンドシェイク)

+++

![Alt Text](https://applech2.com/wp-content/uploads/2017/06/TLS-1-3-Overview-improve-dfficiency-1024x576.jpg)

+++

* TLS1.2
 * Client Helloからまずはパラメータ交換
 * その後に鍵の交換を経て通信暗号化

* TLS1.3
 * 1RTで通信暗号化
 * ServerHello毎に公開鍵を生成する

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

![Alt Text](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSGpFhkziHY_Pqrw6bmdCYlzWrvJOIn95gmn2EhFYT3GdoyqIm4Gg)

+++

* 機密性(confideniality)
 * データを覗かれていないか
* 完全性(integrity)
 * データが改ざんされていないか
* 可用性(availability)
 * データにアクセスできるか

+++

TLSは機密性と完全性は担保するが、可用性はTCPに依存

+++

現状のTLSではman-on-the-side attackが容易にできてしまう

+++

### @color[orange](QUICプロトコル)

* 暗号化とハンドシェイクを組み合わせ(TLS1.3)
* 0-1 RTハンドシェイク
* ヘッダ、ペイロードも暗号化
* コネクション内でストリームを多重化 
 * ストリーム0はTLS用らしい
* (TCPの)ヘッドオブラインブロッキング解消
* ネットワークモビリティ

+++
### @color[orange](構造イメージ)

![Alt Text](https://ma.ttias.be/wp-content/uploads/2016/07/tcp_udp_quic_http2_compared.png)

+++
### @color[orange](Quick UDP Internet Connections)
* 4タプルを使わない
 * src ip/port
 * dst ip/port
* 64bitのConnectionIDで識別

+++
IPアドレスが変わってもコネクションを維持できる

+++

### @color[orange](フロー制御、輻輳制御)
Rich Signaling for Congestion Control and Loss Recovery  
なんか色々あるっぽい

+++ 
### @color[orange](ストリーム多重化)
![Alt Text](https://ma.ttias.be/wp-content/uploads/2016/07/spdy_multiplexed_assets_head_of_line_blocked.png)

+++
### @color[orange](TCPのヘッドオブブロッキングを解消)
![Alt Text](https://ma.ttias.be/wp-content/uploads/2016/07/quic_multiplexing.png)

+++

### @color[orange](前方誤り訂正)
* 各パケットに必要以上のヘッダデータを含む
* ロスしたパケットを他パケットで復元できる

+++
### @color[orange](QUIC ハンドシェイク)

![Alt Text](https://ma.ttias.be/wp-content/uploads/2016/07/tcp_3_way_handshake_with_tls.png)
![Alt Text](https://ma.ttias.be/wp-content/uploads/2016/07/udp_quic_with_tls.png)

+++
3-wayハンドシェイクとTLSハンドシェイクを一体化

+++

### @color[orange](double encryption)
QUIC初期の問題  
QUICとTLSでそれぞれ2重に暗号化

* QUICのヘッダ・ペイロードの暗号化
* TLSのアプリケーション層の暗号化

+++

なんかクライアントとサーバで暗号化のタイミングがズレとかで大変だったらしい

+++

TLS上は暗号化されていてもQUICは暗号化されていなくて平文で通信している間はパケット挿入可能(ACK、RST)問題など

+++
TLSの方を変更

+++

* TLSのレイヤで行っていた暗号化の仕事を全てQUIC上で行う事に
* QUIC+TLSで5層構造
 * TLS レイヤが少なくなり、4層構造になった
 * ↑はQUIC レイヤで処理する

+++

* TLSは鍵交換と相手の認証(ハンドシェイク)
* QUICが暗号化

今のように1RTで暗号化とハンドシェイクを完結する流れにした

+++
### @color[orange](Packet Number暗号化)

![Alt Text](https://cdn-ak.f.st-hatena.com/images/fotolife/A/ASnoKaze/20171201/20171201003908.png)

+++

* Packet Numberは送信毎に一意だが、単調増加
* ネットワークが切り替わってもPacketNumberは使い続ける
* クライアントを追跡できる

+++
PacketNumberも切り替えるとか色々考えたけど面倒だからPacket Numberも暗号化する事にした(PNE)

+++
QUIC内で平文なのは下記のみ
* PacketType
* ConnectionID
* Protocol Version

---
### @color[orange](まとめ１) 
* QUIC + TLS1.3
 * 可用性改善
 * パフォーマンスを最適化

* 事業者がプロトコルを見て最適化すると硬直化が起こる
 * 結果的には中を見れない方がプロトコル改善できる

+++
### @color[orange](まとめ２) 

* 暗号化はQUICのタスク
 * ハンドシェイクはTLS
* だいたい暗号化される
* what to expose is decided explicitly

+++

* QUICは2つある
 * gQUIC →Google
 * iQUIC →IETF

---
### @color[orange](Encrypted SNI) 
SNIはClientHelloに含まれるので平文  
暗号化するぞ

+++
SNI
* Server Name Indication
* 1つのIPアドレスで複数ドメインの証明書を扱う機能
* これがないと証明書毎にパブリックIPが必要

+++
DNS over HTTPSでなんとかするらしい

+++
