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
redis_version | 255.255.255 | 
redis_git_sha1 | 32e61ee2 | `(git show-ref --head --hash=8 2> /dev/null || echo 00000000) | head -n1` の結果
redis_git_dirty | 0 | `git diff --no-ext-diff 2> /dev/null | wc -l` の結果
redis_build_id | 49a819ce78a07d0d | redisのバージョン、build_id、git_dirty、git_sha1のcrc64チェックサム
redis_mode | standalone | 
os | Darwin 19.6.0 x86_64 |
arch_bits | 64 |
multiplexing_api | kqueue |
atomicvar_api | c11-builtin |
gcc_version | 4.2.1 |
process_id | 38293 |
process_supervised | no |
run_id | 171825293c57aa676e943d31ca4ee6632cb865c9 |
tcp_port | 6379 |
server_time_usec | 1627283878649452 |
uptime_in_seconds | 3746 |
uptime_in_days | 0 |
hz | 10 |
configured_hz | 10 |
lru_clock | 16671142 |
executable | /Users/springmt/redis/./src/redis-server |
config_file | |
io_threads_active | 0 |
