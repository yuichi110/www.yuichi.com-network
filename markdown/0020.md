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

## ネットワークの仮想化

今まで学んできた構成はスイッチに接続されたネットワーク機器が一つのネットワークを作りました。
これはつまり作るネットワークの数以上のスイッチが必要ということです。

第二章でお話したように1つのネットワークは必要以上に大きくすることが望ましくないため、
適切なサイズに分離する必要があります。
ただ、ネットワークの数だけ機器をそろえると金銭面、管理面のコストが大幅に増大してしまいます。

そのため、1つのネットワーク機器でいくつものネットワークを扱う様々な技術が利用され、
その代表的なものがVLAN(L2)とVRF(L3)です。
L3スイッチは実質的にVLANの仕様が必須ですので、まずはVLANの仕組みと使い方について説明をしましょう。

## VLAN

### なぜVLANが必要か

第二章はMACアドレスを使ったパケット転送の仕組みについて集中的にお伝えしたかったので扱いませんでしたが、
スイッチにはVLAN(Virtual LAN)と呼ばれる重要な機能があります。

その名前から分かるようにVLANは1台のスイッチでLANを仮想的に分割する技術です。
家庭用のスイッチを思い浮かべて頂くと分かりますが、全てのポートが同じグループに属していると、
そこに繋がる機器は全てお互いに通信ができます。
家庭内であればそれで問題ないでしょうが、企業のネットワークだとお客さんがアクセスできるネットワークと
社員向けのネットワークを分離する必要などがあります。

<<図>>

上記の構成では1つのスイッチ(実際は冗長化のために同じ設定がされたスイッチ2台以上を使います)が、
お客さん向け、社内向け、社外向けサーバー(例えばウェブサーバー)、
社内向けサーバー(例えばファイル共有サーバーやActive Directoryなど)とネットワークを区切っています。
ネットワークが区切られているため、お客さんのノートPCが社内ファイルサーバーにアクセスすることを防げます。
これには後の章で話すセキュリティ関係の話が絡みますので、その際に詳細を扱います。

上記のように1台のスイッチに複数の別個のネットワークを扱わせるのではなく、
ネットワークごとに物理スイッチを買うという選択肢もあります。
ただ、ネットワークが増えれば増えるほど機器の台数が増えて金銭コストもかかりますし、管理しきれなくなります。
そのため、ある規模以上の社内インフラを作る場合はVLANが利用されることが一般的です。

### VLANの仕組み

第二章で学んだL2転送の仕組みを思い出して下さい。
L2転送において最も重要だったのは「MACアドレステーブル」を使った「宛先MACアドレスと出力ポートのマッピング」でした。
VLANはこのMACアドレステーブルを複数に分割し、各MACテーブルに利用するポートを結びつけます。

たとえば1つめのネットワークとなるVLAN1は、VLAN1のMACアドレステーブルとそれを使うポート群を持っています。
そして2つめのネットワークとなるVLAN2は、VLAN2のMACアドレステーブルとそれを使うポート群を持っています。
VLAN1のネットワークとVLAN2のネットワークは異なるため、通信は完全に分離されています。
たとえばVLAN1の機器がARP Requestを出しても、それはVLAN1のポートにのみフラッドされて、
VLAN2にフラッドされることはありません。
VLAN2の機器のARPもこれと同様です。

<<図>>

上記の構成ではスイッチのG0/1とG0/2がVLAN1に割り当てられており、
G0/3とG1/0がVLAN2に割り当てられています。
G0/1に接続されるPC1は同じVLAN1に属するPC2には通信をできますが、
PC3とPC4はVLANが異なるため通信できません。
VLAN1とVLAN2はネットワークが異なるため、
本来であれば「10.0.1.0/24」「10.0.2.0/24」といった異なるネットワークアドレスを使うべきですが、
「同じネットワークでなければARPが出されない」という仕組みがあるため、
あえて同じレンジにしています。

ここまでの話を実際に機器を操作しながら確認してみます。
まずVLANを使うためにはVLANを作る必要があります。
これには「vlan <VLAN番号コマンド>」を使います。

```
S1(config)# vlan 2
S1(config-vlan)# end
```

存在するVLANを確認するためには「show vlan」コマンドを使います。
このコマンドはどのVLANにどのポートが属しているかも表示してくれます。

```
S1# show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                Gi1/0
2    VLAN0002                         active    
```

現在はVLAN1とVLAN2が存在することが分かります。
VLAN1は最初から存在し、VLAN2は先ほど作りました。
そしてVLAN1にはポートG0/0(VIRLの管理ポート)からG1/0までが属していることが分かります。

先の構成を作るために、G0/3とG1/0の2つのポートをVLAN2に加えます。
これにはインターフェース設定モードにて「switchport access vlan <VLAN番号>」を使います。
今回ですと以下のようになります。

```
S1(config)#int g0/3
S1(config-if)#switchport access vlan 2
S1(config-if)#int g1/0
S1(config-if)#switchport access vlan 2
```

あらためて「show vlan」コマンドでVLANの設定を確認します。

```
S1#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1, Gi0/2
2    VLAN0002                         active    Gi0/3, Gi1/0
```

G0/1とG0/2がVLAN1に属し、G0/3とG1/0がVLAN2に属するという構成を作れました。
PC同士に通信をさせて、VLAN1とVLAN2のネットワークが分離されていることを確認します。

```
PC1#ping 10.0.1.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.102, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 5/5/6 ms
```

PC1とPC2は同じL2ネットワークであるVLAN1に属するため、pingが成功します。
一方、PC1とPC3は異なるVLANであるため、pingが失敗します。

```
PC1#ping 10.0.1.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.103, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

PC3からPC2への通信もVLANが異なるためpingに失敗しますが、
PC3からPC4へは同じVLANであるためpingが成功します。

```
PC3#ping 10.0.1.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.102, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
PC3#ping 10.0.1.104
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.104, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 3/3/4 ms
```

第二章で使った「show mac address-table」コマンドでMACテーブルを確認すると、
MACアドレスとポートの情報だけでなく、VLAN番号も記載があることが分かります。

```
S1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0000.0000.0101    DYNAMIC     Gi0/1
   1    0000.0000.0102    DYNAMIC     Gi0/2
   2    0000.0000.0103    DYNAMIC     Gi0/3
   2    0000.0000.0104    DYNAMIC     Gi1/0
Total Mac Addresses for this criterion: 4
```

VLAN1のポートから来たフレームの転送にはVLAN1のMACテーブルのみが利用され、
VLAN2は全く関係がありません。

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


## アクセスポートとトランクポート

小規模なネットワークはスイッチ1台で実現できますが、
大きなネットワークを作るためにはVLANを設定した複数のスイッチを接続します。
その際、どのネットワークにどのVLAN番号を使うかは、必ず接続される全てのスイッチで統一してください。
たとえば「10.0.1.0/24」をスイッチAでVLAN1にし、スイッチBではVLAN2にするといったことはしないでください。
技術的には可能ですが管理が大変になるだけでメリットがありません。

当然ですが、上位のスイッチと下位のスイッチを繋ぐリンクもVLANを意識する必要があります。
スイッチとスイッチがVLAN1のポートでのみ繋がれていれば、
VLAN2のネットワークの通信はスイッチを跨って行き来することができません。
存在するVLANの数だけポートとケーブルを利用するというのも一つの手法ですが、
VLANが100個存在すれば100本も繋ぐ必要があるため現実的ではありません。

「トランク(Trunk)」という仕組みを使うことで、この問題を解決できます。
トランクはスイッチとスイッチを繋ぐポートで複数のVLANを流すために用いられ、
スイッチAとスイッチBを繋ぐポートを「トランクポート」として設定すると、VLAN1もVLAN2もそのポートを使います。
なお、今まで利用してきた1つのVLANしか流さないポートは「アクセスポート」と呼ばれています。

<<図>>

ただ、同じリンクで複数のVLANのフレームをやりとりするということは、
そのフレームがどのVLANのものかを識別できる目印のようなものが必要だということです。
スイッチはフレームをトランクポートに転送する際に「タグ」と呼ばれる目印を付けることで、
それがどのフレームのものかを識別しています。
また、「タグがない」というのも1つのVLANを識別する手がかりになるため、
必要に応じて「このVLANのフレームはトランクポートにタグなしとする」と設定することができます。
このトランクポートでのタグを使わないVLANのことを「ネイティブ VLAN (native vlan)」と呼びます。
そして、ネイティブVLANでないVLANは全てタグがついていますので、「タグVLAN Tag VLAN」と呼ばれます。
トランクのタグがどのように実現されているかは第六章(TCP/IP)にて扱います。

アクセスポートとトランクポートがどのようなものか分かって頂けたかと思いますので、
実際に以下のようなネットワークを構築します。

<<図>>

スイッチのポートのモード(アクセスorトランク)を設定するには「switchport mode <access|trunk>」コマンドを使います。
デフォルトはアクセスポートとなっています。

```
S1(config-vlan)#int g0/3
S1(config-if)#switchport trunk encapsulation dot1q
S1(config-if)#switchport mode trunk
```

上記の例ではトランクポートに設定する前に、
トランクを実現する手法(タグのつけかた)を「dot1q」として設定しています。
dot1q以外の手法も使えますが、完全に廃れていますのでdot1qを指定してください。

トランクポートを確認するには「show interface trunk」コマンドを使います。
これを使うとトランクポートの一覧と、それぞれのポートがどのVLANを通しているのかが分かります。

```
S1#show int trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/3       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/3       1-4094

Port        Vlans allowed and active in management domain
Gi0/3       1-2

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/3       1-2
```

注意をしてほしいのは「show vlan」コマンドではトランクポートが表示されないことです。
これにはアクセスポートしか表示されません。

```
S1#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1
2    VLAN0002                         active    Gi0/2
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0   
2    enet  100002     1500  -      -      -        -    -        0      0   
1002 fddi  101002     1500  -      -      -        -    -        0      0   
1003 tr    101003     1500  -      -      -        -    -        0      0   
1004 fdnet 101004     1500  -      -      -        ieee -        0      0   
1005 trnet 101005     1500  -      -      -        ibm  -        0      0   

Remote SPAN VLANs
------------------------------------------------------------------------------


Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
```

スイッチ2にも同様の設定を加えて、スイッチ3のポートもトランクに変更します。
1つのポートずつトランクの設定をしても良いのですが、3つのポートに同じ設定をいれるため、
「range」を使って複数のインターフェースを同時に指定してみます。

```
S3(config)#int range g0/1-3
S3(config-if-range)#switchport trunk encapsulation dot1q
S3(config-if-range)#switchport mode trunk
```

rangeに続けて複数のインターフェースを指定でき、上記はg0/1からg0/3を指定しています。
このコマンドを発行した後でトランクインターフェースの一覧を確認すると、
確かにg0/1からg0/3までが、まとめてトランクの設定をされていることが分かります。

```
S3#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      1
Gi0/2       on               802.1q         trunking      1
Gi0/3       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/1       1-4094
Gi0/2       1-4094
Gi0/3       1-4094

Port        Vlans allowed and active in management domain
Gi0/1       1-3
Gi0/2       1-3
Gi0/3       1-3

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/1       1-3
Gi0/2       1-3
Gi0/3       none
```

この設定が完了したら、3台のスイッチをまたがってVLANが繋がります。
同じVLANのPC同士でpingができることを確認してください。

最後に次のセクションのためにSW3のルーターに接続するポートG0/3で、
ネイティブVLANを変更します。
デフォルトのネイティブVLANはVLAN1であるため、VLAN1のフレームにはタグがついていません。
ただ、G0/3ではデフォルトVLANを全く利用しないVLAN99に割り当てます。
こうすることで、VLAN1にもタグがつけられるようになります。

```
switchport trunk native vlan 99
```

<<図>>

注意をして欲しいのはスイッチとスイッチを接続する場合は、
ネイティブVLANを必ず揃える必要があるということです。
たとえばスイッチAとスイッチBがつながっており、スイッチAのポートのネイティブVLANが10で、
スイッチBのネイティブVLANが20だとします。
このときスイッチAがVLAN10のフレームをスイッチBに転送すると、そのフレームはVLAN20に変化してしまいます。
繋がるべき通信ができなかったり、繋がるべきでない箇所が繋がるため、必ず揃えて下さい。

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

## 通過させるVLANの制御

VLANの制御をする際に重要になるのが、どのVLANを通すかということです。
適切に設定されておらずVLANが分断されてしまったり、
不必要にスイッチ間でVLANを繋げすぎて無駄に大きいネットワークを作ることはよくありません。

VLANを扱ううえでまず知っておくべきことは、あるVLAN-Xがスイッチで作成されていないと、
そのVLAN-Xのフレームは全て破棄するということです。
例えば以下のような構成だと、下位のスイッチにはVLAN-2があるものの、
それらを繋ぐ上位のスイッチにVLAN-2が存在しないため、フレームを届けることができません。

<<図>>

上位のスイッチでは下位のスイッチが使っている全てのVLANを作成しておくようにしてください。
試しにSW3でVLANを消してみると、スイッチを跨ぐ通信が届かなくなることが確認できるはずです。
なお、VLAN1は消せません。

フレームを届けられないほど大きな問題ではありませんが、
不必要にフレームを届けすぎてしまうことも問題となります。
例えば以下の図を見て下さい。

<<図>>


「アラウドVLAN(Allowed VLAN)」をトランクポートに設定することで、
VLANの通信を届けすぎてしまうことを防げます。
許可する(Allow)という名前から分かるように、
アラウドVLANとして設定されたVLANのフレームだけをそのポートで送受信するようになります。
デフォルトは全てのVLANを許可しています。

アラウドVLANの設定方法は複数あり、全てを列挙したり、これ以外と指定したり、今ある設定にこれを追加などとできます。
細かい設定方法は本質にはそれほど重要ではないため、
ここでは全てを列挙する「switchport trunk allowed vlan <VLAN番号群>」を使います。

```
S3(config)#int g0/1
S3(config-if)#switchport trunk allowed vlan 1,2  
```

この設定をすると、g0/1のインターフェースではVLAN1,2しかやりとりしないようになります。
それ以外の全てのVLAN宛のフレームが届いた場合は破棄しますし、
そのポートからVLAN1,2以外のフレームを出さないようになります。

アラウドVLANの設定も「show interface trunk」で確認できます。

```
S3#show int trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      1
Gi0/2       on               802.1q         trunking      1
Gi0/3       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/1       1-2
Gi0/2       1-4094
Gi0/3       1-4094

Port        Vlans allowed and active in management domain
Gi0/1       1-2
Gi0/2       1-3
Gi0/3       1-3

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/1       1-2
Gi0/2       1-3
Gi0/3       1-3
```

オフィスネットワークですと、ネイティブVLANとアラウドVLANの活用場面はあまり多くありません。
ただ、サーバールームやデータセンターなどではよく利用されています。
コラムに記載しましたので興味があるかたはご一読ください。


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



## VRF

VLANはL2の分割技術。
L3の分割技術はなにか。VRF。

VRFはルーティングテーブルを分割する
VLANに比べて利用される場面が少ない。
複雑になるので必要以上に使わないこと。

## コラム


### サーバー仮想化とvSwitch

サーバーへはトランク
