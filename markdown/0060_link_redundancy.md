# リンクの冗長化

## 概要


## 冗長化の必要性

今までお話してきた第二章から第四章までの構成は、機器の故障を一切考慮していません。
試しに機器を「reload」コマンドで再起動してみると、再起動している間の通信は全く通らなくなります。
家のPCや家電が壊れるように、企業が使っているネットワーク機器も壊れますので、
「1つの機器が壊れたら社内ネットワークが使えなくなります」という構成は許容されません。

この一箇所壊れたら使えなくなる箇所は「**SPOF(Single Point Of Failure)**」と呼ばれており、
ネットワークの冗長化技術を使うことでSPOFの数を可能な限り減らすことが必要です。


ネットワーク機器のあいだはケーブルで結ばれています。
これらのケーブルやケーブルを挿すポートは、確率こそ高くないものの壊れることがあります。
自宅のネットワークであれば壊れても不便さに耐えて、
新しいケーブルや機器を買ってくるまでは我慢をすればなんとかなります。

ただ、企業や組織のネットワークは自宅のように壊れたら停止するというのは許容できません。
ネットワークが繋がらなくなることで会社のホームページに繋がらなくなれば顧客を逃す可能性がありますし、
社内向けの重要なサービス(たとえば端末にログインするための認証サーバー)が使えなければ業務が停止します。

企業のネットワークは「壊れてもネットワークを提供できる」ことが重要となるため、
壊れることを前提としてシステムの運用を維持し続けられる「**冗長化**」と呼ばれる仕組みを取り入れています。

冗長化にも様々な種類のものがありますが、
ネットワークでの冗長化は一般的には「リンクレベル(L1)の冗長化」「イーサネットレベル(L2)での冗長化」「ルーティングレベル(L3)での冗長化」の3つに加えて、
「個別のネットワーク製品がサポートする冗長化」が主なものです。
サーバー側の機器で冗長化する技術もありますが、ネットワークエンジニアの範疇ではないため、
概要さえ知っていれば通常は問題ありません。

また、信頼性がさほど必要のない検証環境では、とにかく安く抑えるためにスイッチなどのインフラを冗長化をしないという場合もあります。
商用環境ではないので落ちたところで不便以上の大きな影響はないため、壊れたらベンダーの保守契約で翌日交換などすれば問題なしという姿勢です。
コストと信頼性の優劣は資金や環境によって異なりますので、適切な冗長化レベルを選択する必要があります。

本ページではリンクレベルの冗長化技術について扱います。

## ポートチャネル

2つの機器が1本のケーブルで接続している際に、それが壊れると当然ながら2台が切り離されてしまいます。
無理な取り回しでケーブル自体が破損することもありますし、ケーブルをさすポートやSFPが初期故障や経年劣化で壊れることもあります。

一本のリンクにトラブルが発生して切り離される問題は、複数本のリンクを利用していれば回避することができます。
2つ以上の物理リンクを束ねて、仮想的に一本の論理リンクを作ることで、
そのうちの一本の物理リンクが壊れても2つの機器の間の接続を維持できます。

![image](./0025_image/01.png)

このような技術は「**ポートチャネル**」や「**LAG(Link AGgregation)**」と呼ばれており、
ネットワーク機器だけでなくサーバー機器でも利用されます。
一般的にはネットワーク機器でポートチャネルという用語を使い、サーバー機器でLAGという用語を使いますが、
両者に違いはありません。
チャネルやLAGは束ねられた論理リンクのことにも使われ、
それを構成する物理リンクは「**メンバーポート**」と呼ばれます。

チャネルを使うべきに考慮すべきことは接続する「両方の機器の設定をそろえる」ことです。
例えば接続するスイッチの片側だけチャネルの設定をして、もう片側はなにもしないということはありません。
両方のスイッチで統一されたチャネルの設定が必要です。

以下の構成で実際にチャネルの設定をします。

![image](./0025_image/01.png)

チャネルにLACPというプロトコルを使用することもできますが、
これについては後ほど扱います。

なお、リンクレベルの冗長化はネットワーク機器や重要なホストであるサーバーに対してのみ行います。
一般的なユーザーが使うPCやデスクトップはポートを複数持たないため、壊れれば交換が必要です。
ただ、現在だとユーザー用のPCもwifiとポートを持つことが多いため、
どちらかが壊れても、もう一方を使うことでネットワークに繋がることはできます。


### チャネルの設定

チャネルの設定をする際は、念のために先にインターフェースをダウンさせてください。
インターフェースを落とさずに片側ずつ順に設定をいれると、
チャネルの設定がされている側とされていない側が存在する状況に一時的になります。
両方の機器の設定が完了してからインターフェースをアップさせます。

```
L2SW3(config)#int range g0/0-3, g1/0
L2SW3(config-if-range)#shut
```

チャネルの設定をするには、チャネルのメンバーポートの設定を全て統一したうえで、
「channel-group <チャネル番号> mode <チャネルモード>」コマンドを発行します。

チャネル番号は論理ポートの番号で、好きな値を利用することができます。
同じチャネル番号が設定されている物理ポートが束ねられて論理ポートになります。
分かりやすさの観点からチャネルで繋がる両方の機器で番号を揃えることが多いです。

チャネルモードはチャネルにどのようなプロトコルを使うかという設定です。
今回はプロトコルを使わないので「on」という設定にします。
ひとつひとつのインターフェースに個別で設定をいれてもいいのですが、
rangeを使って複数インターフェースを指定すると手間がかかりません。

```
L2SW3(config)#int range g0/1-2            
L2SW3(config-if-range)#channel-group 1 mode on
Creating a port-channel interface Port-channel 1
```

メッセージにもあるように、このコマンドを発行すると論理インターフェースであるポートチャネルが作成されます。
すでに指定した論理インターフェースが作成されていた場合は、それに追加されます。

注意をしてほしいのはchanne-groupコマンドで束ねられる物理インターフェースは、
全て同一の設定を持っている必要があるということです。
たとえばあるインターフェースがアクセスポートとしてVLAN10に属していて、
別のインターフェースがアクセスポートとしてVLAN20に属している場合は、チャネルを組むことができません。
メンバーポートに余計なコマンドが入っていたら全てnoコマンドで消して初期化してしまい、
そのうえで上記のchannel-groupコマンドを発行してください。

「show interface status」コマンドで確認をすると、
新しく「Po1」というインターフェースが作成されていることがわかります。

```
L2SW3#show int status

Port      Name               Status       Vlan       Duplex  Speed Type
Gi0/0                        disabled     1            auto   auto RJ45
Gi0/1                        disabled     1            auto   auto RJ45
Gi0/2                        disabled     1            auto   auto RJ45
Gi0/3                        disabled     1            auto   auto RJ45
Gi1/0                        disabled     1            auto   auto RJ45
Po1                          notconnect   unassigned   auto   auto
```

どのチャネルにどの物理インターフェースが割り当てられているかは「show etherchannel summary」コマンドで確認できます。
割当に加えて、チャネル自体の状態(アップしているかダウンしているか)やプロトコル、メンバーごとの状態も把握できます。

```
L2SW3#show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)          -        Gi0/1(D)    Gi0/2(D)    
```

Po1の状態は「S(Switchport)でL2」「D(Down)」であることがわかり、
メンバーポートもDownとなっています。

対向側にも同様の設定をして、物理インターフェースをアップさせると、
ポートチャネルもアップします。

```
L2SW3(config)#int range g0/1-2            
L2SW3(config-if-range)#no shut
L2SW3(config-if-range)#end

L2SW3#show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)          -        Gi0/1(P)    Gi0/2(P)
```

ポートチャネルの状態が「U(USE)」になり、
メンバーポートも「P(bundled in port-channel)」となり利用されていることがわかります。

MACアドレステーブルは物理インターフェースではなくチャネル単位での学習になります。

```text
L2SW3#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0000.0000.0101    DYNAMIC     Po1
   1    0000.0000.0102    DYNAMIC     Po2
   1    fa16.3e28.b93e    DYNAMIC     Po1
   1    fa16.3e38.e7b9    DYNAMIC     Po2
   1    fa16.3e8c.f77e    DYNAMIC     Po2
   1    fa16.3e98.78ea    DYNAMIC     Po1
```

## ロードバランス

チャネルを組むことで機器間のリンクに冗長性を持たせることができるようになりますが、
それだけでなく利用可能なリンク帯域が物理リンクの合計値となります。
チャネルを異なる通信速度の物理リンクで束ねることは避けるため、
通常は「物理リンクの帯域 x 物理リンク数」で合計帯域を求めることができます。

チャネルを流れるトラフィックはメンバーポートのどれかを利用します。
これを「**ロードバランス**」といいます。
ロードバランスはトラフィックの「**フロー**」単位で行われます。
フローは「あるホストAのアプリケーションと別のホストBのアプリケーションの間の通信」のことで、
パケットとしては「イーサネットのアドレス、IPのアドレス、TCP/UDPのポート番号が全て同じ」という特徴があります。
基本的には「同じフローのパケットは同じメンバーポートを使う」と認識してもらえば問題ありません。

![image](./0025_image/01.png)

同じフローが複数のメンバーポートを使うということはないため、
1つのフローが物理リンク以上の帯域を使うことはできません。
ただ、ネットワークの広帯域な箇所を流れるパケットは、
ルーティングなどで様々なホストから集約されているため広帯域なのであり、
ひとつの巨大なフローで広帯域となっていることは通常の環境では発生しません。

ロードバランスでうまく通信をメンバーポートにばらけさせるには、
「**ハッシュアルゴリズム**」に気を配る必要があります。
ロードバランスでどのメンバーポートが選ばれるかは、
トラフィックのヘッダの値をハードウェアの「**ハッシュ関数**」にかけて、
その算出した値により決められています。
このハッシュ関数にどのヘッダを使うかを決めるのがハッシュアルゴリズムの重要なポイントとなります。

![image](./0025_image/01.png)

たとえば上記の図ではトラフィックがルーターAからルーターBへ流れています。
このような状況ではL2アドレスは全てのホストの通信で同じになっているため、
ハッシュアルゴリズムでL2アドレスのみを見ていればロードバランスがうまくされません。

機器がL2からL4までをハッシュに使えるのであれば、そうしてしまうのが無難です。
L2しかチェックしなければ異なるフローが同一のハッシュ値になり同一のメンバーポートによってしまいますが、
L4の値までをチェックすれば異なるフローは異なるハッシュ値となるため、
複数のメンバーポートにばらけてくれることが期待されます。


## LACP

ポートチャネルにプロトコルを用いることも可能です。
このプロトコルの役目は「適切に分散をさせる」ことですので、
物理リンク上を流れるトラフィック自体をどうこうするというわけではなく、
不健全なリンクを使うことによる障害を防ぐという目的で使われています。

チャネルで使うプロトコルには「**LACP(Link Aggregation Control Protocol)**」と「PAgP」があり、
両者とも用途は同じです。
Cisco独自のPAgPを標準化したものがLACPであり、現在はLACPの主流です。
特別な理由がない限りはPAgPではなくLACPを使うようにしてください。

LACPはプロトコルを使わないチャネル(mode on)の機能拡張といえます。
チャネルの両端でLACPプロトコルを使うことで、
リンクダウンにならないリンク不良(たとえばファイバーの片側断線)などでデータロストが発生することを防げます。

チャネルをLACPで組むにはチャネルを作成する際に「mode active」を指定します。
すでに作成されているLACPのチャネルに新しいメンバーを追加する際も同じです。

```text
L2SW3(config)#int range g0/3,g1/0
L2SW3(config-if-range)#channel-group 2 mode active
Creating a port-channel interface Port-channel 2
```

スタティックの場合と同じように「Port-Channel2 (Po2)」が作成されました。
「show etherchannel summary」コマンドでチャネルを確認すると、プロトコルがLACPとなっています。

```
L2SW3#show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)          -        Gi0/1(P)    Gi0/2(P)    
2      Po2(SU)         LACP      Gi0/3(P)    Gi1/0(P)    
```
