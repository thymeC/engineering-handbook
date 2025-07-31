## docker run -it --name my_centos7 centos:7 /bin/bash
```
(base) thyme@chenjingdeMacBook-Pro ~ % docker run -it --name my_centos7 centos:7 /bin/bash
[root@419a4bfd22cb /]# ls -la
total 68
drwxr-xr-x   1 root root  4096 Jul 30 17:54 .
drwxr-xr-x   1 root root  4096 Jul 30 17:54 ..
-rwxr-xr-x   1 root root     0 Jul 30 17:54 .dockerenv
-rw-r--r--   1 root root 12298 Nov 13  2020 anaconda-post.log
lrwxrwxrwx   1 root root     7 Nov 13  2020 bin -> usr/bin
drwxr-xr-x   5 root root   360 Jul 30 17:54 dev
drwxr-xr-x   1 root root  4096 Jul 30 17:54 etc
drwxr-xr-x   2 root root  4096 Apr 11  2018 home
lrwxrwxrwx   1 root root     7 Nov 13  2020 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Nov 13  2020 lib64 -> usr/lib64
drwxr-xr-x   2 root root  4096 Apr 11  2018 media
drwxr-xr-x   2 root root  4096 Apr 11  2018 mnt
drwxr-xr-x   2 root root  4096 Apr 11  2018 opt
dr-xr-xr-x 216 root root     0 Jul 30 17:54 proc
dr-xr-x---   2 root root  4096 Nov 13  2020 root
drwxr-xr-x  11 root root  4096 Nov 13  2020 run
lrwxrwxrwx   1 root root     8 Nov 13  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root  4096 Apr 11  2018 srv
dr-xr-xr-x  11 root root     0 Jul 30 17:54 sys
drwxrwxrwt   7 root root  4096 Nov 13  2020 tmp
drwxr-xr-x  13 root root  4096 Nov 13  2020 usr
drwxr-xr-x  18 root root  4096 Nov 13  2020 var
[root@419a4bfd22cb /]# docker
bash: docker: command not found
[root@419a4bfd22cb /]# ps 
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
   17 pts/0    00:00:00 ps
[root@419a4bfd22cb /]# pwd
/
[root@419a4bfd22cb /]# cd usr
[root@419a4bfd22cb usr]# ls
bin  etc  games  include  lib  lib64  libexec  local  sbin  share  src  tmp
[root@419a4bfd22cb usr]# mkdir thyme
[root@419a4bfd22cb usr]# cd thyme
[root@419a4bfd22cb thyme]# touch newfile
[root@419a4bfd22cb thyme]# ls
newfile
[root@419a4bfd22cb thyme]# vi newfile
[root@419a4bfd22cb thyme]# cat newfile
hhh
[root@419a4bfd22cb thyme]# tail newfile
hhh
[root@419a4bfd22cb thyme]#                                                    
```
