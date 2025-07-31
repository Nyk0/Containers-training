# Using Compose

1. Prepare Compose directory.

```bash
root@node1:~# mkdir -p dockerfiles/compose
root@node1:~# cd dockerfiles/compose
root@node1:~/dockerfiles/compose# wget https://raw.githubusercontent.com/Nyk0/Containers-training/refs/heads/main/05-Docker-compose/compose.yml
```

2. Launch the compose deployment.

```bash
root@node1:~/dockerfiles/compose# docker compose up
```

3. Create a SSH tunnel from your computer.

```bash
ssh -p 2424 -N -f -L 127.0.0.1:8080:192.168.45.11:80 root@127.0.0.1
```
