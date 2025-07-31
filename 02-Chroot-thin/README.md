# Thin Nginx chroot

1. Create a directory to host the thin Nginx chroot:

```bash
root@node1:~# mkdir -p /chroot/thin/opt
```

2. Copy our /opt/nginx-XXX to /chroot/thin/opt/:

```bash
root@node1:~# cp -r /opt/nginx-XXX /chroot/thin/opt/
```

3. Change the content of index.html of the chrooted nginx:

root@node1:~# echo "In thin chroot" > /chroot/thin/opt/nginx-XXX/html/index.html

4. Analyze libraries dependencies of Nginx binary:

```bash
root@node1:~# ldd /chroot/thin/opt/nginx-XXX/sbin/nginx
  linux-vdso.so.1 (0x00007ffc9e5e0000)
  libcrypt.so.1 => /lib/x86_64-linux-gnu/libcrypt.so.1 (0x00007f3668fe5000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3668e04000)
  /lib64/ld-linux-x86-64.so.2 (0x00007f3669108000)
```

5. Re-create the library hierarchy inside the thin chroot directory but beware:

```bash
root@node1:~# file /lib/x86_64-linux-gnu/libcrypt.so.1
/lib/x86_64-linux-gnu/libcrypt.so.1: symbolic link to libcrypt.so.1.1.0
```

```bash
root@node1:~# ls -al /lib/x86_64-linux-gnu/libcrypt.so.1
lrwxrwxrwx 1 root root 17 Jan  6  2023 /lib/x86_64-linux-gnu/libcrypt.so.1 -> libcrypt.so.1.1.0
```

```bash
root@node1:~# file /lib/x86_64-linux-gnu/libcrypt.so.1.1.0
/lib/x86_64-linux-gnu/libcrypt.so.1.1.0: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=6c22e42bcfd160936db1a3eb5477f86c808d3e7f, stripped
```

```bash
root@node1:~# mkdir -p /chroot/thin/lib/x86_64-linux-gnu
```

```bash
root@node1:/lib/x86_64-linux-gnu# cd /chroot/thin/lib/x86_64-linux-gnu
root@node1:/chroot/thin/lib/x86_64-linux-gnu# cp /lib/x86_64-linux-gnu/libcrypt.so.1.1.0 ./
root@node1:/chroot/thin/lib/x86_64-linux-gnu# ln -s libcrypt.so.1.1.0 libcrypt.so.1
root@node1:/chroot/thin/lib/x86_64-linux-gnu# ls -al
total 212
drwxr-xr-x 2 root root   4096 Jul 31 07:16 .
drwxr-xr-x 3 root root   4096 Jul 31 07:10 ..
lrwxrwxrwx 1 root root     17 Jul 31 07:16 libcrypt.so.1 -> libcrypt.so.1.1.0
-rw-r--r-- 1 root root 206776 Jul 31 07:16 libcrypt.so.1.1.0
```

> [!TIP]
> Repeat this step for each shared library except linux-vdso.so.1.

6. Import both root and nginx user to the thin chroot:

```bash
root@node1:~# mkdir /chroot/thin/etc
```

```bash
root@node1:~# grep root /etc/passwd > /chroot/thin/etc/passwd
root@node1:~# grep nginx /etc/passwd >> /chroot/thin/etc/passwd
root@node1:~# grep root /etc/group > /chroot/thin/etc/group
root@node1:~# grep nginx /etc/group >> /chroot/thin/etc/group
```

7. Create the link in /chroot/thin/opt:

```bash
root@node1:~# cd /chroot/thin/opt/
root@node1:/chroot/thin/opt# ln -s nginx-XXX nginx
```

8. Try to run it :

```bash
root@node1:~# chroot /chroot/thin /opt/nginx/sbin/nginx
```

9. Test from frontend

```bash
root@frontal:~# curl http://node1.lab.local
In thin chroot
```
