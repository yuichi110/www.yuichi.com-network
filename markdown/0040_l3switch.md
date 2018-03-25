# L3スイッチとネットワーク仮想化

{{ TOC }}

## L3スイッチ

第二章でスイッチについて学び、第三章でルーターについて学びました。
本章では両者の機能を持つ、つまりL2スイッチングもL3ルーティングもできるL3スイッチについて解説します。

2000年より前はルーターとスイッチの役割が明確に分かれていました。
企業内でネットワークを作ろうと思えば、スイッチでネットワークを作り、ルーターでネットワークを繋ぐことが必須でした。
この状況はL3スイッチが発売されると一変しました。
便利だからと家庭内でルーターにスイッチの機能を融合したブロードバンドルーターが使われるように、
企業もルーターとスイッチの両方の機能を持った便利な機器を欲しがり、スイッチにルーターの機能を融合したL3スイッチが採用されるようになったのです。
私もオフィスやデータセンターのネットワークを設計することがありますが、
その時に中心となる機器はやはりL3スイッチです。

L3スイッチは「スイッチ」と名前がついていることから分かるように、ルーターよりは今まで学んできたL2スイッチに似ています。
多くのポートを持っているので多数の機器を接続でき、なおかつハードウェアで転送処理を行うため、パケットの転送速度は高速です。
ただ、L2スイッチと違ってネットワーク間のパケット転送(ルーティング)や、ルーティングの下地を作るためのルーティングプロトコルなどを利用できます。

なお、L3スイッチをルーターとして使うこともできますが、
ルーターとして販売されている製品ほど多くの機能を持っていない場合が多いです。
例えばNATやVPNといった機能は多くのL3スイッチが提供していません。
そのため、現在の典型的な企業ネットワークでは社内でのルーティング処理は全てL3スイッチで行い、
「社内と社外の境界」や「本社と支社の境界」にのみルーターを使うという構成が多くなっています。
こういった境界のみ先ほど述べたNATやVPNといった機能が必要となってくるためです。

また、L3転送機能を持たないL2専用のスイッチも利用されています。
ずばり言うと、両者の違いは値段です。
同じパフォーマンスであれば、一般的にL2スイッチのほうがL3スイッチより安価です。
スイッチを利用する箇所全てで必ずL3転送が必要なわけではありませんので、
そういったところではL2スイッチが利用されます。



### ルーターを使ったVLAN間でのルーティング

VLANを使ってネットワークを分離したのはいいのですが、VLAN間でも通信をできるようにする必要があります。
これこそL3スイッチの出番なのですが、知らない人には話が飛びすぎるためルーターを使って実現します。

準備として先ほどのVLAN2のPCにアドレスを振り直して、デフォルトゲートウェイを設定します。

```
PC1(config)#ip route 0.0.0.0 0.0.0.0 10.0.1.1

PC2(config)#ip route 0.0.0.0 0.0.0.0 10.0.1.1

PC3(config)#ip route 0.0.0.0 0.0.0.0 10.0.2.1
PC3(config)#int g0/1
PC3(config-if)#ip address 10.0.2.103 255.255.255.0

PC4(config)#ip route 0.0.0.0 0.0.0.0 10.0.2.1
PC4(config)#int g0/1
PC4(config-if)#ip address 10.0.2.104 255.255.255.0
```

そして異なるネットワーク間の通信にはルーターを使いますので、
VLAN1とVLAN2に繋がるルーターを設置します。

```
PC1#ping 10.0.2.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.2.103, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/8/12 ms

PC1#traceroute 10.0.2.103
Type escape sequence to abort.
Tracing the route to 10.0.2.103
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.1 6 msec 6 msec 7 msec
  2 10.0.2.103 11 msec 12 msec *
```

S1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0000.0000.0001    DYNAMIC     Gi0/1
   1    0000.0000.0002    DYNAMIC     Gi0/2
   1    fa16.3e26.e3ca    DYNAMIC     Gi1/1
   2    0000.0000.0003    DYNAMIC     Gi0/3
   2    0000.0000.0004    DYNAMIC     Gi1/0
   2    fa16.3e7d.9c6d    DYNAMIC     Gi1/2


## ルーターのサブインターフェース

トランクを使うことで、VLANの数だけスイッチとスイッチをケーブルで繋ぐ必要がなくなりました。
同じことがルーターとの接続でも言えます。
先ほど紹介したVLAN1とVLAN2をルーターでルーティングさせる手法では、
VLANの数だけルーターのポートを接続しました。
ルーターでもトランク及びVLANタグを使うことで、1つのポートで複数のネットワークに接続することができます。
それには「サブインターフェース」と呼ばれる仕組みを使います。

<<図>>

インターフェースに対して「このタグが付いたパケットはこのサブインターフェースで扱う」と設定すれば、
同じ物理インターフェースに対して届いたパケットであっても、
パケットのタグに応じて利用する「仮想的なインターフェースであるサブインターフェース」が変わります。




```
R1(config)#int g0/1
R1(config-if)# no ip address
R1(config-if)# int g0/1.1
R1(config-subif)#encapsulation dot1Q 1
R1(config-subif)#ip address 10.0.2.1 255.255.255.0
R1(config)#int g0/1.2
R1(config-subif)#encapsulation dot1Q 2
R1(config-subif)#ip address 10.0.2.1 255.255.255.0
R1(config)#int g0/1.3
R1(config-subif)#encapsulation dot1Q 3
R1(config-subif)#ip address 10.0.3.1 255.255.255.0
```


R1#show ip int bri
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.255.0.219    YES NVRAM  administratively down down    
GigabitEthernet0/1         10.0.1.1        YES NVRAM  up                    up      
GigabitEthernet0/1.2       10.0.2.1        YES manual up                    up      
GigabitEthernet0/1.3       10.0.3.1        YES manual up                    up      


なお、ルーターはスイッチとは異なるのでVLANという概念はありません。
ただ、IPアドレスやルーティングの制御といった管理面で捉えると、
サブインターフェースとVLAN番号とマッピングをしたほうがわかりやすいです。
そのため、私はサブインターフェースの番号はタグの番号、つまりVLANの番号を振るようにしています。
上記で言えば、VLAN1に対応するサブインターフェースは「X.1」、VLAN2であれば「X.2」となります。
同様にVLAN100であれば「X.100」を与えます。



## SVIとVLAN間ルーティング

ここからはL3スイッチについて扱います。
今まで学んできた構成では「VLAN間の通信(ネットワークを跨ぐL3転送)」をルーターを介して行ってきましたが、
L3スイッチを使うことでこれをスイッチ単独で実現することができます。

L3スイッチの仕組みを一言で言ってしまうと、「VLANに仮想的なインターフェースを作成し、
そのインターフェース間でルーティングができるスイッチ」と言えます。
分かりにくいかもしれませんので図を使って説明します。

<<図>>

ルーターでネットワークを跨ぐパケット転送(ルーティング)をするには、
そのルーターのインターフェースに対してパケットを届ける必要がありました。
第3章でお話したように、それにはPCにデフォルトゲートウェイを指定したり、
ルーター間で手動もしくは自動(OSPFなどのルーティングプロトコル)で行うことで実現してきました。
そのような仕組みで届けられたパケットを、ルーターはパケットの宛先IPを確認して転送します。
要するにルーティングをするには「IPを持ったインターフェース」が必要なわけです。

今まで学んできたようにスイッチにはVLANというL2の世界はあるものの、
IPというL3の世界はありませんでした。
そのため、VLAN間でパケットを転送するには外にルーターを置いて、
そのVLAN(L2)が転送に使うインターフェースを用意していました。
ただ、スイッチのVLANの世界にIPがないから外にルーターが必要なのであれば、
VLANの世界にIPを持ち込んでしまえばいいのす。
そのようにしてスイッチのVLANごとに「SVI」という仮想的なインターフェースが作成され、
そのSVIを使ってVLAN間でルーティングができるようになりました。

それでは実際にL3スイッチにSVIを作成して、
VLAN間ルーティングを実現してみます。
はじめにSVI作成前はルーティングができないことを念のために確認しておきます。

L3スイッチであるL3SW1にはIPを持つインターフェースはなく、
VLAN1のPC1(10.0.1.101/24)から、VLAN2のPC3(10.0.2.103)へのpingは失敗します。

```
L3SW1#show ip int bri
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  administratively down down    
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      

PC1#ping 10.0.2.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.2.103, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

L3スイッチにSVIを作成します。
SVIの作成は「interface vlan <VLAN番号>」とします。
こうするとそのVLANにSVIが作成され、インターフェース設定モードに移行します。
すでにSVIが作られていた場合はインターフェース設定モードに移るだけです。

```
L3SW1(config)#int vlan 1
L3SW1(config-if)#ip address 10.0.1.1 255.255.255.0
L3SW1(config-if)#no shut
L3SW1(config-if)#int vlan 2
L3SW1(config-if)#ip add 10.0.2.1 255.255.255.0
L3SW1(config-if)#no shut
```

SVIを作成し、それにルーターのインターフェースと同じようにIPアドレスを与えました。
「show ip interface brief」でポート一覧を確認すると、Vlan1, Vlan2という新しいインターフェースが作成されてアップしていることが分かります。
設定したIPアドレスも与えられています。

```
L3SW1#show ip int bri
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  administratively down down    
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      
Vlan1                  10.0.1.1        YES manual up                    up      
Vlan2                  10.0.2.1        YES manual up                    up    
```

これだけでSVIの作成は完了です。
VLAN1のPC1から、VLAN2のPC3にpingをすると成功するようになりました。

```
PC1#ping 10.0.2.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.2.103, timeout is 2 seconds:
.!!!!
```

この結果から分かるように、L3スイッチはL2スイッチのように同じネットワーク内のMACアドレスを使ったL2通信機能も提供し、
ルーターのように異なるネットワーク間でのパケットのルーティング機能も提供します。

これは非常に便利なのですが、L2の世界とL3の世界がどう構成されているか、
物理的に把握しづらいという問題が新しく発生するようになりました。
L3スイッチを上手に使うコツは「規則正しい設計」によって、L2の世界とL3の世界を綺麗に分けることです。
詳細は最終章のネットワークのデザインにてお話いたしますが、
多数あるVLANのそれぞれの範囲を分かりやすくして、ルーティングするL3スイッチを1箇所(冗長化されます)に統一するのが基本です。

## SVIでルーティングプロトコルを使う

SVIはルーターのインターフェースと同じように利用できますので、
第三章でお話したルーティングプロトコルを利用することもできます。
まだお話していない「ルーテッドポート」も絡めて使い方を確認してみます。

以下の図の構成を作成します。

<<図>>
