Mac上でやっているのでlldbでやる

```
% lldb -p 38293
(lldb) process attach --pid 38293
Process 38293 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP
    frame #0: 0x00007fff69b66766 libsystem_kernel.dylib`kevent + 10
libsystem_kernel.dylib`kevent:
->  0x7fff69b66766 <+10>: jae    0x7fff69b66770            ; <+20>
    0x7fff69b66768 <+12>: movq   %rax, %rdi
    0x7fff69b6676b <+15>: jmp    0x7fff69b62629            ; cerror_nocancel
    0x7fff69b66770 <+20>: retq   
Target 0: (redis-server) stopped.

Executable module set to "/Users/springmt/redis/src/redis-server".
Architecture set to: x86_64h-apple-macosx-.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP
  * frame #0: 0x00007fff69b66766 libsystem_kernel.dylib`kevent + 10
    frame #1: 0x000000010770aaf2 redis-server`aeProcessEvents [inlined] aeApiPoll(eventLoop=0x00007fcce2439560, tvp=0x00007ffee84fc6e0) at ae_kqueue.c:0
    frame #2: 0x000000010770aaa7 redis-server`aeProcessEvents(eventLoop=<unavailable>, flags=27) at ae.c:396
    frame #3: 0x000000010770ae5d redis-server`aeMain(eventLoop=0x00007fcce2439560) at ae.c:488:9
    frame #4: 0x000000010771932f redis-server`main(argc=1, argv=0x00007ffee84fc7f8) at server.c:6403:5
    frame #5: 0x00007fff69a20cc9 libdyld.dylib`start + 1
    frame #6: 0x00007fff69a20cc9 libdyld.dylib`start + 1
```
