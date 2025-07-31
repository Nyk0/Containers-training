# Building custom images

1. Create a fat image.

```bash
root@node1:~# mkdir -p dockerfiles/mynginx-debian
root@node1:~# cd dockerfiles/mynginx-debian
root@node1:~/dockerfiles/mynginx-debian# cp -pr /chroot/fat/opt/nginx-XXX ./
```

```bash
root@node1:~/dockerfiles/mynginx-debian# cat Dockerfile
FROM debian:latest

COPY nginx-XXX /opt/nginx-XXX

RUN ln -s /opt/nginx-XXX /opt/nginx && \
  useradd -r -d /var/empty -s /bin/false nginx

EXPOSE 80

CMD ["/opt/nginx/sbin/nginx"]
```

```bash
root@node1:~/dockerfiles/mynginx-debian# docker build -t mynginx:1 .
root@node1:~/dockerfiles/mynginx-debian# docker run mynginx:1
```

```bash
root@node1:~/dockerfiles/mynginx-debian# docker ps -a
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS     NAMES
07bbe5aab852   mynginx:1   "/opt/nginx/sbin/ngi…"   5 minutes ago   Up 5 minutes   80/tcp    agitated_heyrovsky
```

```bash
root@node1:~/dockerfiles/mynginx-debian# curl 172.17.0.2
In fat chroot
```

2. Create a thin image.

```bash
root@node1:~# mkdir -p dockerfiles/mynginx-scratch
root@node1:~# cd dockerfiles/mynginx-scratch
root@node1:~/dockerfiles/mynginx-scratch# cp -pr /chroot/thin ./
```

```bash
root@node1:~/dockerfiles/mynginx-scratch# cat Dockerfile
FROM scratch

COPY thin /

EXPOSE 80

CMD ["/opt/nginx/sbin/nginx"]
```

```bash
root@node1:~/dockerfiles/mynginx-scratch# docker build -t mynginxthin:1 .
root@node1:~/dockerfiles/mynginx-scratch# docker run mynginxthin:1
```

```bash
root@node1:~/dockerfiles/mynginx-scratch# curl 172.17.0.2
In thin chroot
```

3. Images size.

```bash
root@node1:~# docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
mynginxthin   1         106518801416   8 minutes ago    6.22MB
mynginx       1         31d89c6ac8ab   20 minutes ago   120MB
```
