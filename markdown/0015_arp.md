# スイッチを使った同一ネットワーク内の通信

{{ TOC }}

## 概要


## 機器の構成図

先の説明では自宅用のブロードバンドルーターを題材にネットワークの仕組みについて学びました。
本章ではCiscoの機材を利用して、スイッチと同一ネットワーク内の通信の基礎についてしっかり学びたいと思います。

本章では以下のような構成を使います。

![image](./0010_image/01.png)

PCとしてCiscoのルーターを使いますが、パケットの転送などはさせずにホストとして利用します。
そのため、WindowsやMac、サーバーとなるLinuxなどと同じだと思っていただいて構いません。

なお、本章ではスイッチに対して設定はほとんど加えず家庭用のスイッチと同じように利用します。
業務用スイッチ特有の設定は第4章から扱います。


### Routerの設定

本章からはCiscoのルーターとスイッチに対して設定を加えて環境のセットアップを行います。
設定に関しては本書は必要最低限の説明に留めますので、詳細はCiscoのドキュメントや専門書籍を参照ください。

Ciscoの機器はUnixやLinuxなどと同じようにコマンド(CLI)で操作をします。
その際に重要になるのが「モード」という概念で、
コマンドを使ってモードを行き来することで機器の設定や状態を見たり、設定変更を加えたりします。
操作する機種によってCiscoのOSが異なるため、若干モードに違いがありますが、大差はありません。
ここでは一番標準的なIOSを題材にしてモードの説明をします。

![image](./0010_image/02.png)

上記の図に本書で利用するモードの状態遷移図を記載しましたので、機器の設定をする際はこれを意識してください。
まず、大まかに「一般ユーザモード」「管理者モード」「設定変更モード」の3つがあります。
一般ユーザモードは管理者ではないものの、機器の状態を見ることを許されている状態です。
管理者モードは全ての機器の状態(管理者ユーザには許可されていないものも含む)を確認でき、
なおかつ設定変更モードにも移れるモードです。
設定変更モードは名前の通り、機器の設定を変更できる状態です。
管理者が設定や状態を確認するときが「管理者モード」になり、設定変更する際に「設定変更モード」になります。

設定変更モードも細かく分類ができ、機器のホスト名といった全般的なものを変更できるのが「グローバル設定変更モード」です。
管理者モードから設定変更モードに移るとこれになります。
一方、インターフェースの設定は「グローバル設定変更モード」から「インターフェース設定変更モード」に移って行います。
グローバル設定変更モード以外の詳細な設定を変更するモードは多数あり、
インターフェースやルーティングプロトコルなどの管理単位で分かれています。

Cisco機器の操作に不慣れなかたもいると思いますので、
ここではPCとして使用するルータにIPを設定するという手順をステップごとに説明します。
本書はCiscoのネットワーク機器のシミュレーターであるVIRLを利用することを前提としていますが、
実機での操作も大差ありません。
VIRLのセットアップや基本的な操作方法については巻末の補足「Cisco VIRL」に記載しております。

まずCiscoのIOSの機器(ルーターとスイッチ)を初回起動すると、
設定が何もされていないのでダイアログ(質問形式)で設定をするかと聞かれます。
今回は手動で入力していきますので、「no」と回答し、エンターボタンで次に進みます。

```text
% Please answer 'yes' or 'no'.
Would you like to enter the initial configuration dialog? [yes/no]: no

Press RETURN to get started!
```

先に説明した一般ユーザーモードとしてログインされますので、
「enable」コマンドで管理者モードに移ります。
ユーザープロンプト「>」「#」で、一般ユーザモードか管理者モードかが区別できます。

```text
Router>enable
Router#
```

管理者モードで機器の設定や状態を確認する場合は「show」コマンドを使います。
showに続けて決められた単語を並べることで、目的に応じた内容を得られます。
showコマンドにも体系がありますが、とりあえず使うことを優先して細かな説明は省きます
「show ip interface brief」コマンドでルーターのインターフェースの一覧を得ます。

```text
Router#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES unset  administratively down down    
GigabitEthernet0/1         unassigned      YES unset  administratively down down    
```

インターフェースが2つあり、IPアドレスは設定されておらず、
状態が「administratively down(明示的にdownさせている)」となっています。
VIRLではG0/0のインターフェースは機器の管理用に作成されているため、
トポロジには表示されていません。
使いませんので本書では全てシャットダウンしておきます。

次にルーターの設定をします。
管理者モードから設定モードに移行するには「configure terminal」コマンドを入力します。
そうするとプロンプトが代わり、機器の設定ができるようになりました。
この状態では先ほどのshowコマンドは使えません。

ホスト名の変更は「hostname <ホスト名>」というコマンドを使います。
全ての機器の名前が同じだと区別がしにくいため、図と同じ名前であるPC1を設定します。
プロンプトに表示されていたホスト名が変更されました。

```text
Router# configure terminal
Router(config)#hostname PC1
PC1(config)#
```

次にこのルーターのインターフェースにIPアドレスを与えます。
「interface <設定したいインターフェース名>」とすることで、
設定モードから「インターフェース設定モード」に移ります。
そして「ip address <IPアドレス> <サブネットマスク>」とすることで、
ルーターのインターフェースにIPをふることができます。
こちらも図の通りに設定しています。
インターフェースが「admin shut」ですので「no shutdown」でインターフェースをアップさせます。

```text
R1(config)#interface GigabitEthernet0/1
R1(config-if)#ip address 10.0.1.101 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#
*Oct 25 00:14:33.986: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
*Oct 25 00:14:34.986: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
```

ルーターからインターフェースがアップしたという旨のメッセージが得られました。
設定が完了したので、設定モードから管理者モードに戻ります。
「exit」コマンドで設定モードから1階層戻るという動きをし、
「end」コマンドで一気に管理者モードまで戻るという動作をします。

```text
R1(config-if)#exit
R1(config)#end
R1#
*Oct 25 00:15:17.728: %SYS-5-CONFIG_I: Configured from console by console
```

再度、インターフェースの状態を「show ip interface brief」コマンドで確認します。
各コマンドで使われる単語は他に被る単語がない状態であれば、省略可能です。
そのため「show ip int bri」と入力すれば「show ip interface brief」として動きます。

```text
R1#show ip int bri
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES unset  administratively down down    
GigabitEthernet0/1         10.0.1.101      YES manual up                    up      
```

インターフェース G0/1 にIPアドレスが振られていることがわかりますね。
最後に「write」コマンドで設定をルーターに保存します。
様々なコマンドを打つことで設定はルーターやスイッチに適用されていきますが、
設定を保存しないと再起動をした際に更新した設定が全て消えています。

```text
R1#write
Building configuration...
[OK]
R1#
*Oct 25 00:16:41.001: %GRUB-5-CONFIG_WRITING: GRUB configuration is being updated on disk. Please wait...
*Oct 25 00:16:41.831: %GRUB-5-CONFIG_WRITTEN: GRUB configuration was written to disk successfully.
```

以上でルーターをPC1として使う設定が完了しました。
名前とIPを変えた同じ設定をPC2とPC3にも加えて下さい。

最後にスイッチの設定を変更します。
スイッチの設定もルーターとほとんど同じですので、スイッチ名を「S1」に変更します。

ルーターはインターフェースがデフォルトでAdmin Downになっていたのに対して、
スイッチはデフォルトでアップになっています。
先に申したように管理用のG0/0のインタフェースは使わないので、
これだけ明示的に「shutdown」コマンドでダウンさせておきます。

```text
Switch>en
Switch#show ip int bri
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up      
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      
GigabitEthernet0/3     unassigned      YES unset  up                    up  

Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname S1
S1(config)#int g0/0
S1(config-if)#shut
S1(config)#end
S1#show ip int bri
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  administratively down down    
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      
GigabitEthernet0/3     unassigned      YES unset  up                    up      

S1#write
```

以上で全ての機器の設定が完了しました。

### 通信をさせる

本章の冒頭で紹介した構成が作られ、3台のルーター(PCの代わりに使う)の設定が完了したとしましょう。
スイッチはほぼデフォルトのままです。

PC1(192.168.0.101)からPC2(192.168.0.102)へPingで疎通確認をすると、
「!」が表示されて問題なく応答が得られていることがわかります。
参考までに存在しないIPである192.168.0.104にPingをすると、期待通りに応答が得られないことがわかります。

```text
R1#ping 10.0.1.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.102, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 4/4/5 ms

R1#ping 10.0.1.104
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.104, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

市販の安いスイッチと同じように業務用の本格的なスイッチも、それに繋げた機器同士で通信できるようにします。
この仕組を深掘りしてみましょう。

### MACアドレスとMACアドレステーブル

あるホストAとあるホストBが通信する場合、IPアドレスを利用して相手を特定してパケットをやりとりします。
パケットは様々なホストで中継されますが、「ネットワークを跨ぐ通信」と「ネットワーク内の通信」のいずれかです。
ネットワークを跨ぐ通信は「IPアドレス」によってどこにどう送るかが決まりますが、
ネットワーク内の通信は「MACアドレス(Media Access Control address)」を使って制御されています。

つまりPC1からPC2への通信は、以下のような流れで処理されているということです。

![image](./0010_image/03a.png)

1. PC1がPC2のMACアドレスに向けてパケットを投げる
2. スイッチがそれを受けとり、どこあてのMACアドレスかを見てPC2宛であることに気づく
3. スイッチがPC2が接続されているインターフェースからパケットをだす
4. PC2がパケットを受け取る
5. PC2からPC1宛への通信も同様に行う

MACアドレスもIPアドレスと同じく、インターフェースごとに持っているアドレスです。
各ルーター1,2のインターフェースがどのようなMACアドレスを持っているか確認します。

```text
PC1#show int g0/1
GigabitEthernet0/1 is up, line protocol is up
  Hardware is iGbE, address is fa16.3e1f.a079 (bia fa16.3e1f.a079)
  Internet address is 10.0.1.101/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Auto Duplex, Auto Speed, link type is auto, media type is RJ45
以下省略

PC2#show int g0/1
GigabitEthernet0/1 is up, line protocol is up
  Hardware is iGbE, address is fa16.3e7f.78a1 (bia fa16.3e7f.78a1)
  Internet address is 10.0.1.102/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Auto Duplex, Auto Speed, link type is auto, media type is RJ45
以下省略
```

上記出力を見て分かるように、PC1はMACアドレス「」を持っていて、PC2はMACアドレス「」を持っています。
次にスイッチがどのようにMACアドレスを認識しているかを確認してみます。

```text
S1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    fa16.3e1f.a079    DYNAMIC     Gi0/1
   1    fa16.3e7f.78a1    DYNAMIC     Gi0/2
```

この出力を見ると、物理インターフェースとMACアドレスの結びつけが分かります。
PC1はインターフェースXXの先にいて、PC2はインターフェースXXの先にいます。
この「どのMACアドレスの機器がどのインターフェースの先にいるか」という対応表のことを「MACアドレステーブル」と呼んでいます。
スイッチはこのMACアドレステーブルに従って、どこのインターフェースからパケットを出すかを判断しています。
市販のスイッチは業務用とは異なり状態などは確認できませんが、この仕組みは同じです。

さて、MACアドレスとMACアドレステーブルがどのようなものか分かったと思いますので、
先ほどのPC1とPC2の通信の流れを詳細に確認してみます。

<<図>>

なお、MACアドレスを使った通信は機器と機器の間にスイッチがない直結の場合でも行われます。
PCとPCを直結することは珍しいですが、ルーターとルーターを繋ぐ場合もMACアドレスを使ってお互いにパケットをやりとりしています。

```text
PC1#show int g0/1 | inc address
  Hardware is iGbE, address is fa16.3e1f.a079 (bia fa16.3e1f.a079)
  Internet address is 10.0.1.101/24
```

### ARP学習とARP Table

同一ネットワーク内でパケットをやりとりする場合はMACアドレスを使うのでした。
ただ、それをするためにはPCが「相手のMAC」アドレスを知ったり、
ホストとホストの間にいるスイッチが「どのインターフェースの先にどのMACアドレスを持つ機器がいるか」というMACアドレステーブルを更新する必要があります。

本章ではPC1からPC2に対してPingをしましたが、その際に指定した宛先はPC2のIPアドレスです。
通常は機器と機器が通信をする際は、相手をIPアドレスで指定します。
ただ、同じネットワーク内の転送にはMACアドレスが利用されます。
そのため、IPアドレスからMACアドレスを求める仕組みが必要となり、ARPという仕組みがそれを実現しています。

<<図>>

PC1がPC2に通信をする際に、まずARPを使って「192.168.0.102のアドレスを持つ人は応答して下さい」と特殊なパケットを投げます。
このパケットはスイッチで受け取ったポートを除く全てのポートに対して拡散されますので、これがPC2とPC3に届きます。
PC3はそのARPを見て「これは私のIPではないな。無視」として応答しませんが、PC2は「私のIPアドレスだ。応答しよう」となり、
ARPに対して応答します。
この応答にはMACアドレスが含まれているため、それを受け取ったPC1は「IPが192.168.0.102の機器のMACアドレスはXXXXだ」と分かり、
PC2に対してパケットを送ることができるようになります。

1パケット送るごとにこのようなことをやっていると非効率ですので、PC1はこの結果をしばらく覚えておき、
継続する通信ではARPは発生しません。
この覚えていることを「ARPキャッシュ」と呼びます。
ただ、定められた一定時間の間にPC2と通信をしないと、
このキャッシュは消されます。
これはIPとMACアドレスの対応関係が永遠に続くとは限らないためです。
例えば新しい機器に、すでに取り外された機器のIPアドレスを与えた際に、
昔の機器のMACアドレス宛にパケットを送り続けたら困りますよね。

ARPの学習状況がどのようになっているかは以下のコマンドで確認できます。

```text
PC1#show ip arp  
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.0.1.101              -   fa16.3e1f.a079  ARPA   GigabitEthernet0/1
Internet  10.0.1.102              3   fa16.3e7f.78a1  ARPA   GigabitEthernet0/1


PC1#ping 10.0.1.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.103, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 4/4/5 ms

PC1#show ip arp    
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.0.1.101              -   fa16.3e1f.a079  ARPA   GigabitEthernet0/1
Internet  10.0.1.102              6   fa16.3e7f.78a1  ARPA   GigabitEthernet0/1
Internet  10.0.1.103              0   fa16.3e18.af47  ARPA   GigabitEthernet0/1

PC1#ping 10.0.1.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.103, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/5 ms

S1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    fa16.3e18.af47    DYNAMIC     Gi0/3
   1    fa16.3e1f.a079    DYNAMIC     Gi0/1
   1    fa16.3e7f.78a1    DYNAMIC     Gi0/2

PC3#show int g0/1 | inc address
  Hardware is iGbE, address is fa16.3e18.af47 (bia fa16.3e18.af47)
  Internet address is 10.0.1.103/24
```

### MAC学習とMac address Table

次にMACアドレステーブルの構築です。
同一ネットワーク内の機器同士の通信にはMACアドレスを使い、やりとりするパケットには
「誰あてということを示す宛先MAC」と「誰からの通信ということを示す送信元MAC」が含まれています。
PC1からPC2に対してパケットを投げた際に、それを中継するスイッチとして確実に分かることは、
「PC1から送られてきたパケットを受け取った先に、PC1が確実に存在する」ということです。
そのタイミングでMACアドレステーブルに対して「PC1のMACアドレス宛の通信は、インターフェースXから出す」と書き込みます。

送信元MACアドレスはパケットを受け取ったタイミングで学習できますが、
宛先はMACアドレステーブルに登録されていないと、どこにいるか分かりません。
どのポートの先に通信を届けたい機器がいるか分からない状況では、
先ほどのARPと同じように「全てのポートからパケットを出す」ことで相手にパケットを届けようとします。

PC2宛のパケットはPC2のポートとPC3のポートから出され、PC3はその宛先MACが自分宛てでないため無視します。
一方PC2はそれが自分宛てのパケットであるため、それを処理(この場合はPingに応答)し、必要であれば送信元に返信をします。
返信は宛先がPC1のMACアドレスで、送信元がPC2のMACアドレスとなっているため、
これを中継するスイッチは返信を受け取ったタイミングで「ポートXの先にMACアドレスXXXXXの機器がいる」とMACアドレステーブルを更新します。
MACアドレステーブルにMACアドレスが登録されてしまえば、そのMAC宛の通信はそのMACがいるポートからしか出なくなります。
そのため、PC1とPC2のパケットのやりとりがPC3に対して届けられてしまうということはありません。

<<図>>

このMACアドレステーブルに書かれている内容(どのMACアドレスがどのポートの先にいるか)も、
先ほどのARPと同じように一定時間利用されないと消されてしまいます。
使っている機器を別の場所に動かした場合に、昔の場所に永遠にパケットを送り続けると困るためです。

### 大きなネットワークの構築

1つのネットワークに接続する機器が全て同じスイッチに接続されているとは限りません。
スイッチはネットワークを拡張する役割があるので、スイッチにスイッチを繋げるとより大きなネットワークを作ることができます。
業務用のスイッチの多くは24ポートもしくは48ポートですので、大きな部署で人数分のPCとIP電話を繋げばすぐにポートを使い尽くしてしまいます。
そのような時はスイッチを木構造で繋げることで大きな一つのネットワークを作ります。
例えば以下のような上位のスイッチ(スイッチを繋げるためのスイッチ)に下位のスイッチ(ユーザーを繋げるスイッチ)を繋げるという構造です。

<<図>>

ユーザが増えれば上位スイッチに繋げる下位スイッチを増やせばいいですし、それでも足りなければ階層構造を3つに増やせます。
実際はこの階層化に加えて冗長化技術(機器が壊れてもネットワークを提供し続けられる)も使いますが、それは後ほど扱います。

先ほどの1つのスイッチの構成と、スイッチを多段に繋げた構成ではMACアドレステーブルの学習内容が異なります。
PCを繋げたポートは1つのMACアドレスしか学習しませんが、スイッチを繋げたポートはその先にいる機器の複数のMACアドレスを学習します。
つまりMACアドレステーブルではポートとMACアドレスの対応関係が「1:1」ではなく、「1:多」となります。
PCが繋がるポートも同じで、特別なソフトウェアなどで複数のMACアドレスを使うと、スイッチのポートは複数のMACアドレスを学習します。

実際に以下の構成をVIRLで作成して、MACアドレスを確認してみます。

<<図>>

なお、特定の状況を除いてMACアドレスを変更することはあまり多くありませんが、
MACアドレステーブルの確認をしやすくするためにPCのインターフェースのMACアドレスを以下のように「mac-address」コマンドで変更します。

```text
PC1#conf t  
PC1(config)#int g0/1
PC1(config-if)#mac-address 0000.0000.0101
PC1(config-if)#end
```

ホストIPアドレスと同じMACを割り当てました。
「show interface」コマンドで確認すると、設定したMACアドレスがインターフェースに割り当てられていることがわかります。

```text
PC1#show int g0/1 | inc address
  Hardware is iGbE, address is 0000.0000.0101 (bia fa16.3e81.8a63)
  Internet address is 10.0.1.101/24
```

MACアドレスはランダムに割り振られていて重複などは通常はしません。
人為的にアドレスを与えるとトラブルになりますので、特別な理由がない限りは変更をしないでください。
同じようにPC2のMACアドレスを「0000.0000.0102」にし、
PC3のMACアドレスを「0000.0000.0103」にしました。

MACアドレスを変更したうえで、PC1,2,3の間でPingを発行し、MACアドレスをスイッチに学習させています。
各スイッチのMACアドレスの学習状況は以下のようになりました。

<<図>>

S1はPC3をS3を経由して学ぶため、PC3のMACは「S3が接続されているG0/3インタフェース」で学んでいます。
S2スイッチはPC1,2をS3を経由して学ぶため、S3が接続されるG0/2で2つのPCを学習しています。
S3はS1に接続される2つのPCのMACをS1が接続されるG0/1で学び、PC3はG0/2で学んでいます。

各スイッチの実際のMACテーブルは以下のようなものとなっていました。

```text
S1#show mac address-table          
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0000.0000.0101    DYNAMIC     Gi0/1
   1    0000.0000.0102    DYNAMIC     Gi0/2
   1    0000.0000.0103    DYNAMIC     Gi0/3
   1    fa16.3e3a.db08    DYNAMIC     Gi0/3
   1    fa16.3e56.0961    DYNAMIC     Gi0/3


S2#show mac address-table          
         Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0000.0000.0101    DYNAMIC     Gi0/2
   1    0000.0000.0102    DYNAMIC     Gi0/2
   1    0000.0000.0103    DYNAMIC     Gi0/1
   1    fa16.3e3a.a66a    DYNAMIC     Gi0/2
   1    fa16.3ea4.4f78    DYNAMIC     Gi0/2

S3#show mac address-table
         Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0000.0000.0101    DYNAMIC     Gi0/1
   1    0000.0000.0102    DYNAMIC     Gi0/1
   1    0000.0000.0103    DYNAMIC     Gi0/2
   1    fa16.3e56.0961    DYNAMIC     Gi0/2
   1    fa16.3ea4.4f78    DYNAMIC     Gi0/1      
```

上記のテーブルにはPC以外のMACアドレスも学習されていますが、
これはスイッチのインターフェースに割り当てられているものです。
例えばS2とS3が学習している「fa16.3ea4.4f78」は、S1のG0/3インタフェースのMACアドレスです。

```text
S1#show int g0/3 | inc address
  Hardware is iGbE, address is fa16.3ea4.4f78 (bia fa16.3ea4.4f78)
```

実はスイッチ間ではIPパケット以外にもお互いの機器情報の交換といった様々な通信が行われています。
それもMACアドレスを使って届け合うため、MAC学習されてMACテーブルにのってきます。
実際に運用をする際はMACアドレステーブルを気にすることはほとんどありませんが、
なにかトラブルが発生した場合は対象のMACアドレスがどこでどのように学習されているか調べることで問題解決の糸口が見つかる可能性があります。

### 大きなネットワークの問題点

さて、大きなネットワークセグメントを使えば多数の機器を繋げることが分かりました。
理屈としてはスイッチにスイッチを繋げていけば世界中のPCを繋げたネットワークが作れそうな気がしますが、
なぜそうなっていないと思いますか。

これにはいくつかの理由があります。
まず、ARPの動き(ブロードキャスト)を思い出して下さい。
あれはネットワーク全てに拡散されます。
もし1つのネットワークに接続された1億台のPCが互いにARPを投げあったら、ARPだけでネットワークは埋め尽くされて、
PCもARPへの応答(無視するか返信するか)処理でいっぱいいっぱいになってしまいます。
この問題を防ぐためにネットワークを必要以上に大きくすることはできません。

次の問題はMACアドレステーブルの大きさです。
今回の実験を通して分かったと思いますが、MACテーブルのエントリの数はおおよそネットワークに属する機器の数と同じになります。
スイッチがフレームを受け取った際にどこの出口から出せばいいかということを管理していますが、
高速な転送をするために転送チップに直接エントリを書くためそれほど多くの学習はできません。
MACテーブルの上限値を超えたフレームの学習はできないため、なにかを忘れる必要があり、
それが再びアンノウンユニキャストによるフラッディングを導いてしまいます。

大きなネットワークを作らないのは、障害を局所化するという意味合いもあります。
たとえば部署ごとにネットワークを切っていれば、ある部署の障害はその部署に閉じます。
一方、社内全員が同じネットワークに属していれば、障害は全社規模となってしまいます。

他には同じネットワーク内にある機器同士の通信を制御しにくいという問題点もあります。
機器は移動しますので、どのスイッチのどのポートに接続するかということは強く縛ることが難しいです。
そのような状況でお客さんと社員が同じネットワークに参加していると、
社外秘のファイルを持つサーバーが見えてしまったり、プリンターが勝手に使われてしまうという問題が起きるかもしれません。
お客さん用のネットワークと社員のネットワークを分離すれば、お客さんが社員用のネットワークにアクセスできないようにすれば済むだけです。

事象では複数のネットワークを繋ぐルーターの役割について扱います。

### コラムL2の機器

スイッチ
 - port
 - uplink

wifi
 - switchに接続
 - コントローラー
