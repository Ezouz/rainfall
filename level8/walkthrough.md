The complexity here is to read the operations of the `main`. Let's try the program :

```
level8@RainFall:~$ ./level8
(nil), (nil) 
```

It keeps looping, and keep stdin open listening to its values every time.

```
iVar2 = fgets(auStack144, 0x80, _stdin);
```

Thanks to `fgets`, `stdin` value is saved in `auStack144`.

```
printf("%p, %p \n", _auth, _service); 
```

We understand that there are 2 global variables named `auth` and `service` that can be usefull to us, let's check;

```
(gdb) info variables
All defined variables:

Non-debugging symbols:

...

0x08049aac  auth
0x08049ab0  service

...
```
comparisons auStack144 
 "auth " "reset" "service" "login"

somehow unlock 
                system("/bin/sh");

