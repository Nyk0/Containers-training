# Part 1: Nginx compilation

1. Get Nginx at http://nginx.org/en/download.html.

2. Decompress and untar it (tar -xzf nginx-XXX.tar.gz).

> [!TIP]
> In this command, XXX refers to the current nginx version.

3. Install the "build-essential" package.

```bash
apt-get install screen
```

4. Move to the Nginx sources directory.

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
./configure --prefix=/opt/nginx-XXX \
  --user=nginx --group=nginx \
  --without-http_rewrite_module \
  --without-http_gzip_module
```

7. Launch compilation:

```bash
make
```

8. Install the software according to the "--prefix" of step 6.

```bash
make install
```

9. Create the dedicated system account:

```bash
useradd -r -d /var/empty -s /bin/false nginx
```

10. Create a symlink to keep a consistant path to Nginx through version upgrades:

```bash
cd /opt
ln -s nginx-XXX nginx
```

11. Browse Nginx executable help:

```bash
/opt/nginx/sbin/nginx -h
```

12. Add “daemon off;” to nginx.conf to prevent daemonization.

13. Launch nginx:

```bash
/opt/nginx/sbin/nginx
```

14. Test Nginx from the frontend:

```bash
apt-get install curl
curl http://node1.lab.local
```

# Part 2: Fat chroot

1. Create a directory to create the fat Nginx chroot:

```bash
mkdir -p /chroot/fat
```

2. Install and use debootstrap to put a stable Debian system in a target directory:

```bash
apt-get install debootstrap
debootstrap stable /chroot/fat/
```

4. Copy our /opt/nginx-XXX to /chroot/fat/opt/

```bash
cp -r /opt/nginx-XXX to /chroot/fat/opt/
```

5. Change the content of index.html of the chrooted Nginx:

```bash
echo "In chroot" > /chroot/fat/opt/nginx-XXX/html/index.html
```

7. Use chroot to chroot a bash in /chroot/fat :
chroot /chroot/fat /bin/bash
8. Check the content of /opt/nginx-1.22.0/html/index.html.
9. Re-create the symlink inside the fat chroot.
10. Re-create the nginx system account inside the fat chroot.
11. Exit the fat chroot and launch a chrooted nginx :
chroot /chroot/fat /opt/nginx/sbin/nginx
12. Get the UID used by nginx. Is there a problem ?
13. Delete the nginx user in the chroot and create a new one with another UID :
groupadd -g 3000 nginx
useradd -r -d /var/empty -s /bin/false -u 3000 -g 3000 nginx
```
