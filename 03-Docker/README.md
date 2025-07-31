# Nginx in Docker

1. Follow instructions here:

https://docs.docker.com/engine/install/debian/

2. Start a screen session.

3. Run bash in your first container:

```bash
root@node1:~# docker run -it debian bash
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
ebed137c7c18: Pull complete
Digest: sha256:b6507e340c43553136f5078284c8c68d86ec8262b1724dde73c325e8d3dcdeba
Status: Downloaded newer image for debian:latest
root@4e7a34f764d9:/#
```

In the meantime:

```bash
root@node1:~# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
4e7a34f764d9   debian    "bash"    4 minutes ago   Up 4 minutes             condescending_cannon
```

```bash
root@node1:~# docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
4e7a34f764d9   debian        "bash"     4 minutes ago   Up 4 minutes                         condescending_cannon
11c0b3ca5f94   hello-world   "/hello"   6 minutes ago   Exited (0) 6 minutes ago             blissful_gould
```bash

```bash
root@node1:~# docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
debian        latest    c93961f83a1e   10 days ago    117MB
hello-world   latest    74cc54e27dc4   6 months ago   10.1kB
```

4. Containers and images.

```bash
root@4e7a34f764d9:/# echo "hello" > toto.text
root@4e7a34f764d9:/# cat toto.text
hello
root@4e7a34f764d9:/# exit
exit
```

```bash
root@node1:~# docker run -it debian bash
root@286ed6a1000f:/# cat toto.text
cat: toto.text: No such file or directory
root@286ed6a1000f:/# exit
exit
root@node1:~# docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
286ed6a1000f   debian        "bash"     3 minutes ago    Exited (0) 3 seconds ago              xenodochial_torvalds
4e7a34f764d9   debian        "bash"     10 minutes ago   Exited (0) 3 minutes ago              condescending_cannon
11c0b3ca5f94   hello-world   "/hello"   12 minutes ago   Exited (0) 12 minutes ago             blissful_gould
root@node1:~# docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
debian        latest    c93961f83a1e   10 days ago    117MB
hello-world   latest    74cc54e27dc4   6 months ago   10.1kB
```

```bash
root@node1:~# docker start -ai 4e7a34f764d9
root@4e7a34f764d9:/# cat toto.text
hello
```

5. Network backend

```bash
root@node1:~# iptables -L -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  anywhere            !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        anywhere

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
```

```bash
root@node1:~# ip a
...
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether ce:3d:e7:6a:a2:33 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::cc3d:e7ff:fe6a:a233/64 scope link
       valid_lft forever preferred_lft forever
```

6. Clean all.

```bash
root@node1:~# docker system prune -af
Deleted Containers:
92fdba480cf028190ba17c23a6d409076e86d29d505e08f2f8ae1e4fbd422038
286ed6a1000f61335deae20ac5db11512189adee88d26855f745f13c44c3db15
4e7a34f764d9a1089e7508084a5cf8b8ce13745dc73ff764e681e32c8abee480
11c0b3ca5f94447d1ba016901d33ebf30409a1aebe693ab9366d89ad587a4e6a

Deleted Images:
untagged: hello-world:latest
untagged: hello-world@sha256:ec153840d1e635ac434fab5e377081f17e0e15afab27beb3f726c3265039cfff
deleted: sha256:74cc54e27dc41bb10dc4b2226072d469509f2f22f1a3ce74f4a59661a1d44602
deleted: sha256:63a41026379f4391a306242eb0b9f26dc3550d863b7fdbb97d899f6eb89efe72
untagged: debian:latest
untagged: debian@sha256:b6507e340c43553136f5078284c8c68d86ec8262b1724dde73c325e8d3dcdeba
deleted: sha256:c93961f83a1e578ef9b2105f1718e307c2e1c7811ba540c6e7176aa3e12752d7
deleted: sha256:175a198361755136c75d537bc9057c9c8e1b7a20eb771374a8339eaadfef29e9

Total reclaimed space: 116.6MB
```

7. Basic Layer 3 connections between containers.

Run two containers:

```bash
root@node1:~# docker run -it debian /bin/bash
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
ebed137c7c18: Pull complete
Digest: sha256:b6507e340c43553136f5078284c8c68d86ec8262b1724dde73c325e8d3dcdeba
Status: Downloaded newer image for debian:latest
root@e203d62f0297:/#
```

```bash
root@node1:~# docker run -it debian /bin/bash
root@55a50ce12d7a:/#
```

```bash
root@node1:~# docker ps -q | xargs -n 1 docker inspect --format '{{ .NetworkSettings.IPAddress }} {{ .Name }} {{ .Config.Hostname }}' | sed 's/ \// /'
172.17.0.3 agitated_lewin 55a50ce12d7a
172.17.0.2 jolly_wilson e203d62f0297
```

From a container:

```bash
root@55a50ce12d7a:/# apt-get update
root@55a50ce12d7a:/# apt-get install iputils-ping
root@55a50ce12d7a:/# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.091 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.065 ms
```

From the host:

```bash
root@node1:~# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.104 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.102 ms
```

8. Basic Layer 4 connections to a container.

Run in foreground:

```bash
root@node1:~# docker run nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
59e22667830b: Pull complete
140da4f89dcb: Pull complete
96e47e70491e: Pull complete
2ef442a3816e: Pull complete
4b1e45a9989f: Pull complete
1d9f51194194: Pull complete
f30ffbee4c54: Pull complete
Digest: sha256:84ec966e61a8c7846f509da7eb081c55c1d56817448728924a87ab32f12a72fb
Status: Downloaded newer image for nginx:latest
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/07/31 10:05:04 [notice] 1#1: using the "epoll" event method
2025/07/31 10:05:04 [notice] 1#1: nginx/1.29.0
2025/07/31 10:05:04 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14+deb12u1)
2025/07/31 10:05:04 [notice] 1#1: OS: Linux 6.1.0-37-amd64
2025/07/31 10:05:04 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/07/31 10:05:04 [notice] 1#1: start worker processes
2025/07/31 10:05:04 [notice] 1#1: start worker process 29
```

```bash
root@node1:~# docker inspect --format='{{ .NetworkSettings.IPAddress }} {{ .Config.ExposedPorts }}' de3fc32db493
172.17.0.2 map[80/tcp:{}]
root@node1:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
de3fc32db493   nginx     "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes   80/tcp    beautiful_elgamal
```

```bash
root@node1:~# curl 172.17.0.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

9. Exposing container's layer 4 to the host interfaces.

Exit and restart nginx container like this:

```bash
root@node1:~# docker run -P nginx
```

```bash
root@node1:~# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                       NAMES
e1ab6761c05f   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:32768->80/tcp, [::]:32768->80/tcp   gracious_elgamal
```

```bash
root@node1:~# netstat -laputn | grep 32768
tcp        0      0 0.0.0.0:32768           0.0.0.0:*               LISTEN      3813/docker-proxy
tcp6       0      0 :::32768                :::*                    LISTEN      3818/docker-proxy
```

Test from a remote host:

```bash
root@frontal:~# curl http://node1.lab.local:32768
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

Exit and restart nginx container like this:

```bash
root@node1:~# docker run --network="host" nginx
```

```bash
root@node1:~# netstat -laputn | grep nginx
tcp  0  0 0.0.0.0:80  0.0.0.0:*  LISTEN  4000/nginx: master
tcp6  0  0  :::80  :::*  LISTEN  4000/nginx: master
```

Test from a remote host:

```bash
root@frontal:~# curl http://node1.lab.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```
