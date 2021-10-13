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



