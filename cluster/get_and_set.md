Clusterの確認

```
% ./redis/redis/src/redis-cli -p 30001 CLUSTER NODES
ec7d6a21305906410839267e789c50b238eff942 127.0.0.1:30003@40003 master - 0 1634131398148 3 connected 10923-16383
a1c349eb26143ed43b837f36ecdef9b0d784278d 127.0.0.1:30005@40005 slave 3aa27fc41432d28561e042bb75887a4d97868794 0 1634131398352 1 connected
3aa27fc41432d28561e042bb75887a4d97868794 127.0.0.1:30001@40001 myself,master - 0 1634131398000 1 connected 0-5460
6bd8620239771bd782f586480dbb18532a225238 127.0.0.1:30002@40002 master - 1634131398554 1634131397537 2 connected 5461-10922
b98eef26fb94dcf3a19d4c475636fed70707da75 127.0.0.1:30006@40006 slave 6bd8620239771bd782f586480dbb18532a225238 0 1634131398352 2 connected
39c1c10a4cd641bbbed514dec8148dc8c81a65fd 127.0.0.1:30004@40004 slave ec7d6a21305906410839267e789c50b238eff942 0 1634131398000 3 connected
```

masterに接続してKeyをSet and Getする

```
./redis/redis/src/redis-cli -p 30001 
```

set してみる

```
127.0.0.1:30001> set sample1 1
OK
```

```
22:47:51.133629 IP localhost.59866 > localhost.pago-services1: Flags [P.], seq 3121842814:3121842847, ack 724219217, win 5426, options [nop,nop,TS val 1601944493 ecr 1601888061], length 33
E..U..@...............u1...~+*.Q...2.I.....
_{.._z.=*3
$3
set
$7
sample1
$1
1

22:47:51.133685 IP localhost.pago-services1 > localhost.59866: Flags [.], ack 33, win 6377, options [nop,nop,TS val 1601944493 ecr 1601944493], length 0
E..4..@.............u1..+*.Q.........(.....
_{.._{..
22:47:51.133958 IP localhost.pago-services1 > localhost.59866: Flags [P.], seq 1:6, ack 33, win 6377, options [nop,nop,TS val 1601944493 ecr 1601944493], length 5
E..9..@.............u1..+*.Q.........-.....
_{.._{..+OK

22:47:51.134025 IP localhost.59866 > localhost.pago-services1: Flags [.], ack 6, win 5426, options [nop,nop,TS val 1601944493 ecr 1601944493], length 0
E..4..@...............u1....+*.V...2.(.....
_{.._{..
```

別でセットしてみる

```
127.0.0.1:30001> set sample2 1
(error) MOVED 12364 127.0.0.1:30003
127.0.0.1:30001> CLUSTER KEYSLOT sample2
(integer) 12364
```

hash slotの計算結果、別nodeに格納するべきkeyであることがわかる。

ここで redis-cliを `-c` optionを付けて実行し、どのようになるかを観察する。

```
127.0.0.1:30001> set sample2 1
-> Redirected to slot [12364] located at 127.0.0.1:30003
OK
127.0.0.1:30003> 
```

SETするkeyが別Nodeであることがわかると、redirectされ、cliは別portにアクセスしなおされる。

(30001 portへのTCP接続は一旦切って30003 portのnodeに対してconnectする)

その上で、SETがあらためて実装され完了する。

```
23:02:10.782776 IP localhost.60869 > localhost.pago-services1: Flags [P.], seq 18:51, ack 63451, win 5398, options [nop,nop,TS val 1602785507 ecr 1602770854], length 33
E..U..@...............u1p.hbh........I.....
_..._.W.*3
$3
set
$7
sample2
$1
1

23:02:10.782841 IP localhost.pago-services1 > localhost.60869: Flags [.], ack 51, win 6378, options [nop,nop,TS val 1602785507 ecr 1602785507], length 0
E..4..@.............u1..h...p.h......(.....
_..._...
23:02:10.783020 IP localhost.pago-services1 > localhost.60869: Flags [P.], seq 63451:63481, ack 51, win 6378, options [nop,nop,TS val 1602785508 ecr 1602785507], length 30
E..R..@.............u1..h...p.h......F.....
_..._...-MOVED 12364 127.0.0.1:30003

23:02:10.783058 IP localhost.60869 > localhost.pago-services1: Flags [.], ack 63481, win 5397, options [nop,nop,TS val 1602785508 ecr 1602785508], length 0
E..4..@...............u1p.h.h..=.....(.....
_..._...
23:02:10.783198 IP localhost.60869 > localhost.pago-services1: Flags [F.], seq 51, ack 63481, win 5397, options [nop,nop,TS val 1602785508 ecr 1602785508], length 0
E..4..@...............u1p.h.h..=.....(.....
_..._...
23:02:10.783243 IP localhost.pago-services1 > localhost.60869: Flags [.], ack 52, win 6378, options [nop,nop,TS val 1602785508 ecr 1602785508], length 0
E..4..@.............u1..h..=p.h......(.....
_..._...
23:02:10.783331 IP localhost.pago-services1 > localhost.60869: Flags [F.], seq 63481, ack 52, win 6378, options [nop,nop,TS val 1602785508 ecr 1602785508], length 0
E..4..@.............u1..h..=p.h......(.....
_..._...
23:02:10.783549 IP localhost.60869 > localhost.pago-services1: Flags [.], ack 63482, win 5397, options [nop,nop,TS val 1602785508 ecr 1602785508], length 0
E..4..@...............u1p.h.h..>.....(.....
_..._...
23:02:10.783656 IP localhost.60882 > localhost.30003: Flags [S], seq 3032706570, win 65535, options [mss 16344,nop,wscale 6,nop,nop,TS val 1602785508 ecr 0,sackOK,eol], length 0
E..@..@...............u3..n
.........4....?........
_...........
23:02:10.783892 IP localhost.30003 > localhost.60882: Flags [S.], seq 797791088, ack 3032706571, win 65535, options [mss 16344,nop,wscale 6,nop,nop,TS val 1602785508 ecr 1602785508,sackOK,eol], length 0
E..@..@.............u3../.Sp..n......4....?........
_..._.......
23:02:10.783916 IP localhost.60882 > localhost.30003: Flags [.], ack 1, win 6379, options [nop,nop,TS val 1602785508 ecr 1602785508], length 0
E..4..@...............u3..n./.Sq.....(.....
_..._...
23:02:10.783932 IP localhost.30003 > localhost.60882: Flags [.], ack 1, win 6379, options [nop,nop,TS val 1602785508 ecr 1602785508], length 0
E..4..@.............u3../.Sq..n......(.....
_..._...
23:02:10.784105 IP localhost.60882 > localhost.30003: Flags [P.], seq 1:34, ack 1, win 6379, options [nop,nop,TS val 1602785509 ecr 1602785508], length 33
E..U..@...............u3..n./.Sq.....I.....
_..._...*3
$3
set
$7
sample2
$1
1

23:02:10.784151 IP localhost.30003 > localhost.60882: Flags [.], ack 34, win 6379, options [nop,nop,TS val 1602785509 ecr 1602785509], length 0
E..4..@.............u3../.Sq..n,.....(.....
_..._...
23:02:10.785926 IP localhost.30003 > localhost.60882: Flags [P.], seq 1:6, ack 34, win 6379, options [nop,nop,TS val 1602785510 ecr 1602785509], length 5
E..9..@.............u3../.Sq..n,.....-.....
_..._...+OK
```

## replicaにアクセスしてみる

```
127.0.0.1:30005> get sample1
(error) MOVED 47 127.0.0.1:30001
127.0.0.1:30005> READONLY
OK
127.0.0.1:30005> get sample1
"1"
```

slaveへのアクセスは予めREADONLYが必要となる

```
% ./redis/redis/src/redis-cli -c -p 30005
127.0.0.1:30005> get sample1
-> Redirected to slot [47] located at 127.0.0.1:30001
"1"
```
clusterモードだとこれもredirectされる

```
127.0.0.1:30005> set sample1 2
-> Redirected to slot [47] located at 127.0.0.1:30001
OK
```
setであってもredirectされる

## Ruby Clientを使ったテスト

clusterの状態
```
ec7d6a21305906410839267e789c50b238eff942 127.0.0.1:30003@40003 master - 0 1634170806660 3 connected 10923-16383
3aa27fc41432d28561e042bb75887a4d97868794 127.0.0.1:30001@40001 master - 0 1634170806660 8 connected 0-5460
6bd8620239771bd782f586480dbb18532a225238 127.0.0.1:30002@40002 master - 0 1634170806252 2 connected 5461-10922
39c1c10a4cd641bbbed514dec8148dc8c81a65fd 127.0.0.1:30004@40004 slave ec7d6a21305906410839267e789c50b238eff942 0 1634170806252 3 connected
b98eef26fb94dcf3a19d4c475636fed70707da75 127.0.0.1:30006@40006 slave 6bd8620239771bd782f586480dbb18532a225238 0 1634170806252 2 connected
a1c349eb26143ed43b837f36ecdef9b0d784278d 127.0.0.1:30005@40005 myself,slave 3aa27fc41432d28561e042bb75887a4d97868794 0 1634170806000 8 connected
```

```
irb(main):001:0> redis = Redis.new(cluster: %w[redis://127.0.0.1:30005])
=> #<Redis client v4.4.0 for redis://127.0.0.1:30001/0 redis://127.0.0.1:30002/0 redis://127.0.0.1:30003/0>
irb(main):002:0> redis.get("sample1")
=> "3"
irb(main):006:0> redis.connection
=> 
[{:host=>"127.0.0.1", :port=>30001, :db=>0, :id=>"redis://127.0.0.1:30001/0", :location=>"127.0.0.1:30001"},
 {:host=>"127.0.0.1", :port=>30002, :db=>0, :id=>"redis://127.0.0.1:30002/0", :location=>"127.0.0.1:30002"},
 {:host=>"127.0.0.1", :port=>30003, :db=>0, :id=>"redis://127.0.0.1:30003/0", :location=>"127.0.0.1:30003"}]
```

cluster modeだと #connection は、下記の呼び出しとなる。
https://github.com/redis/redis-rb/blob/master/lib/redis.rb#L3602-L3603

clusterの最初にslaveにつないでも、CLUSTER NODES打ったあと、TCPコネクション切って、masterへのコネクションに切り替わる。

https://github.com/redis/redis-rb/blob/a3d8071d7f936541293b912ff934d79a90f458ef/lib/redis/cluster.rb#L122

```
09:20:22.951206 IP localhost.51786 > localhost.30005: Flags [F.], seq 57, ack 1262, win 6359, options [nop,nop,TS val 1637985096 ecr 1637985092], length 0
E..4..@..............Ju5...OS@]......(.....
a..Ha..D
09:20:22.951241 IP localhost.30005 > localhost.51786: Flags [.], ack 58, win 6378, options [nop,nop,TS val 1637985096 ecr 1637985096], length 0
E..4..@.............u5.JS@]....P.....(.....
a..Ha..H
09:20:22.951325 IP localhost.30005 > localhost.51786: Flags [F.], seq 1262, ack 58, win 6378, options [nop,nop,TS val 1637985096 ecr 1637985096], length 0
E..4..@.............u5.JS@]....P.....(.....
a..Ha..H
09:20:22.951357 IP localhost.51786 > localhost.30005: Flags [.], ack 1263, win 6359, options [nop,nop,TS val 1637985096 ecr 1637985096], length 0
E..4..@..............Ju5...PS@]......(.....
a..Ha..H

```

```
09:20:36.626571 IP localhost.51787 > localhost.pago-services1: Flags [P.], seq 18:44, ack 63451, win 5491, options [nop,nop,TS val 1637998329 ecr 1637985115], length 26
E..N..@..............Ku1.l.0...;...s.B.....
a...a..[*2
$3
get
$7
sample1

09:20:36.626662 IP localhost.pago-services1 > localhost.51787: Flags [.], ack 44, win 6379, options [nop,nop,TS val 1637998329 ecr 1637998329], length 0
E..4..@.............u1.K...;.l.J.....(.....
a...a...
09:20:36.628007 IP localhost.pago-services1 > localhost.51787: Flags [P.], seq 63451:63458, ack 44, win 6379, options [nop,nop,TS val 1637998330 ecr 1637998329], length 7
E..;..@.............u1.K...;.l.J...../.....
a...a...$1
3
```

### hash slotをまたぐ場合

```
irb(main):010:0> redis.get("sample1")
=> "3"
irb(main):011:0> redis.get("sample2")
=> "1"
```
クライアントでslot計算している。

https://github.com/redis/redis-rb/blob/a3d8071d7f936541293b912ff934d79a90f458ef/lib/redis/cluster.rb#L253-L256

```
09:43:13.148179 IP localhost.51787 > localhost.pago-services1: Flags [P.], seq 66:92, ack 63537, win 5489, options [nop,nop,TS val 1639328245 ecr 1638768839], length 26
E..N..@..............Ku1.l.`.......q.B.....
a.).a...*2
$3
get
$7
sample1

09:43:13.148255 IP localhost.pago-services1 > localhost.51787: Flags [.], ack 92, win 6378, options [nop,nop,TS val 1639328245 ecr 1639328245], length 0
E..4..@.............u1.K.....l.z.....(.....
a.).a.).
09:43:13.148360 IP localhost.pago-services1 > localhost.51787: Flags [P.], seq 63537:63544, ack 92, win 6378, options [nop,nop,TS val 1639328245 ecr 1639328245], length 7
E..;..@.............u1.K.....l.z...../.....
a.).a.).$1
3

09:43:13.148375 IP localhost.51787 > localhost.pago-services1: Flags [.], ack 63544, win 5489, options [nop,nop,TS val 1639328245 ecr 1639328245], length 0
E..4..@..............Ku1.l.z.......q.(.....
a.).a.).
09:43:17.039095 IP localhost.52170 > localhost.30003: Flags [P.], seq 16:42, ack 59, win 6378, options [nop,nop,TS val 1639332066 ecr 1638348296], length 26
E..N..@...............u3.a.Y.g.......B.....
a.8.a.6.*2
$3
get
$7
sample2

09:43:17.039144 IP localhost.30003 > localhost.52170: Flags [.], ack 42, win 6379, options [nop,nop,TS val 1639332066 ecr 1639332066], length 0
E..4..@.............u3...g...a.s.....(.....
a.8.a.8.
09:43:17.040955 IP localhost.30003 > localhost.52170: Flags [P.], seq 59:66, ack 42, win 6379, options [nop,nop,TS val 1639332067 ecr 1639332066], length 7
E..;..@.............u3...g...a.s...../.....
a.8.a.8.$1
1
```

### reprica true
```
irb(main):001:0> redis = Redis.new(cluster: %w[redis://127.0.0.1:30005], replica: true)
=> #<Redis client v4.4.0 for redis://127.0.0.1:30001/0 redis://127.0.0.1:30002/0 redis://127.0.0.1:30003/0 redis://127.0.0.1:30004/0 redis://127.0.0.1:30005/0 redis://127.0.0.1:30006/0>
irb(main):002:0> redis.connection
=> 
[{:host=>"127.0.0.1", :port=>30001, :db=>0, :id=>"redis://127.0.0.1:30001/0", :location=>"127.0.0.1:30001"},
 {:host=>"127.0.0.1", :port=>30002, :db=>0, :id=>"redis://127.0.0.1:30002/0", :location=>"127.0.0.1:30002"},
 {:host=>"127.0.0.1", :port=>30003, :db=>0, :id=>"redis://127.0.0.1:30003/0", :location=>"127.0.0.1:30003"},
 {:host=>"127.0.0.1", :port=>30004, :db=>0, :id=>"redis://127.0.0.1:30004/0", :location=>"127.0.0.1:30004"},
 {:host=>"127.0.0.1", :port=>30005, :db=>0, :id=>"redis://127.0.0.1:30005/0", :location=>"127.0.0.1:30005"},
 {:host=>"127.0.0.1", :port=>30006, :db=>0, :id=>"redis://127.0.0.1:30006/0", :location=>"127.0.0.1:30006"}]
```

https://github.com/redis/redis-rb/blob/a3d8071d7f936541293b912ff934d79a90f458ef/lib/redis/cluster.rb#L258-L270

このあたりで制御されています。

```
irb(main):003:0> redis.get("sample1")
=> "3"
irb(main):004:0> redis.get("sample2")
=> "1"
```

readonlyが発行され、実行される

```
09:54:09.021675 IP localhost.52929 > localhost.30005: Flags [P.], seq 1:19, ack 1, win 6379, options [nop,nop,TS val 1639972604 ecr 1639972603], length 18
E..F..@...............u5w....W.......:.....
a...a...*1
$8
readonly

09:54:09.021742 IP localhost.30005 > localhost.52929: Flags [.], ack 19, win 6379, options [nop,nop,TS val 1639972604 ecr 1639972604], length 0
E..4..@.............u5...W..w........(.....
a...a...
09:54:09.021824 IP localhost.30005 > localhost.52929: Flags [P.], seq 1:6, ack 19, win 6379, options [nop,nop,TS val 1639972604 ecr 1639972604], length 5
E..9..@.............u5...W..w........-.....
a...a...+OK

09:54:09.021857 IP localhost.52929 > localhost.30005: Flags [.], ack 6, win 6379, options [nop,nop,TS val 1639972604 ecr 1639972604], length 0
E..4..@...............u5w....W.......(.....
a...a...
09:54:09.022061 IP localhost.52929 > localhost.30005: Flags [P.], seq 19:45, ack 6, win 6379, options [nop,nop,TS val 1639972604 ecr 1639972604], length 26
E..N..@...............u5w....W.......B.....
a...a...*2
$3
get
$7
sample1

09:54:09.022115 IP localhost.30005 > localhost.52929: Flags [.], ack 45, win 6379, options [nop,nop,TS val 1639972604 ecr 1639972604], length 0
E..4..@.............u5...W..w..&.....(.....
a...a...
09:54:09.022972 IP localhost.30005 > localhost.52929: Flags [P.], seq 6:13, ack 45, win 6379, options [nop,nop,TS val 1639972605 ecr 1639972604], length 7
E..;..@.............u5...W..w..&...../.....
a...a...$1
3

09:54:09.023005 IP localhost.52929 > localhost.30005: Flags [.], ack 13, win 6379, options [nop,nop,TS val 1639972605 ecr 1639972605], length 0
E..4..@...............u5w..&.W.......(.....
a...a...
09:54:13.189672 IP localhost.52930 > localhost.30004: Flags [S], seq 1485578366, win 65535, options [mss 16344,nop,wscale 6,nop,nop,TS val 1639976722 ecr 0,sackOK,eol], length 0
E..@..@...............u4X. ~.........4....?........
a...........
09:54:13.189863 IP localhost.30004 > localhost.52930: Flags [S.], seq 139898076, ack 1485578367, win 65535, options [mss 16344,nop,wscale 6,nop,nop,TS val 1639976723 ecr 1639976722,sackOK,eol], length 0
E..@..@.............u4...V..X. ......4....?........
a...a.......
09:54:13.189879 IP localhost.52930 > localhost.30004: Flags [.], ack 1, win 6379, options [nop,nop,TS val 1639976723 ecr 1639976723], length 0
E..4..@...............u4X. ..V.......(.....
a...a...
09:54:13.189891 IP localhost.30004 > localhost.52930: Flags [.], ack 1, win 6379, options [nop,nop,TS val 1639976723 ecr 1639976723], length 0
E..4..@.............u4...V..X. ......(.....
a...a...
09:54:13.189992 IP localhost.52930 > localhost.30004: Flags [P.], seq 1:19, ack 1, win 6379, options [nop,nop,TS val 1639976723 ecr 1639976723], length 18
E..F..@...............u4X. ..V.......:.....
a...a...*1
$8
readonly

09:54:13.190017 IP localhost.30004 > localhost.52930: Flags [.], ack 19, win 6379, options [nop,nop,TS val 1639976723 ecr 1639976723], length 0
E..4..@.............u4...V..X. ......(.....
a...a...
09:54:13.192538 IP localhost.30004 > localhost.52930: Flags [P.], seq 1:6, ack 19, win 6379, options [nop,nop,TS val 1639976725 ecr 1639976723], length 5
E..9..@.............u4...V..X. ......-.....
a...a...+OK

09:54:13.192564 IP localhost.52930 > localhost.30004: Flags [.], ack 6, win 6379, options [nop,nop,TS val 1639976725 ecr 1639976725], length 0
E..4..@...............u4X. ..V.......(.....
a...a...
09:54:13.192694 IP localhost.52930 > localhost.30004: Flags [P.], seq 19:45, ack 6, win 6379, options [nop,nop,TS val 1639976725 ecr 1639976725], length 26
E..N..@...............u4X. ..V.......B.....
a...a...*2
$3
get
$7
sample2

09:54:13.192725 IP localhost.30004 > localhost.52930: Flags [.], ack 45, win 6379, options [nop,nop,TS val 1639976725 ecr 1639976725], length 0
E..4..@.............u4...V..X. ......(.....
a...a...
09:54:13.194255 IP localhost.30004 > localhost.52930: Flags [P.], seq 6:13, ack 45, win 6379, options [nop,nop,TS val 1639976726 ecr 1639976725], length 7
E..;..@.............u4...V..X. ....../.....
a...a...$1
1


```

#### failover

```
ec7d6a21305906410839267e789c50b238eff942 127.0.0.1:30003@40003 master - 0 1634170806660 3 connected 10923-16383
3aa27fc41432d28561e042bb75887a4d97868794 127.0.0.1:30001@40001 master - 0 1634170806660 8 connected 0-5460
6bd8620239771bd782f586480dbb18532a225238 127.0.0.1:30002@40002 master - 0 1634170806252 2 connected 5461-10922
39c1c10a4cd641bbbed514dec8148dc8c81a65fd 127.0.0.1:30004@40004 slave ec7d6a21305906410839267e789c50b238eff942 0 1634170806252 3 connected
b98eef26fb94dcf3a19d4c475636fed70707da75 127.0.0.1:30006@40006 slave 6bd8620239771bd782f586480dbb18532a225238 0 1634170806252 2 connected
a1c349eb26143ed43b837f36ecdef9b0d784278d 127.0.0.1:30005@40005 myself,slave 3aa27fc41432d28561e042bb75887a4d97868794 0 1634170806000 8 connected
```

```
```
