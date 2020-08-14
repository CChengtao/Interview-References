ctrl-c：发送 SIGINT 信号给前台进程组中的所有进程。常用于终止正在运行的程序；

ctrl-z：发送 SIGTSTP信号给前台进程组中的所有进程，常用于挂起一个进程；

ctrl-d：不是发送信号，而是表示一个特殊的二进制值，表示 EOF，作用相当于在终端中输入exit后回车；

ctrl-\：发送 SIGQUIT 信号给前台进程组中的所有进程，终止前台进程并生成 core 文件；

ctrl-s：中断控制台输出；

ctrl-q：恢复控制台输出；

ctrl-l：清屏

```
cct@TNT:~$ stty -a
speed 38400 baud; rows 30; columns 120; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = M-^?; eol2 = M-^?; swtch = M-^?; start = ^Q; stop = ^S;
susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 hupcl -cstopb cread -clocal -crtscts
-ignbrk brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc ixany imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke -flusho -extproc
```

