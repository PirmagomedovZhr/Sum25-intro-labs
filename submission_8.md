# Lab 8 — SRE Lab

## Task 1 · Key Metrics & Disk Management

### 1.1 Resource‑intensive processes

```console
$ ps -eo pid,comm,%cpu --sort=-%cpu | head -n 4
    PID COMMAND         %CPU
  11201 chrome           62.2
   4849 gnome-shell      10.4
   5687 firefox           8.4

$ ps -eo pid,comm,%mem --sort=-%mem | head -n 4
    PID COMMAND         %MEM
   5687 firefox           4.9
  11201 chrome            3.1
   4849 gnome-shell       2.8
```

`chrome` (renderer) держит > 60 % CPU — открыто множество вкладок; `gnome-shell` рендерит Wayland‑сессию, `firefox` занимает до 5 % RAM.

```console
$ iostat -xz 1 3          # обрезано до итогов
avg-cpu:  %user %system %iowait %idle
           5.53    0.38    0.13  93.96
Device            r/s    rkB/s   %util
nvme0n1            0      0.3    0.30
```

I/O‑ожидание < 1 %; дисковых «узких мест» нет.

### 1.2 Disk‑usage deep dive

```console
$ sudo du -h /var | sort -h | tail -n 10
3.8G /var/lib/docker/overlay2/2af804…
6.1G /var/lib/snapd/cache
7.1G /var/lib/snapd
88G  /var/lib/docker
95G  /var/lib
96G  /var

$ sudo find /var -type f -printf '%s %p\n' | sort -nr | head -n 3
3349475328 /var/lib/docker/overlay2/yqju4…/diff/app/trains.db
3349475328 /var/lib/docker/overlay2/ssmsk…/diff/trains.db
3349475328 /var/lib/docker/overlay2/jzkh0…/diff/app/trains.db
```

*Главные «пожиратели» места* — слои Docker‑образов: три копии `app/trains.db` по \~3.1 GB каждая. Логи systemd‑journal занимают < 700 MB.

**Действия:** prune неиспользуемые Docker‑образы (`docker image prune -a`), вынести БД в отдельный volume, уменьшить snap‑кэш.

---

## Task 2 · Checkly Monitoring

Мониторинг настроен в Checkly (Free trial).

- **URL monitor**: GET `https://example.com`  →  **200 OK** за \~457 ms. Assertion «Status code equals 200» установлен, запуск каждые 5 мин.
- Alert правило: e‑mail при **Failed check** или **Latency > 1000 ms**.

### Скриншоты (загружены на Moodle)

&#x20;&#x20;

---

## Выводы

- CPU‑peaks дают браузеры Chrome/Firefox; среднее idle > 90 %.
- `/var` разросся до 96 GB — >90 % из‑за Docker‑слоёв; очистка даёт мгновенный выигрыш.
- Checkly обеспечивает внешнее SLA‑наблюдение; первый сигнал об ошибке придёт < 5 мин, что покрывает целевой SLO 99.9 %.

**Submission prepared by:** Zhalil

