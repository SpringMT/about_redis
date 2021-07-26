redis-cli infoの情報まとめ

https://redis.io/commands/INFO

## 詳細

https://github.com/redis/redis/blob/17511df59b96bfeab8b46d474c19ec929e605bb9/src/server.c#L4662-L4713

GIT_SHA1、GIT_DIRTY、BUILD_IDはビルド時に埋め込まれる。

https://github.com/redis/redis/blob/d96f47cf06b1cc24b82109e0e87ac5428517525a/src/mkreleasehdr.sh

```
% cat release.h
#define REDIS_GIT_SHA1 "32e61ee2"
#define REDIS_GIT_DIRTY "       0"
#define REDIS_BUILD_ID "909c4ad01506-1626840941"
```

## Server

 key | 例 | 説明
-----|----|---
redis_version | 255.255.255 | `version.h` の値
redis_git_sha1 | 32e61ee2 | `(git show-ref --head --hash=8 2> /dev/null || echo 00000000) | head -n1` の結果
redis_git_dirty | 0 | `git diff --no-ext-diff 2> /dev/null | wc -l` の結果
redis_build_id | 49a819ce78a07d0d | redisのバージョン、build_id、git_dirty、git_sha1のcrc64チェックサム
redis_mode | standalone | サーバーのモード standalone or sentinel or cluster https://github.com/redis/redis/blob/17511df59b96bfeab8b46d474c19ec929e605bb9/src/server.c#L4640-L4642 ここらへん
os | Darwin 19.6.0 x86_64 | uname(2) の sysname + release + machine https://github.com/redis/redis/blob/17511df59b96bfeab8b46d474c19ec929e605bb9/src/server.c#L4656
arch_bits | 64 | longのサイズで判定 https://github.com/redis/redis/blob/17511df59b96bfeab8b46d474c19ec929e605bb9/src/server.c#L2648
multiplexing_api | kqueue | redisのevent loopで使っている仕組み https://github.com/redis/redis/blob/7913d34d7c5fb9fdc35afc4de57311cad84522e0/src/ae.c#L52-L64 https://github.com/redis/redis/blob/074e28a46eb2646ab33002731fac6b4fc223b0bb/src/ae_kqueue.c#L141
atomicvar_api | c11-builtin | 利用しているAtomicvar API C++ 11以降が使えるか https://cpprefjp.github.io/reference/atomic/atomic.html
gcc_version | 4.2.1 | `__GNUC__` コンパイルに使ったgccのバージョン
process_id | 38293 | getpid(2) https://linuxjm.osdn.jp/html/LDP_man-pages/man2/getpid.2.html の値
process_supervised | no | suprvisedの仕組み https://github.com/redis/redis/blob/17511df59b96bfeab8b46d474c19ec929e605bb9/src/server.c#L4644-L4650
run_id | 171825293c57aa676e943d31ca4ee6632cb865c9 | サーバーを一意に識別するID sentinelやclusterで使う https://github.com/redis/redis/blob/17511df59b96bfeab8b46d474c19ec929e605bb9/src/server.c#L2637
tcp_port | 6379 | サーバーのTCP/IPのポート https://github.com/redis/redis/blob/17511df59b96bfeab8b46d474c19ec929e605bb9/src/server.c#L3202-L3211
server_time_usec | 1627283878649452 | 
uptime_in_seconds | 3746 |
uptime_in_days | 0 |
hz | 10 |
configured_hz | 10 |
lru_clock | 16671142 |
executable | /Users/springmt/redis/./src/redis-server |
config_file | |
io_threads_active | 0 |

uname(2) https://linuxjm.osdn.jp/html/LDP_man-pages/man2/uname.2.html
