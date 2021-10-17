```
./redis/src/redis-cli -c -p 30001 cluster nodes
70eaba987e6eef0f64bad9acdb92c24867225e4b 127.0.0.1:30003@40003 master - 0 1634477918000 3 connected 10923-16383
1015f2dc1d46636374a2efa50e5ae15695723ae9 127.0.0.1:30004@40004 slave 573f1be84fd72542cd1007490bd97738123b48b7 0 1634477918126 2 connected
dc370bfff3c18d35dbcde975c73627f48ddbe370 127.0.0.1:30001@40001 myself,master - 0 1634477917000 1 connected 0-5460
573f1be84fd72542cd1007490bd97738123b48b7 127.0.0.1:30002@40002 master - 0 1634477917619 2 connected 5461-10922
e958966cd62b12db199930db72b34a3426005142 127.0.0.1:30005@40005 slave 70eaba987e6eef0f64bad9acdb92c24867225e4b 0 1634477918330 3 connected
d3326d691cd69221fbe4d7767055c609caaf40ef 127.0.0.1:30006@40006 slave dc370bfff3c18d35dbcde975c73627f48ddbe370 0 1634477918228 1 connected
```

スコアが大きいと順位が上とする

```
zadd "ranking1" 1 "a"
zadd "ranking1" 2 "b"
zadd "ranking1" 3 "c"
zadd "ranking1" 4 "d"
zadd "ranking1" 4 "e"
zadd "ranking1" 5 "f"
zadd "ranking1" 6 "g"
zadd "ranking1" 6 "h"
zadd "ranking1" 7 "i"
```

```
127.0.0.1:30001> keys *
1) "ranking1"
127.0.0.1:30001> zrange ranking1 0 10
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
7) "g"
8) "h"
9) "i"
```

順位としてはこうしたい

順位 | Name | Score
----|---|---
1 | i | 7
2 | h | 6
2 | g | 6
4 | f | 5
5 | e | 4
5 | d | 4
5 | c | 3
8 | b | 2
9 | a | 1

dの人の順位を取得したいとすると

```
127.0.0.1:30001> zscore "ranking1" "d"
"4"
127.0.0.1:30001> zcount "ranking1" "(4" '+inf'
(integer) 4
```

ZCOUNTで得られた値に+1すると順位を得られる。


