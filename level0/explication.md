

We can see that the file level0 belongs to user level1. We are in the same group users, so we can execute it. 
If we reverse engineer the file, we see that the program expects an argument and does a simple comparison with it to open a shell.

In main you can see the line:
```
0x08048ed4      call atoi          ; sym.atoi ; int atoi(const char *str)
0x08048ed9      cmp eax, 0x1a7     ; 423
```

To get the flag, execute level0 program with parameter: 423
```
level0@RainFall:~$ ./level0 423
$ whoami
level1
```

then you can access
```
$ cat /home/user/level1/.pass
```
