

以上がデフォルトゲートウェイの仕組みでした。
次のセクションでは、このデフォルトゲートウェイの仕組みを複雑にしたルーティングの仕組みを説明します。


インターネットは複数のネットワークから構成されています。
ブロードバンドルーターの仕事は「自宅内のネットワーク」と「外の全てのネットワーク」とのやりとりを中継することでした。
つまりブロードバンドルーターは2つのネットワークを中継する役割を持っています。

業務用のルーターもブロードバンドルータと同じくネットワークとネットワークを繋ぐ仕事を担当しています。
ただ、家庭内のシンプルなネットワークを提供するブロードバンドルーターと違い、
たくさんのネットワークを繋げて通信させる複雑な機能を多く持っています。
本章ではルーターが提供するパケットを転送する仕組みに絞って解説をします。

![image](./0025_image/01.png)

スイッチは同じネットワーク内の機器同士でパケットを転送するために、MACアドレスを利用していました。
宛先MACアドレスとスイッチのインターフェスが対応付けられています。

一方、ルーターはIPアドレスのネットワークアドレスを基準にしてパケットを転送します。
第一章でもお話したようにIPアドレスはネットワーク自体を表す「ネットワークアドレス」と、
ネットワーク内のホストを表す「ホストアドレス」があります。
一般的にIPアドレスというとホストアドレスのことをさしますが、
これに「どこまでがネットワークアドレスか」を表す「サブネットマスク」を適用するとネットワークアドレスが得られます。

複数のネットワークが存在する環境では複数のルーターがスイッチを介して接続されているか、
ルーター同士が直接接続されていますので、自分が接続されているネットワーク宛のパケット以外を受け取った場合は、
お互いにどのルーターにパケットを送るか事前に決められています。
そのルールに従って複数のルーター間でパケットを転送しあうことで、パケットを宛先のネットワークまで送ります。


# ルーティングの仕組み

デフォルトゲートウェイを使った他のネットワークへのパケット転送は最も簡単なルーティングの仕組みです。
ルーターはこれをもう少し複雑にした仕組みにしたがってパケットを転送しています。

例えば以下のようなネットワークがあるとしましょう。

<<図>>

PC1 -- Switch -- Router --- Router --- Switch -- PC2
                                   --- PC3
                            Router --- PC4

PC1からPC2への通信と、PC3への通信は利用される経路が違います。
PC1からRouter1へはデフォルトゲートウェイとして転送されるだけですが、
Router1から最終的な宛先ネットワークに送るためには「どのルーターに転送するか」ということを知っている必要があります。
たとえばPC2が属するネットワーク「」に送るためにはRouter2に転送し、
PC3が属するネットワーク「」もRouter2に転送、PC4が属するネットワークはRouter3に転送という具合です。
ルーターもPCのデフォルトゲートウェイと同じように宛先が分からない場合に送るデフォルトの宛先を設定することができ、
ここではそれをインターネットに接続されているRouter0としています。

PC1>ping 10.0.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/6 ms
PC1>ping 10.0.99.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.99.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/4/7 ms
PC1>ping 10.0.99.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.99.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
PC1>ping 10.0.2.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.2.102, timeout is 2 seconds:
U.U.U
Success rate is 0 percent (0/5)




R1#conf t
R1(config)#ip route 10.0.2.0 255.255.255.0 10.0.99.2
R1(config)#ip route 10.0.3.0 255.255.255.0 10.0.99.3
R1(config)#ip route 10.0.4.0 255.255.255.0 10.0.99.3


R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 7 subnets, 2 masks
C        10.0.1.0/24 is directly connected, GigabitEthernet0/1
L        10.0.1.1/32 is directly connected, GigabitEthernet0/1
S        10.0.2.0/24 [1/0] via 10.0.99.2
S        10.0.3.0/24 [1/0] via 10.0.99.3
S        10.0.4.0/24 [1/0] via 10.0.99.3
C        10.0.99.0/24 is directly connected, GigabitEthernet0/2
L        10.0.99.1/32 is directly connected, GigabitEthernet0/2



R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#ip route 10.0.1.0 255.255.255.0 10.0.99.1
R2(config)#ip route 10.0.3.0 255.255.255.0 10.0.99.3
R2(config)#ip route 10.0.4.0 255.255.255.0 10.0.99.3
R2(config)#end
R2#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 7 subnets, 2 masks
S        10.0.1.0/24 [1/0] via 10.0.99.1
C        10.0.2.0/24 is directly connected, GigabitEthernet0/1
L        10.0.2.1/32 is directly connected, GigabitEthernet0/1
S        10.0.3.0/24 [1/0] via 10.0.99.3
S        10.0.4.0/24 [1/0] via 10.0.99.3
C        10.0.99.0/24 is directly connected, GigabitEthernet0/2
L        10.0.99.2/32 is directly connected, GigabitEthernet0/2



R3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#ip route 10.0.1.0 255.255.255.0 10.0.99.1
R3(config)#ip route 10.0.2.0 255.255.255.0 10.0.99.2
R3(config)#end
R3#
R3#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
S        10.0.1.0/24 [1/0] via 10.0.99.1
S        10.0.2.0/24 [1/0] via 10.0.99.2
C        10.0.3.0/24 is directly connected, GigabitEthernet0/1
L        10.0.3.1/32 is directly connected, GigabitEthernet0/1
C        10.0.4.0/24 is directly connected, GigabitEthernet0/2
L        10.0.4.1/32 is directly connected, GigabitEthernet0/2
C        10.0.99.0/24 is directly connected, GigabitEthernet0/3
L        10.0.99.3/32 is directly connected, GigabitEthernet0/3


PC1>ping 10.0.99.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.99.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/9/15 ms
PC1>ping 10.0.2.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.2.102, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 14/16/20 ms
PC1>ping 10.0.2.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.2.103, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/14/25 ms
PC1>ping 10.0.3.104
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.3.104, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/11/16 ms
PC1>ping 10.0.4.105
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.4.105, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 15/26/42 ms



R1#show ip arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.0.1.1                -   fa16.3eb5.d36a  ARPA   GigabitEthernet0/1
Internet  10.0.1.101             12   fa16.3ea2.6334  ARPA   GigabitEthernet0/1
Internet  10.0.99.1               -   fa16.3e1d.4a85  ARPA   GigabitEthernet0/2
Internet  10.0.99.2               4   fa16.3ed0.b70e  ARPA   GigabitEthernet0/2
Internet  10.0.99.3               6   fa16.3eba.e89c  ARPA   GigabitEthernet0/2


S1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    fa16.3e1d.4a85    DYNAMIC     Gi0/1
   1    fa16.3eba.e89c    DYNAMIC     Gi0/3
   1    fa16.3ed0.b70e    DYNAMIC     Gi0/2


この「ネットワークXへの通信を宛先X」に転送するという情報を管理しているのが「ルーティングテーブル」です。
MACアドレステーブルが宛先MACと出力ポートを管理していたように、
ルーティングテーブルは「宛先ネットワークと"転送するIPもしくはインターフェース"」をテーブルとして管理しています。

宛先ネットワークが自分が接続されるネットワークであれば、そのインターフェースからパケットを送ります。
もし直接接続されていないネットワークであれば、転送するIPに対してパケットを送ります。
ルーターからルーターに転送することを繰り返して、最終的に宛先のネットワークまでパケットが届けられます。
上記のネットワークでルーティングテーブルがどのようになっているか、以下の図に記載します。

<<図>>

なお、ルーティングテーブルに記載されていないネットワーク宛のパケットはルーターが破棄します。
スイッチが宛先を知らないパケットを全てのポートから拡散したのは、
1つのネットワークという狭い範囲の中で宛先がどこかのポートの先にあるかもしれないからです。
ルーターでも接続されるルーター全てにパケットを拡散すれば宛先に届くかもしれませんが、
そのようなことをすると宛先不明のパケットが世界中で繋がるルーター間で無限に拡散しあうことになります。
そのため、ルーターでのパケット転送においては宛先不明のパケットは破棄すると決められています。
ただ、実際のネットワークではデフォルトルート(宛先不明の際に転送するアドレス)が設定されますので、
組織内のネットワークのみルーティングテーブルで管理し、それにない宛先はインターネットに対して転送するのが一般的です。
このあたりの話は第七章の「インターネットの仕組み」にて説明します。

ルーティングテーブルの仕組みをおおまかに分かって頂けたかと思いますので、
ここからは実際に機器を操作してネットワークを構築し、機器がどのようにルーティングテーブルを管理しているか実際に確認します。

ルーティングテーブルの構築は大まかに2つあり、
管理者が手動で設定するものと、ルーティングプロトコルを使ってルーター同士で互いに教え合うというものです。
ルーティングプロトコルは次のセクションで扱うため、ここでは手動で設定を行う「スタティックルート」という手法を使います。
スタティックルートの設定コマンドは「ip route XXXX YYYY ZZZZ」というもので、
これをすると XXXX のネットワークアドレス(サブネットマスクがYYYY)宛の通信はZZZZに送るという
ルーティングテーブルのルール(エントリ)を作れます。
なお、自分が直接接続されているルーティングテーブルのエントリはインターフェースにIPアドレスを与えた時点で勝手に作成されますので、
コマンドでルーティングテーブルにエントリを追加する必要はありません。

先ほどの図を参考にRouter1,2,3にそれぞれ以下の設定を与えます。

```
```

実際の機器のルーティングテーブルを確認してみます。

```
```

このようにしてルーターに対してパケットをどのように転送するかというルールを定義できます。

なお、ルーティングテーブルのことをRIBやFIBと呼ぶことがありますが、
RIBはソフトウェアが持つテーブルで、FIBはソフトウェアで作ったRIBをハードウェアに落とし込んだテーブルとなります。
現在の機器は高速にパケット転送を行うために様々な仕組みを使っているため、
ルーティングテーブルもかなり複雑な仕組みで利用されています。
ただ、その仕組は使う機器のベンダや型番によって異なりますので、普通のネットワーク管理者は気を配る必要はありません。





重要なのはL2転送に使われるMACアドレスはネットワークごとに変わるのに対して、
L3転送に使われるIPアドレスは送信元から宛先まで常に同じことです。
MACアドレスはL2転送に使われて、IPアドレスがL3転送に使われるという仕組みがあるため当然といえば当然ですが、
きちんとネットワークのことを勉強した人以外はあまり認識していないような気がします。


R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.0.1.0/24 is directly connected, GigabitEthernet0/1
L        10.0.1.1/32 is directly connected, GigabitEthernet0/1
C        10.0.2.0/24 is directly connected, GigabitEthernet0/2
L        10.0.2.1/32 is directly connected, GigabitEthernet0/2
