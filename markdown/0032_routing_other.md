
# 経路集約

ルーティングプロトコルを使うことでルーター同士がお互いに経路情報を交換するため、
簡単に複雑なネットワークでパケット転送がうまくいく状態にできます。
ただ、そのようなネットワークの設計は間違っているため、通常は「経路集約」を念頭においたネットワークの設計をします。

経路集約は「連続した小さな複数のネットワーク」を「大きな一つのネットワーク」として外に見せるテクニックです。
これを使うことでルーティングテーブルに記載されるエントリの数が減らせるので、ネットワークを管理しやすくなります。
ただ、「連続した」とあるようにネットワークの設計をする際にレンジが近いネットワークアドレスを固める必要があります。

具体例として東京に本社を持ち、日本全国に支店を持つ会社でOSPFを使って経路情報を交換するというシナリオを考えます。
社内アドレスとして10.0.0.0/8を利用します。

10.0.0.0/8を切り出して、/24のネットワークを必要になったタイミングで各拠点に割りあて、
足りなくなったら追加で増設するというルールでネットワークの運用をしていたとしましょう。
すると以下のような社内ネットワークが構築されることになります。

<<図>>

このようなネットワークを構築すると、それぞれの拠点が持つ全ての/24のネットワークをお互いに教え合う必要があります。

一方、各拠点に/16のネットワークを事前に割り当てて、それを拠点内で分割するというルールでネットワークを運用したとします。

<<図>>

このようなネットワークを作ると経路を集約することができます。
例えば大阪にあるPCが東京にあるサーバーに通信をする場合、
大阪にあるルーターは宛先アドレスを見て「10.1.0.0/16」のアドレスであれば、
とりあえず東京に送って後は東京のルーターに任せるということができます。


PC_OKNW1#ping 10.1.1.101
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.101, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/16/25 ms
PC_OKNW1#ping 10.1.2.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.2.102, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/17/25 ms
PC_OKNW1#ping 10.2.1.101
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.1.101, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 10/13/19 ms
PC_OKNW1#ping 10.2.2.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.2.102, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/13/16 ms


R_TKY1#show ip route
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

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
C        10.0.0.0/24 is directly connected, GigabitEthernet0/1
L        10.0.0.1/32 is directly connected, GigabitEthernet0/1
C        10.1.0.0/24 is directly connected, GigabitEthernet0/2
L        10.1.0.1/32 is directly connected, GigabitEthernet0/2
O        10.1.1.0/24 [110/2] via 10.1.0.2, 00:26:48, GigabitEthernet0/2
O        10.1.2.0/24 [110/2] via 10.1.0.3, 00:27:16, GigabitEthernet0/2
O IA     10.2.0.0/24 [110/2] via 10.0.0.2, 00:19:00, GigabitEthernet0/1
O IA     10.2.1.0/24 [110/3] via 10.0.0.2, 00:14:24, GigabitEthernet0/1
O IA     10.2.2.0/24 [110/3] via 10.0.0.2, 00:14:14, GigabitEthernet0/1
O IA     10.3.0.0/24 [110/2] via 10.0.0.3, 00:09:59, GigabitEthernet0/1


R_OKNW1#show ip route
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

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
C        10.0.0.0/24 is directly connected, GigabitEthernet0/1
L        10.0.0.3/32 is directly connected, GigabitEthernet0/1
O IA     10.1.0.0/24 [110/2] via 10.0.0.1, 00:08:32, GigabitEthernet0/1
O IA     10.1.1.0/24 [110/3] via 10.0.0.1, 00:08:32, GigabitEthernet0/1
O IA     10.1.2.0/24 [110/3] via 10.0.0.1, 00:08:32, GigabitEthernet0/1
O IA     10.2.0.0/24 [110/2] via 10.0.0.2, 00:08:32, GigabitEthernet0/1
O IA     10.2.1.0/24 [110/3] via 10.0.0.2, 00:08:32, GigabitEthernet0/1
O IA     10.2.2.0/24 [110/3] via 10.0.0.2, 00:08:32, GigabitEthernet0/1
C        10.3.0.0/24 is directly connected, GigabitEthernet0/2
L        10.3.0.1/32 is directly connected, GigabitEthernet0/2


R_TKY2#show ip route
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

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
O IA     10.0.0.0/24 [110/2] via 10.1.0.1, 00:29:33, GigabitEthernet0/1
C        10.1.0.0/24 is directly connected, GigabitEthernet0/1
L        10.1.0.2/32 is directly connected, GigabitEthernet0/1
C        10.1.1.0/24 is directly connected, GigabitEthernet0/2
L        10.1.1.1/32 is directly connected, GigabitEthernet0/2
O        10.1.2.0/24 [110/2] via 10.1.0.3, 00:29:33, GigabitEthernet0/1
O IA     10.2.0.0/24 [110/3] via 10.1.0.1, 00:21:49, GigabitEthernet0/1
O IA     10.2.1.0/24 [110/4] via 10.1.0.1, 00:17:12, GigabitEthernet0/1
O IA     10.2.2.0/24 [110/4] via 10.1.0.1, 00:17:01, GigabitEthernet0/1
O IA     10.3.0.0/24 [110/3] via 10.1.0.1, 00:12:46, GigabitEthernet0/1



PC_OKNW1#ping 10.1.1.101
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.101, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 25/37/52 ms
PC_OKNW1#ping 10.1.2.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.2.102, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 11/17/31 ms
PC_OKNW1#ping 10.2.1.101  
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.1.101, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 15/27/53 ms
PC_OKNW1#ping 10.2.2.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.2.102, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/11/14 ms


R_OKNW1#show ip route
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

      10.0.0.0/8 is variably subnetted, 7 subnets, 3 masks
C        10.0.0.0/24 is directly connected, GigabitEthernet0/1
L        10.0.0.3/32 is directly connected, GigabitEthernet0/1
O IA     10.1.0.0/16 [110/2] via 10.0.0.1, 00:20:36, GigabitEthernet0/1
O IA     10.2.0.0/16 [110/2] via 10.0.0.2, 00:17:17, GigabitEthernet0/1
O        10.3.0.0/16 is a summary, 00:20:36, Null0
C        10.3.0.0/24 is directly connected, GigabitEthernet0/2
L        10.3.0.1/32 is directly connected, GigabitEthernet0/2


R_TKY1#show ip route     
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

      10.0.0.0/8 is variably subnetted, 9 subnets, 3 masks
C        10.0.0.0/24 is directly connected, GigabitEthernet0/1
L        10.0.0.1/32 is directly connected, GigabitEthernet0/1
O        10.1.0.0/16 is a summary, 00:06:53, Null0
C        10.1.0.0/24 is directly connected, GigabitEthernet0/2
L        10.1.0.1/32 is directly connected, GigabitEthernet0/2
O        10.1.1.0/24 [110/2] via 10.1.0.2, 00:06:03, GigabitEthernet0/2
O        10.1.2.0/24 [110/2] via 10.1.0.3, 00:06:16, GigabitEthernet0/2
O IA     10.2.0.0/16 [110/2] via 10.0.0.2, 00:06:16, GigabitEthernet0/1
O IA     10.3.0.0/16 [110/2] via 10.0.0.3, 00:06:16, GigabitEthernet0/1


R_TKY2#show ip route
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

      10.0.0.0/8 is variably subnetted, 8 subnets, 3 masks
O IA     10.0.0.0/24 [110/2] via 10.1.0.1, 00:05:31, GigabitEthernet0/1
C        10.1.0.0/24 is directly connected, GigabitEthernet0/1
L        10.1.0.2/32 is directly connected, GigabitEthernet0/1
C        10.1.1.0/24 is directly connected, GigabitEthernet0/2
L        10.1.1.1/32 is directly connected, GigabitEthernet0/2
O        10.1.2.0/24 [110/2] via 10.1.0.3, 00:05:31, GigabitEthernet0/1
O IA     10.2.0.0/16 [110/3] via 10.1.0.1, 00:05:31, GigabitEthernet0/1
O IA     10.3.0.0/16 [110/3] via 10.1.0.1, 00:05:31, GigabitEthernet0/1

# 経路選択アルゴリズム

ルーターが学習する経路情報

ルーティングテーブルを見ただけでどのようにルーティングされるか分かるようなネットワークを作るべきですが、


 - プレフィックス長
 - AD値
 - メトリック


R1#show ip route 10.2.0.0
Routing entry for 10.2.0.0/24
 Known via "ospf 1", distance 110, metric 3, type inter area
 Last update from 10.0.4.4 on GigabitEthernet0/3, 00:00:49 ago
 Routing Descriptor Blocks:
 * 10.0.4.4, from 10.2.0.1, 00:00:49 ago, via GigabitEthernet0/3
     Route metric is 3, traffic share count is 1
R1#
R1#
R1#show ip route 10.2.0.0
Routing entry for 10.2.0.0/24
 Known via "ospf 1", distance 110, metric 4, type inter area
 Last update from 10.0.1.2 on GigabitEthernet0/2, 00:02:59 ago
 Routing Descriptor Blocks:
 * 10.0.1.2, from 10.2.0.1, 00:02:59 ago, via GigabitEthernet0/2
     Route metric is 4, traffic share count is 1

aa

R1#show ip route 10.2.0.0
Routing entry for 10.2.0.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.0.4.4
      Route metric is 0, traffic share count is 1




R1#show ip route 10.2.0.0
Routing entry for 10.2.0.0/24
  Known via "ospf 1", distance 110, metric 4, type inter area
  Last update from 10.0.1.2 on GigabitEthernet0/2, 00:25:25 ago
  Routing Descriptor Blocks:
  * 10.0.1.2, from 10.2.0.1, 00:25:25 ago, via GigabitEthernet0/2
      Route metric is 4, traffic share count is 1

ip route を追加

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

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
C        10.0.1.0/24 is directly connected, GigabitEthernet0/2
L        10.0.1.1/32 is directly connected, GigabitEthernet0/2
O        10.0.2.0/24 [110/2] via 10.0.1.2, 01:31:52, GigabitEthernet0/2
O        10.0.3.0/24 [110/3] via 10.0.1.2, 01:31:32, GigabitEthernet0/2
C        10.0.4.0/24 is directly connected, GigabitEthernet0/3
L        10.0.4.1/32 is directly connected, GigabitEthernet0/3
O        10.0.5.0/24 [110/4] via 10.0.1.2, 01:12:53, GigabitEthernet0/2
C        10.1.0.0/24 is directly connected, GigabitEthernet0/1
L        10.1.0.1/32 is directly connected, GigabitEthernet0/1
S        10.2.0.0/24 [1/0] via 10.0.4.4

ip route を/16変更

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

      10.0.0.0/8 is variably subnetted, 11 subnets, 3 masks
C        10.0.1.0/24 is directly connected, GigabitEthernet0/2
L        10.0.1.1/32 is directly connected, GigabitEthernet0/2
O        10.0.2.0/24 [110/2] via 10.0.1.2, 01:44:53, GigabitEthernet0/2
O        10.0.3.0/24 [110/3] via 10.0.1.2, 01:44:33, GigabitEthernet0/2
C        10.0.4.0/24 is directly connected, GigabitEthernet0/3
L        10.0.4.1/32 is directly connected, GigabitEthernet0/3
O        10.0.5.0/24 [110/4] via 10.0.1.2, 01:25:54, GigabitEthernet0/2
C        10.1.0.0/24 is directly connected, GigabitEthernet0/1
L        10.1.0.1/32 is directly connected, GigabitEthernet0/1
S        10.2.0.0/16 [1/0] via 10.0.4.4
O IA     10.2.0.0/24 [110/4] via 10.0.1.2, 00:10:27, GigabitEthernet0/2       

PC1#traceroute 10.2.0.102
Type escape sequence to abort.
Tracing the route to 10.2.0.102
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.0.1 4 msec 4 msec 3 msec
  2 10.0.1.2 5 msec 6 msec 7 msec
  3 10.0.2.3 9 msec 12 msec 8 msec
  4 10.0.3.5 8 msec 16 msec 7 msec
  5 10.2.0.102 8 msec 16 msec *
PC1#traceroute 10.2.0.102
Type escape sequence to abort.
Tracing the route to 10.2.0.102
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.0.1 3 msec 3 msec 3 msec
  2 10.0.4.4 5 msec 3 msec 8 msec
  3 10.0.5.5 6 msec 8 msec 10 msec
  4 10.2.0.102 12 msec 8 msec *
PC1#

PC1#traceroute 10.2.0.102
Type escape sequence to abort.
Tracing the route to 10.2.0.102
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.0.1 8 msec 4 msec 2 msec
  2 10.0.4.4 7 msec 6 msec 5 msec
  3 10.0.5.5 7 msec 6 msec 11 msec
  4 10.2.0.102 8 msec 20 msec *
PC1#
PC1#
PC1#traceroute 10.2.0.102
Type escape sequence to abort.
Tracing the route to 10.2.0.102
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.0.1 3 msec 5 msec 3 msec
  2 10.0.1.2 2 msec 5 msec 3 msec
  3 10.0.2.3 12 msec 5 msec 6 msec
  4 10.0.3.5 7 msec 4 msec 4 msec
  5 10.2.0.102 5 msec 9 msec *


# コラム

ルーターとスイッチのソフトウェアとハードウェア
RIBとFIB
転送用のASICとバックプレーン側のASIC




PCとルーターは同じネットワークに接続されているため、パケットをルーターに送るには第二章で説明した手順が使われます。
復習がてらに説明すると以下の流れとなります。

1. ARPでデフォルトゲートウェイのMACアドレスを問い合わせ(キャッシュされている場合は3へ)
2. デフォルトゲートウェイであるルーターがMACアドレスを通知
3. 宛先MACアドレスをデフォルトゲートウェイのMACに設定し、パケットを転送
4. パケットを受け取ったルーターは宛先に応じてそのパケットをさらに転送

ルーターから他のルーターに転送する場合も同様にARPを使って、
次の宛先となるルータのMACアドレスを取得して同じネットワーク内の転送をします。
ルーターから「パケットを届ける宛先の最終地点であるホスト」に転送する場合もARPとL2転送が使われます。

ルーターの転送の仕組みは追って話しますが、ネットワークA,B,Cという構成でネットワークAにいるホストから、
ネットワークCにいるホストに通信をする場合は以下のようなパケットになります。

<<図>>


## ルート再配送
