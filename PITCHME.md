
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
* QUICのペイロードも暗号化
* コネクション内でストリームを多重化 
 * ストリーム0はTLS用らしい
* (TCPの)ヘッドオブラインブロッキング解消
* ネットワークモビリティ

+++
### @color[orange](構造イメージ)

![Alt Text](https://ma.ttias.be/wp-content/uploads/2016/07/tcp_udp_quic_http2_compared.png)

+++
* Quick UDP Internet Connections
* いわゆる4タプルがない
 * src ip/port
 * dst ip/port
* UDPですし
* 64bitのConnectionIDで識別
* ConnectionID以外全て暗号化

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
* 各パケットに必要以上のペイロードを含む
* ロスしたパケットを他パケットで復元できる

+++
### @color[orange](QUIC ハンドシェイク)

![Alt Text](https://www.google.co.jp/imgres?imgurl=x-raw-image%3A%2F%2F%2F96932850de81b2d78503fca884be6056ddbd185edae94e2cf4a670ff656f47ba&imgrefurl=http%3A%2F%2Fwww.soumu.go.jp%2Fmain_content%2F000485068.pdf&docid=berV8EilmmRr2M&tbnid=17KB3lAok5aU0M%3A&vet=10ahUKEwjpso2DiLzdAhWMfbwKHU3OBuIQMwhAKAMwAw..i&w=821&h=442&bih=764&biw=1600&q=quic%20%E3%83%8F%E3%83%B3%E3%83%89%E3%82%B7%E3%82%A7%E3%82%A4%E3%82%AF&ved=0ahUKEwjpso2DiLzdAhWMfbwKHU3OBuIQMwhAKAMwAw&iact=mrc&uact=8)

+++
3-wayハンドシェイクとTLSハンドシェイクを一体化

+++

### @color[orange](double encryption)
QUIC初期の問題  
QUICとTLSでそれぞれ2重に暗号化

+++

なんかクライアントとサーバで暗号化のタイミングがズレとかで大変だったらしい

+++

TLS上は暗号化されていても、QUICは暗号化されていなくて平文なので
平文で通信している間はパケット挿入可能(ACK、RST)

+++
TLSの方を変更

+++

TLSのレイヤで行っていた暗号化の仕事を全てQUIC上で行う。
QUIC+TLSで5層構造
→TLS recordレイヤがなくなり、4層構造になった
QUIC packetレイヤで処理する

+++

* TLSは鍵交換と相手の認証
* QUICが暗号化

1RTで暗号化が完結する流れにした

+++

コネクションIDが切り替わった瞬間のパケット番号がわかるとその瞬間がバレてしまう  
パケット番号も暗号化する事に(PNE)

+++

パケット番号の暗号化：
AES_GCM(PN, payload)→ciphertext+AEAD_TAG, AES_CTR(ciphertext, PN)→PNE
※AES_CTRのnonceにchipertextの先頭使う

明示的に決められた情報だけが平文で通信される 

+++

gQUIC  
→Google

iQUIC  
→IETF

---

Encrypted SNI
SNIの暗号化

---