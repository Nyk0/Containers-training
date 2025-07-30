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
