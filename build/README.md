## 手元でbuild

Mac(intel)でビルドしてみる

```
make BUILD_TLS=yes
make test
```

テストも手元でサクッと通った。

## redis server立ち上げ

```
./src/redis-server
```

## redis serverとの通信
https://redis.io/topics/protocol

### CLIを使う

```
./src/redis-cli
127.0.0.1:6379> PING
PONG
```
送信部分はこんな感じ

```
0000   02 00 00 00 45 00 00 42 00 00 40 00 80 06 00 00   ....E..B..@.....
0010   7f 00 00 01 7f 00 00 01 f9 0f 18 eb f4 4c 9d 69   .............L.i
0020   d3 1f 5e dc 80 18 17 a5 fe 36 00 00 01 01 08 0a   ..^......6......
0030   34 95 46 48 34 95 05 71 2a 31 0d 0a 24 34 0d 0a   4.FH4..q*1..$4..
0040   50 49 4e 47 0d 0a                                 PING..
```

TCPのコントロールビットでPSHのフラグが立っています。

これは受信したデータをバッファリングせずにすぐに上位のアプリケーションに渡すためのフラグです。

これは、 `REDIS_OPT_NO_PUSH_AUTOFREE` できるぽい感じがします。

https://github.com/redis/redis/blob/d96f47cf06b1cc24b82109e0e87ac5428517525a/deps/hiredis/README.md#specifying-no-handler

data部分は `2a310d0a 24340d0a 50494e47 0d0a`

`2a 31 0d 0a` は `* 1 \r \n`

`24 34 0d0a` は `$ 4 \r \n`

`50494e47` は `P I N G`

`0d 0a` は`\r\n`

受信部分は

```
0000   02 00 00 00 45 00 00 3b 00 00 40 00 80 06 00 00   ....E..;..@.....
0010   7f 00 00 01 7f 00 00 01 18 eb f9 0f d3 1f 5e dc   ..............^.
0020   f4 4c 9d 77 80 18 18 e7 fe 2f 00 00 01 01 08 0a   .L.w...../......
0030   34 95 46 48 34 95 46 48 2b 50 4f 4e 47 0d 0a      4.FH4.FH+PONG..
```

data部分は `2b504f4e470d0a`

`2b` は `+`

`50 4f 4e 47` は P O N G

`0d 0a` は`\r\n`

#### Redisのプロトコル RESP
https://redis.io/topics/protocol

これを参照すると、送信では `*` の直後1によりArrayが1個で、`$` 直後の値4により長さが4 byteのBulk Strings(P I N G) であることがわかる。

受信では、 `+` によって文字列が帰ってくることがわかり、PONGが返っている。

RESPは必ず CRLFで終わる。

### telnet
telnetでも通信できる。


