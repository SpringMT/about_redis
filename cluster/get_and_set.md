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

hash slotの計算結果、
