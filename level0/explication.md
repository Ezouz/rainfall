
ls -la => level1 owner / level0 group
permissions => file rw permission for group / read execute / extended

download file
`$scp -P 4242 level0@192.168.56.103:/home/user/level0/level0 /home/zouz/Desktop/level0`

open it with cutter, in main you can see the line:
`
0x08048ed4      call atoi          ; sym.atoi ; int atoi(const char *str)
0x08048ed9      cmp eax, 0x1a7     ; 423
`
so we see it takes a parameter, transform  and compare its value

execute level0 program with parameter: 423
`$./level0 423`

then you can access
`$ls -la /home/user/level1`
output: 
`
...
-rw-r--r--+ 1 level1 level1   65 Sep 23  2015 .pass
`

read it !
