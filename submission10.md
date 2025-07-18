
# Lab 10 — IPFS & 4EVERLAND Deployment Report

## 0 · Purpose

Demonstrate two things:

1. **Run a local IPFS node** under Docker, add a file, fetch it back through the gateway.  
2. **Publish a static web‑site straight from GitHub to IPFS/Filecoin** using free 4EVERLAND Hosting.

---

## 1 · Local IPFS node

```bash
# clean up any previous volumes / containers
$ docker volume rm ipfs_staging ipfs_data || true
$ docker rm -f ipfs_node || true

# start the daemon (kubo v0.36)
$ docker run -d --name ipfs_node \
    -v ipfs_staging:/export \
    -v ipfs_data:/data/ipfs \
    -p 4001:4001 -p 8080:8080 -p 5001:5001 \
    ipfs/kubo:latest

# wait until “Daemon is ready”
$ docker logs -f ipfs_node | grep -m1 'Daemon is ready'
```

> *IPFS Web UI* visible at <http://localhost:5001/webui> (see **screenshot 4**).

### 1.1 Add & fetch a file

```bash
# create small page
$ echo '<h1>Hello IPFS Lab 10</h1>' > index.html

# copy into the container
$ docker cp index.html ipfs_node:/export/

# pin & grab CID
$ CID=$(docker exec ipfs_node ipfs add -Q /export/index.html)
$ echo "CID = $CID"
```

Open <http://localhost:8080/ipfs/$CID> — shows the page (see **screenshot 6**).

---

## 2 · GitHub project

*Repo*: <https://github.com/PirmagomedovZhr/lab10>

Structure:

```
lab10/
 ├─ index.html
 └─ style.css         # optional
```

No build step; goal is pipeline, not UI.

---

## 3 · 4EVERLAND Hosting

1. **Login with GitHub** at <https://dashboard.4everland.org> (free tier).  
2. *Hosting ▸ Projects ▸ Create New*  
3. **Step 1** – select repo `lab10`.  
4. **Step 2** – config (see **screenshot 3**):

   * Hosting platform → **IPFS/Filecoin**  
   * Root dir `./`   Build command ⌀   Output dir `./`   Framework `Other`

5. Click **Deploy**.

4EVERLAND clones, skips npm install, pins files to IPFS, assigns domains, emits **Congratulations** modal (see **screenshot 2**).

Resulting URL (example):  
<https://lab10-y5qunok-pirmagomedovzhr.4everland.app>

Page is live (see **screenshot 1**). Another gateway form:  
`https://4everland-ipfs.com/ipfs/<CID>/index.html`

---

## 4 · Auto‑Rebuild Proof

```
$ sed -i 's/Hello/Hello from IPFS 🚀/' index.html
$ git commit -am "Update banner"
$ git push
```

Dashboard shows new build & CID; site updates within ~30 s.

---

## 5 · Key Findings

| Item | Evidence |
| ---- | -------- |
| Local node connected to ≥26 peers | screenshot 5 |
| File added and retrieved via local gateway | screenshot 6 |
| Project deployed on 4EVERLAND | screenshots 1‑3 |

**Benefits**

* **Content‑addressed hosting** — each commit → immutable CID.  
* Free, globally cached gateway delivered site without own server.  
* Local IPFS node useful for development & understanding pinning.

---  
© 2025 Zhalil  
