```
ec7d6a21305906410839267e789c50b238eff942 127.0.0.1:30003@40003 master - 0 1634135197510 3 connected 10923-16383
3aa27fc41432d28561e042bb75887a4d97868794 127.0.0.1:30001@40001 slave a1c349eb26143ed43b837f36ecdef9b0d784278d 0 1634135198225 7 connected
6bd8620239771bd782f586480dbb18532a225238 127.0.0.1:30002@40002 master - 0 1634135197409 2 connected 5461-10922
39c1c10a4cd641bbbed514dec8148dc8c81a65fd 127.0.0.1:30004@40004 slave ec7d6a21305906410839267e789c50b238eff942 0 1634135198123 3 connected
b98eef26fb94dcf3a19d4c475636fed70707da75 127.0.0.1:30006@40006 slave 6bd8620239771bd782f586480dbb18532a225238 0 1634135197511 2 connected
a1c349eb26143ed43b837f36ecdef9b0d784278d 127.0.0.1:30005@40005 myself,master - 0 1634135198000 7 connected 0-5460
```

30001 portのnodeがslave 30005 portのnodeがmasterで、それぞれredis-cli経由でアクセスしておく

30001はslave

```
127.0.0.1:30001> READONLY
OK
127.0.0.1:30001> get sample1 
"2"
```

30005はmaster

```
127.0.0.1:30005> set sample1 2
OK
```

## failover実行

30001でfailoverを引き起こす

```
127.0.0.1:30001> cluster failover
OK
```

30001 portのnodeがmasterへ、30005 portのnodeがslaveに切り替わる

```
c7d6a21305906410839267e789c50b238eff942 127.0.0.1:30003@40003 master - 0 1634135590590 3 connected 10923-16383
3aa27fc41432d28561e042bb75887a4d97868794 127.0.0.1:30001@40001 master - 0 1634135591101 8 connected 0-5460
6bd8620239771bd782f586480dbb18532a225238 127.0.0.1:30002@40002 master - 0 1634135590693 2 connected 5461-10922
39c1c10a4cd641bbbed514dec8148dc8c81a65fd 127.0.0.1:30004@40004 slave ec7d6a21305906410839267e789c50b238eff942 0 1634135591304 3 connected
b98eef26fb94dcf3a19d4c475636fed70707da75 127.0.0.1:30006@40006 slave 6bd8620239771bd782f586480dbb18532a225238 0 1634135591100 2 connected
a1c349eb26143ed43b837f36ecdef9b0d784278d 127.0.0.1:30005@40005 myself,slave 3aa27fc41432d28561e042bb75887a4d97868794 0 1634135590000 8 connected
```

で、それぞれ同じことをしてみる
```
127.0.0.1:30001> READONLY
OK
127.0.0.1:30001> get sample1 
"2"
127.0.0.1:30001> get sample1
"2"
127.0
```

問題ない

```
127.0.0.1:30005> set sample1 3
-> Redirected to slot [47] located at 127.0.0.1:30001
OK
```

30005はsetを売った場合、30001 portのnode　(master)へredirectされてSETが実行される。
