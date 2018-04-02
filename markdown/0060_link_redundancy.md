# リンクの冗長化

## 概要


## 冗長化の必要性

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


## ロードバランス


## LACP

ポートチャネルにプロトコルを用いることも可能です。
このプロトコルの役目は「適切に分散をさせる」ことですので、
物理リンク上を流れるトラフィック自体をどうこうするというわけではなく、
不健全なリンクを使うことによる障害を防ぐという目的で使われています。

チャネルで使うプロトコルには「**LACP(Link Aggregation Control Protocol)**」と「PAgP」があり、両者とも用途は同じです。
Cisco独自のPAgPを標準化したものがLACPであり、LACPが現在の主流です。
特別な理由がない限りはPAgPではなくLACPを使うようにしてください。

LACP


他には利用するプロトコルも考慮する必要があります。
一般的にはチャネル用のプロトコルを何も使わない「スタティック」と呼ばれる設定にするか、
接続しているリンクが間違っていないか、相手が生きているかを確かめる「LACP」と呼ばれるプロトコルを使う設定にするかのどちらかです。


Wifiを使うノートPCやタブレットといったユーザー機器を除けば、ネットワーク上の機器と機器は基本的にケーブルで接続されています。
みなさんの自宅にあるようなLANケーブルも一般的ですし、
10Gやそれ以上のスピードが求められる環境では専用の銅線ケーブル(Copper)や光ケーブル(Fiber)を使う場合もあります。

(私の体感的には中長距離を接続する光ケーブル用の光信号をやりとりする着脱式のSFPと呼ばれるモジュールが壊れる確率が若干高いです。)


この問題を予防するために、複数のケーブルを論理的に一本に束ねる技術があり、
それが「LAG(Link agligation)」や「Port Channel」「Ether Channel」と呼ばれるものです。
使う機材によって呼び方が変わりますが、一般的にはサーバーではLAG、ネットワーク機器ではチャネルと呼ばれることが多いようです。
本書ではチャネルで統一します。
チャネルに参加するポートのことをメンバーポートと呼びます。

<<図>>

実はたんに複数本のリンクでスイッチ同士を接続するだけでも後述するスパニングツリーで冗長化が実現されるのですが、
チャネルを使うと複数本のリンクを一本の論理的なリンクにするためスイッチ間の帯域が広がります。
そのため、スイッチ間のリンクではチャネルが多用されますし、サーバーへの接続にもチャネルが利用されます。



以下の構成で実際にチャネルの設定をします。

<<図>>

チャネルの設定をする際は、念のためにインターフェースをダウンさせてください。
先にお伝えしたように両方のスイッチが同じ設定を持つことが望ましいため、




今回のスイッチはインターフェースに何も設定されていないので、
そのままrangeを使って複数インターフェースを指定して、channel-groupコマンドを発行します。
g0/1とg0/2をチャネル番号1とし、スタティック設定(mode on)にします。
g0/3とg1/0をチャネル番号2とし、これをLACPの設定(mode active)にします。

```
L2SW3(config)#int range g0/1-2            
L2SW3(config-if-range)#channel-group 1 mode on
Creating a port-channel interface Port-channel 1

LACPの設定
L2SW3(config)#int range g0/3,g1/0
L2SW3(config-if-range)#channel-group 2 mode active
Creating a port-channel interface Port-channel 2
```

上記のメッセージにもあるように、新しく「Port-Channel1 (Po1)」と「Port-Channel2 (Po2)」が作成されました。

```
L2SW3#show int status

Port      Name               Status       Vlan       Duplex  Speed Type
Gi0/0                        disabled     1            auto   auto RJ45
Gi0/1                        disabled     1            auto   auto RJ45
Gi0/2                        disabled     1            auto   auto RJ45
Gi0/3                        disabled     1            auto   auto RJ45
Gi1/0                        disabled     1            auto   auto RJ45
Po1                          notconnect   unassigned   auto   auto
Po2                          notconnect   unassigned   auto   auto
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
2      Po2(SD)         LACP      Gi0/3(D)    Gi1/0(D)    
```



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

マッピングの確認


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


ロードバランスの話が必要。
ハッシュアルゴリズムの計算
