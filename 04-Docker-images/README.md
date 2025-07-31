# Building custom images

1. Create a fat image.

```bash
root@node1:~# mkdir -p dockerfiles/mynginx-debian
root@node1:~# cd dockerfiles/mynginx-debian
```

```bash
root@node1:~/dockerfiles/mynginx-debian# cat Dockerfile
FROM debian:latest

COPY nginx-1.28.0 /opt/nginx-1.28.0

RUN ln -s /opt/nginx-1.28.0 /opt/nginx && \
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

2. 

