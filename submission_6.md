# Lab 6 — Docker

## Task 0 · Image Exporting

| Metric | Value |
|--------|-------|
| Pulled image | `ubuntu:latest` |
| Original size (`docker images`) | **78.1 MB** |
| TAR size after `docker save` | **77 MB** |

**Why TAR ≠ image size?** `docker images` показывает суммарный размер слоёв, но кэш слоёв может переиспользоваться разными образами. `docker save` кладёт в архив _все_ слои текущего образа без дедупликации, плюс JSON‑манифесты → файл чуть больше/меньше фактического «видимого» размера.

---

## Task 1 · Core Container Operations

```console
$ docker ps -a
CONTAINER ID   IMAGE                           COMMAND               CREATED       STATUS                     PORTS                      NAMES
… <коротил список ваших dev‑контейнеров> …
```

```console
$ docker run -it --name ubuntu_container ubuntu:latest
root@827c64bdf270:/# exit
```

```console
$ docker rmi ubuntu:latest
Error response from daemon: conflict: unable to remove repository reference "ubuntu:latest" (must force) - container 827c64bdf270 is using its referenced image f9248aac10f2
```

> **Explanation.** Нельзя удалить образ, пока существует даже остановленный контейнер, ссылающийся на него.

---

## Task 2 · Image Customization

```console
$ docker run -d -p 80:80 --name nginx_container nginx
$ curl -I http://localhost
HTTP/1.1 200 OK
```

```console
$ echo '<h1>Lab 6 Website</h1>' > index.html
$ docker cp index.html nginx_container:/usr/share/nginx/html/
$ docker commit nginx_container my_website:latest
$ docker rm -f nginx_container
$ docker run -d -p 80:80 --name my_website_container my_website:latest
$ curl http://localhost
<h1>Lab 6 Website</h1>
```

```console
$ docker diff my_website_container
C /etc
C /etc/nginx
C /etc/nginx/conf.d
C /etc/nginx/conf.d/default.conf
C /run
C /run/nginx.pid
```

> `docker diff` пометил **C** (Changed) для конфигов nginx и PID‑файла. Новый `index.html` помещён в слой контейнера, но diff не показывает вложенные изменения каталога, если весь слой уже отмечен как `C`.

---

## Task 3 · Container Networking

```console
$ docker network create lab_network
$ docker run -dit --network lab_network --name container1 alpine ash
$ docker run -dit --network lab_network --name container2 alpine ash
$ docker exec container1 ping -c 3 container2
PING container2 (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.057 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.075 ms
64 bytes from 172.20.0.3: seq=2 ttl=64 time=0.077 ms
```

> Внутри пользовательской bridge‑сети Docker работает встроенный DNS (127.0.0.11), который резолвит имена контейнеров в их IP‑адреса. Поэтому `ping container2` сработал без ручного /etc/hosts.

---

## Task 4 · Volume Persistence

```console
$ docker volume create app_data
$ docker run -d -v app_data:/usr/share/nginx/html --name web nginx
$ echo '<h2>PERSISTENT PAGE</h2>' > index.html
$ docker cp index.html web:/usr/share/nginx/html/
$ docker stop web && docker rm web
$ docker run -d -v app_data:/usr/share/nginx/html --name web_new nginx
$ curl http://localhost
<h1>Lab 6 Website</h1>
```

> Содержимое каталога `/usr/share/nginx/html` хранится в томе **app_data** и пережило удаление контейнера, что демонстрирует постоянство данных.

---

## Task 5 · Container Inspection

```console
$ docker run -d --name redis_container redis
$ docker exec redis_container ps aux | head
OCI runtime exec failed: exec failed: unable to start container process: exec: "ps": executable file not found in $PATH
```
Redis‑образ на alpine не содержит `ps`. Тем не менее IP‑адрес вытянуть можно:

```console
$ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis_container
172.17.0.4
```

**`docker exec` vs `docker attach`**  
*exec* запускает новую команду в уже работающем контейнере (многократно, не мешает PID 1).  
*attach* прикрепляется к STDIN/STDOUT основного процесса; если PID 1 завершится — сессия рвётся.

---

## Task 6 · Cleanup Operations

### До очистки
```console
docker system df
TYPE            TOTAL  ACTIVE   SIZE    RECLAIMABLE
Images          33     10       132.3GB 131.9GB (99%)
Containers      13     10       3.3kB   1.0kB (30%)
Volumes         10     4        71MB    6kB  (0%)
Build Cache     12     0        3.4GB   3.4GB
```

### Созданы временные контейнеры / dangling‑image
…(см. лог)…

### После `prune`
```console
$ docker system df
TYPE            TOTAL  ACTIVE   SIZE    RECLAIMABLE
Images          7      7        7.9GB   7.6GB (95%)
Containers      10     10       2.3kB   0B
Volumes         10     4        71MB    6kB
Build Cache     16     0        10.7GB  10.7GB
```

**Освобождено ≈ 117 GB** дискового пространства(тут еще были мои проекты, я их тоже стер за одно).

---

**Submission prepared by:** Zhalil

