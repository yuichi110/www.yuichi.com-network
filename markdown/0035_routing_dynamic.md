# ルーティングプロトコルを使った経路学習

## 概要


## スタティックルートとダイナミックルート

### スタティックルートの欠点

小中規模のオフィスで数台のルーターを使うような場合は、
スタティックルートでルーティングを制御することでネットワーク間の通信を制御をできます。

ただ、大きな組織で多数のネットワークが複雑に構成される場合は、
スタティックルートを使って人の手で経路情報を制御することは困難な場合が多いです。
例えば100台のルーターが複雑に接続されていて、
1000個のネットワークがそこに存在する環境を想像してください。
全てのルーターを手動で設定することは非常に労力がかかり、
なおかつネットワークの変更や追加作業は複雑なものとなることが分かります。

人間のオペレーションにはミスが伴いますので、
複雑になればなるほど人手による設定は信頼できなくなります。


### ルーティングプロトコルを使ったダイナミックルート

スタティックルートの問題点を解決するために「**ルーティングプロトコル**」が使われます。
ルーティングプロトコルはルーターの間で会話をするための言語のようなもので、
隣り合う機器同士で「私はネットワークAに接続されています」「私はネットワークBに接続されています」といった情報をやりとりし、
その情報にもとづいて「ネットワークAにパケットを送るにはルーターR1にパケットを転送する」といったルーティングテーブルを自動で構築します。

これはちょうど、ルーティングプロトコルがネットワークがどのようにネットワークが接続されているかという地図を作り、
それにもとづいてルーティングテーブルという道路標識を自動で作成するようなものです。
人間がトポロジ図を見てスタティックルートで自分で道路標識を作る代わりに、
ルーティングプロトコルが地図(トポロジ図)を作成して、道路標識を自動で作り上げます。

![image](./0010_image/01.png)

複雑なことをしないのであればルーティングプロトコルの設定は単純ですので、
多数の経路情報を多数の機器に自分でスタティックルートで設定するよりも労力がかかりません。
また、ルーティングプロトコルが間違った動作をすることは多くないため、
ルーティングプロトコルが作る経路情報は人間が設定したものよりも信頼することができます。

ルーティングプロトコルを使って学習した経路情報は「**ダイナミックルート**」と呼ばれており、
状況に応じてその経路情報は変化します。
一方、人の手で設定されたスタティックルートは設定を上書きしない限り残り続けます。

ダイナミックルートの情報が変化するということは、
宛先ネットワークに辿り着くためのルートが状況に応じて適切なものに修正されるということです。
途中にあるルーターやスイッチが故障して今までの経路が使えなくなれば、
その経路のエントリが消されて代わりとなる経路があればそれが登録されるといったこともおきます。
スタティックルートであれば相手の状態に関係なくパケットを転送し続けますので、
途中の経路で問題などが起きていても、その経路を使い続けようとします。

なお、ルーティングプロトコルを使った自動的な経路学習と、
スタティックルートを使った人手による経路設定は同時に使うことができます。
よく見られるのが東京オフィスと大阪オフィスの間のような拠点間でのルーティングにはスタティックルートを使い、
各拠点の内部ではダイナミックルートを使うといった構成です。


### ルーティングプロトコルの種類

ルーティングプロトコルにはいくつかの種類があります。
それぞれのプロトコルごとに動作方法や利用目的などが異なっていますが、
大きく分類すると「組織内で使われるプロトコル」と「組織間(つまりインターネット)で使われるプロトコル」の2種類があります。

組織内で使われるプロトコルとは、
ようするにある会社や団体(学校など)の内部ネットワークとして使われるプロトコルということです。
その組織内にIT管理者(外注含む)がいて、その人達が責任を持って管理できます。
最も標準的なものが「**OSPF**」と呼ばれるプロトコルで、
他には「**RIP**」「**EIGRP**」「**ISIS**」といったものがあります。

一方、組織間で使われるプロトコルはA社とB社の間といった管理者が異なる複数のネットワーク間で使われるプロトコルということです。
両方の組織の管理者の間で「こうしましょうね」という取り決めを結ぶことが必須で、
それにしたがってお互いに機器を設定して組織間でのパケット転送をするための経路情報を交換しあいます。
組織間のやりとりに使われるプロトコルは「**BGP**」のみです。

BGPを利用するのはサービスプロバイダーやキャリアと呼ばれる「別の組織にネットワークを提供する組織」がほとんどです。
一部の多国籍企業や大企業などでもBGPを利用することがありますが、
通常はサービスプロバイダーやキャリアが提供するインターネットの回線を使うだけですので、
自分でBGPを使って他社と接続するということはしません。

![image](./0010_image/01.png)

つまり、サービスプロバイダーなどを除く一般的な企業に勤めるネットワークエンジニアであれば、
OSPFさえ理解していればネットワークは問題なく構築できる場合がほとんどです。
仮にサービスプロバイダーやキャリアで働いている場合であっても、
OSPFはBGPに比べてかんたんであるため、
まずはOSPFで経路情報の交換の仕組みを学ぶのがよいでしょう。


## OSPFを使った経路学習

### トポロジ

以下のようなネットワークを題材にしてOSPFの仕組みについて説明し、
その後で実際に機器に設定を加えて、どのようなルーティングテーブルが構築されるか確認します。

![image](./0010_image/01.png)

中心にいるルーターがコアとなって、それに他のルーターがぶらさがっているという構成です。
これに冗長化(壊れてもネットワークが切れない)を加えれば、よく採用される企業のネットワークの形になります。

なお、本ページはルーティングプロトコルがどのようなものであるかを知ってもらうことを目標としています。
その題材として最も普及しているOSPFを使いますが、
その仕組については最も重要な事項を除いて割愛します。


### OSPFの動作説明(簡易版)

細かな仕組みはそれぞれのルーティングプロトコルに依存していますが、
ルーティングプロトコルを使ったルーティングテーブルの構築はルーター間の伝言で実現されています。
たとえば「ネットワークAはこっちにあるよ」「私はネットワークBに接続しています」といった情報を、
バケツリレーのように多数のルーターで共有して各ルーターがルーティングテーブルを構築しています。

OSPFの場合は同一ネットワークにいる他のルーターと「**ネイバー関係**」を結び、
ネイバー間でどういうネットワークに接続されているかといった情報を交換します。
各ルーターは複数のインターフェースを持つので、
ネイバー関係を直接結んでいないルーターの情報もネイバー関係のルーターから伝言というかたちで伝えられます。

![image](./0010_image/01.png)

各ルーターが伝え合うそれぞれのルーターが接続しているネットワークの情報は、
組織内のネットワーク全体の断片情報です。
この断片情報はどれとどれが繋がっているかという情報も持っているため、
それらを全て正しくつなぎ合わせることで組織内のネットワーク構造の全体図を作ることができます。

この全体図は全てのルーターが同じものを持つので、
あるネットワークに辿り着くためのルーターAが考える転送方法とルーターBが考える転送方法は同じです。
それぞれが自分が果たすべき転送の役割をルーティンテーブルのエントリという形に落とし込むことで、
全てのルーターが連携してパケットを宛先ネットワークまで届けることができるようになります。

OSPFのおおまかな仕組みは上記のようなものですが、
これに加えて経路情報を交換するグループである「**エリア**」や、
経路集約といった仕組みがあります。


### OSPFの設定

構成図にもとづいてルーターにOSPFの設置を加えていきます。

![image](./0010_image/01.png)

OSPFを使うには、ルーターでOSPFのプロセスを作成することから始めます。
ルーターのプロセス作成はグローバル設定モードで「**route ospf <プロセス番号>**」とします。


```text
R1(config)#router ospf 1
R1(config-router)#exit
```

1つのルーター上で複数のOSPFプロセスを走らせることができるためプロセス番号の指定がありますが、
通常は1つのプロセスしか走らせません。

このコマンドを発行することでOSPFの設定モードに移行して、
細かいOSPF設定などができます。
ただ、今回は特に設定しないため、すぐに抜けています。

次は各インターフェース上でのOSPFの有効化をします。
OSPFのプロセスを作成しても、インターフェースで有効化しなければOSPFの管理対象に入りません。
インターフェースでOSPFを有効化するには「**ip ospf <プロセス番号> area <エリア番号>**」というコマンドを使います。
プロセス番号は先ほど作成したものを使い、エリア番号はエリアを分けないのであれば0を指定します。

```text
R1(config)#int g0/1
R1(config-if)#ip ospf 1 area 0
R1(config-if)#int g0/2
R1(config-if)#ip ospf 1 area 0
```

こうすることで同一ネットワークに属する他のOSPFが有効になったルーターと通信をして、
ネイバー関係を自動で結べるようになりました。
OSPFを有効にしたルーター同士のやりとりにはマルチキャストが利用されます。
そのため、同一ネットワークで2台以上のルーターでOSPFが有効になっていれば、
相手のIPなどを入力しなくても勝手に通信しあってネイバー関係を結びます。

他のルーターR2及びR3を設定して、R1のOSPFネイバーをshowコマンドで確認します。
ネイバーの確認には「**show ip ospf neighbor**」コマンドを使います。

```text
R1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.99.2         1   FULL/BDR        00:00:34    10.0.99.2       GigabitEthernet0/2
10.0.99.3         1   FULL/DROTHER    00:00:31    10.0.99.3       GigabitEthernet0/2
```

Neighborとして隣接しているR2及びR3が表示されています。
Neighborがうまく形成できない場合は「**show ip ospf interface brief**」などで、
きちんとインターフェースがOSPFの対象に含まれているか確認をしてください。

```text
R1#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/2        1     0               10.0.99.1/24       1     DR    2/2
Gi0/1        1     0               10.0.1.1/24        1     DR    0/0
```

ネイバーが結ばれれば、ルーター間でネットワークの情報のやりとりが開始されます。


### OSPFによる経路学習

```text
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
