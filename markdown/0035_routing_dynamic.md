# ルーティングプロトコルを使った経路学習

## スタティックルートの問題点

小中規模のオフィスで数台のルーターを使うような場合は、スタティックルートで経路制御をできます。
ただ、ネットワーク全体の規模が大きくなってきた場合に手動で全てを管理するには限界があります。

例えば100台のルーターが複雑に接続されていて、
1000個のネットワークがそこに存在する環境を想像してください。
全てのルーターを手動で設定することは非常に労力がかかり、
なおかつネットワークの変更や追加作業は非常に複雑なものとなります。

このような問題を解決するために「**ルーティングプロトコル**」が使われます。
ネットワーク上にあるルーターに適切なルーティングプロトコルの設定をすると、
ルーターのあいだで会話をしてルーティングテーブルをダイナミックに構築します。
そのようにすることでネットワーク全体が勝手に正しくパケットを転送できるようになります。

なお、ルーティングプロトコルを使った自動的な経路学習と、
スタティックルートを使った人手による経路設定は同時に使うことができます。
一部のルートだけスタティックルートで設定し、
それ以外はルーティングプロトコルで自動学習させるといった使い方ができます。


## ルーティングプロトコルの種類

ネットワークの経路を学習するルーティングプロトコルにはいくつかの種類があります。
それぞれ動作方法や利用目的などが異なっていますが、
大きく分類すると「組織内で使われるプロトコル」と「組織間(つまりインターネット)で使われるプロトコル」があります。

組織内で使われるプロトコルとは、
ようするにある会社や団体(学校など)の内部ネットワークとして使われるプロトコルということです。
その組織内にIT管理者(外注含む)がいて、その人達が責任を持って管理できます。

一方、組織間で使われるプロトコルはA社とB社の間といった管理者が異なる複数のネットワーク間で使われるプロトコルということです。
両方の組織の管理者の間で「こうしましょうね」という取り決めを結ぶことが必須で、
それにしたがってお互いに機器を設定して組織間でのパケット転送をするための経路情報を交換しあいます。

前者のプロトコルとしてはOSPFが有名で、他にはRIPやEIGRP及びISISなどがあります。
後者のプロトコルとしてはBGPしか使われていません。

![image](./0010_image/01.png)

BGPを利用するのはサービスプロバイダーやキャリアと呼ばれる「別の組織にネットワークを提供する組織」がほとんどです。
一部の多国籍企業や大企業などでも利用されることがありますが、通常は上記の会社が提供する回線を使うだけですので規模こそ違うものの、
家のネットワークがサービスプロバイダーを使うのと全く方式をとっています。

一般的な企業に勤めるネットワークエンジニアであれば、OSPFさえ理解していればほとんどの組織では社内ネットワークは問題なく構築できます。
かつてCisco社で働いていた際に数十数百の企業ネットワークを見てきましたが、
RIPやEIGRP及びISISの利用率はOSPFに比べると圧倒的に低かったです。

もしサービスプロバイダーやキャリアで働いているのであれば、OSPFに加えてBGPを学習すべきかもしれません。
また先進的なネットワークを構築している会社であれば、社内でもBGP(MPBGP)を使用している場合があります。
ただ、BGPはOSPFに比べるとかなり複雑ですので、基礎的なネットワークの知識が身についた後に学んだほうがよいと思います。


## OSPFを使った経路学習

### トポロジ

以下のようなネットワークを題材にしてOSPFの仕組みについて説明し、
その後で実際に機器に設定を加えて、どのようなルーティングテーブルが構築されるか確認します。

本ページはルーティングプロトコルがどのようなものであるかを知ってもらうことを目標としています。
その題材として最も普及しているOSPFを使いますが、OSPFの詳細をきちんと理解してもらうことは目標としていません。
OSPFの詳細については別ページをご参照ください。

中心にいるルーターがコアとなって、それに他のルーターがぶらさがっているという構成です。
これに冗長化を加えれば、よく採用される企業のネットワークの形になります。

![image](./0010_image/01.png)

### OSPFの動作説明(簡易版)

ルーティングプロトコルを使ったルーティングテーブルの構築は、ルーター同士の伝言ゲームです。
プロトコルというルールにしたがって、お互いにどういうような経路情報を持っているかを教え合い、
それらの情報からそれぞれの機器がルーティングテーブルを構築します。

そのため、おおまかには以下の流れでルーティングテーブルを構築しています。

1. 同一ネットワークに属するルーター同士でネイバー関係(お互いに情報を教え合う)を結ぶ
2. ルーターでアドバタイズするインターフェースを選択する
3. 指定したインターフェースのネットワークアドレスを隣接するルーターにアドバタイズ
4. アドバタイズを受け取った機器はその経路が最短であれば採用し、隣接する他の機器にさらにアドバタイズする(最短経路でなければアドバタイズしない)
5. 上記を繰り返すことでネットワーク全体で経路情報を構築する

上記のステップ2の「最短経路」は、あるネットワークにたどり着くための経路が複数ある場合の一番近い経路です。
どういったルートを短いと判断するかはルーティングプロトコル次第で、例えばRIPですと間にあるルーターの台数が基準となります。
OSPFはRIPより賢く、間のルーターの数ではなくネットワークの帯域(100Mbps, 1Gbpsなど)と、ルーターの台数の両方を基準にします。
特別な状況でない限りルーターの台数が増えても太い帯域の経路をOSPFは選びますが、その経路の選択には厳密なアルゴリズムがあります。

ただ、一般的にはOSPFを利用するルーターは規則的に配置するため、
OSPFの経路選択アルゴリズムを知らなくてもどのようなルートを通って通信するか簡単に推測できる場合がほとんどです。
ルーターとルーターを不規則に接続するようなネットワークでもOSPFによるルーティングは正しく動くでしょうが、
そのようなネットワークはデザインとして間違っているので構築しないでください。


### OSPFの設定

それでは上記図に記載したとおりにルーターに設置を加えていきます。

```text
R1(config)#router ospf 1
R1(config-router)#

R1(config-router)#int g0/1
R1(config-if)#ip ospf 1 area 0
R1(config-if)#int g0/2
R1(config-if)#ip ospf 1 area 0
```

```text
R1#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/2        1     0               10.0.99.1/24       1     DR    2/2
Gi0/1        1     0               10.0.1.1/24        1     DR    0/0

R1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.99.2         1   FULL/BDR        00:00:34    10.0.99.2       GigabitEthernet0/2
10.0.99.3         1   FULL/DROTHER    00:00:31    10.0.99.3       GigabitEthernet0/2
```

### OSPFによる経路学習


```
R2(config)#router ospf 1
R2(config-router)#int g0/1
R2(config-if)#ip ospf 1 area 0
R2(config-if)#int g0/2
R2(config-if)#ip ospf 1 area 0
R2(config-if)#
*Oct 29 01:00:17.295: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.99.1 on GigabitEthernet0/2 from LOADING to FULL, Loading Done
R2(config-if)#end
```

```
R1(config-if)#
*Oct 29 01:00:07.677: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.99.2 on GigabitEthernet0/2 from LOADING to FULL, Loading Done
R1(config-if)#end

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

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        10.0.1.0/24 is directly connected, GigabitEthernet0/1
L        10.0.1.1/32 is directly connected, GigabitEthernet0/1
O        10.0.2.0/24 [110/2] via 10.0.99.2, 00:00:23, GigabitEthernet0/2
C        10.0.99.0/24 is directly connected, GigabitEthernet0/2
L        10.0.99.1/32 is directly connected, GigabitEthernet0/2
```


```
R3(config-if)#int g0/1
R3(config-if)#ip ospf 1 area 0
R3(config-if)#int g0/2
R3(config-if)#ip ospf 1 area 0
R3(config-if)#int g0/3
R3(config-if)#ip ospf 1 area 0
R3(config-if)#
*Oct 29 01:08:37.725: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.99.1 on GigabitEthernet0/3 from LOADING to FULL, Loading Done
*Oct 29 01:08:37.725: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.99.2 on GigabitEthernet0/3 from LOADING to FULL, Loading Done
```

```
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
O        10.0.2.0/24 [110/2] via 10.0.99.2, 00:09:01, GigabitEthernet0/2
O        10.0.3.0/24 [110/2] via 10.0.99.3, 00:00:37, GigabitEthernet0/2
O        10.0.4.0/24 [110/2] via 10.0.99.3, 00:00:37, GigabitEthernet0/2
C        10.0.99.0/24 is directly connected, GigabitEthernet0/2
L        10.0.99.1/32 is directly connected, GigabitEthernet0/2

R1#show ip route ospf
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
O        10.0.2.0/24 [110/2] via 10.0.99.2, 00:31:06, GigabitEthernet0/2
O        10.0.3.0/24 [110/2] via 10.0.99.3, 00:22:42, GigabitEthernet0/2
O        10.0.4.0/24 [110/2] via 10.0.99.3, 00:22:42, GigabitEthernet0/2
```
