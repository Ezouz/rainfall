
```
level8@RainFall:~$ ./level8
(nil), (nil) 
```

in a loop
exits on count 2
=> keep stdin open

        iVar2 = fgets(auStack144, 0x80, _stdin);
        

comparisons auStack144 && "auth" "reset" "service" "login"

somehow unlock 
                system("/bin/sh");


placer des valeurs pour auth et service

edx et esi => 0xbffff5b0

   0x080485cf <+107>:	repz cmps BYTE PTR ds:[esi],BYTE PTR es:[edi]
