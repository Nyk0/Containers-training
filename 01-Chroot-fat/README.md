# Part 1: Nginx compilation

1. Get Nginx at http://nginx.org/en/download.html.

2. Decompress and untar it (tar -xzf nginx-XXX.tar.gz).

```bash
root@node1:~# tar -xzf nginx-XXX.tar.gz
```

> [!TIP]
> In this command, XXX refers to the current nginx version.

3. Install the "build-essential" and "screen" packages.

```bash
root@node1:~# apt-get install build-essential screen
```

4. Move to the Nginx sources directory.

```bash
root@node1:~# cd nginx-XXX
root@node1:~/nginx-XXX#
```

> [!CAUTION]
> It's important to launch every command of 5, 6, 7 and 8 from the nginx sources directory.

5. Browse compilation options with:

```bash
./configure --help | less
```
more specifically :

```bash
--prefix=PATH set installation prefix
--user=USER set non-privileged user for worker processes
--group=GROUP set non-privileged group for worker processes
```

6. Generate Makefile from the configure script:

```bash
root@node1:~/nginx-XXX#./configure --prefix=/opt/nginx-XXX \
  --user=nginx --group=nginx \
  --without-http_rewrite_module \
  --without-http_gzip_module
```

7. Launch compilation:

```bash
root@node1:~/nginx-1.28.0# make
```

8. Install the software according to the "--prefix" of step 6.

```bash
root@node1:~/nginx-XXX# make install
```

9. Create the dedicated system account:

```bash
root@node1:~/nginx-XXX# useradd -r -d /var/empty -s /bin/false nginx
```

10. Create a symlink to keep a consistant path to Nginx through version upgrades:

```bash
root@node1:~/nginx-XXX# cd /opt
root@node1:/opt# ln -s nginx-XXX nginx
```

11. Browse Nginx executable help:

```bash
root@node1:/opt# /opt/nginx/sbin/nginx -h
```

12. Add "daemon off;" to nginx.conf to prevent daemonization.

13. Launch nginx:

```bash
root@node1:/opt# /opt/nginx/sbin/nginx
```

14. Test Nginx from the frontend:

```bash
root@frontal:~# apt-get install curl
root@frontal:~# curl http://node1.lab.local
```

# Part 2: Fat chroot

1. Create a directory to create the fat Nginx chroot:

```bash
root@node1:~# mkdir -p /chroot/fat
```

2. Install and use debootstrap to put a stable Debian system in a target directory:

```bash
root@node1:~# apt-get install debootstrap
root@node1:~# debootstrap stable /chroot/fat/
```

4. Copy our /opt/nginx-XXX to /chroot/fat/opt/

```bash
root@node1:~# cp -r /opt/nginx-XXX to /chroot/fat/opt/
```

5. Change the content of index.html of the chrooted Nginx:

```bash
root@node1:~# echo "In fat chroot" > /chroot/fat/opt/nginx-XXX/html/index.html
```

7. Use chroot to chroot a bash in /chroot/fat:

```bash
root@node1:~# chroot /chroot/fat /bin/bash
root@node1:/#
```

9. Check the content of /opt/nginx-XXX/html/index.html.

11. Re-create the symlink inside the fat chroot.

12. Re-create the nginx system account inside the fat chroot.

> [!CAUTION]
> It's important to use the same UID and GID as in part 1, step 9.

```bash
root@node1:/# groupadd -g YYY nginx
root@node1:/# useradd -r -d /var/empty -s /bin/false -u ZZZ -g nginx nginx
```

Where YYY is the GID and ZZZ the UID of part 1, step 9.

14. Exit the fat chroot and launch a chrooted nginx:

```bash
root@node1:/# exit
exit
root@node1:~#
```

```bash
root@node1:~# chroot /chroot/fat /opt/nginx/sbin/nginx
```

15. Test from frontend:

```bash
root@frontal:~# curl http://node1.lab.local
In chroot
```
