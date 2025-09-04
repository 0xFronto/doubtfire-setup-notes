
````markdown
# Doubtfire Setup & Common Issues (Kali VM / Docker)

---

## Folder and Git Setup

```bash
┌──(kali㉿kali)-[~]
└─$ mkdir ~/doubtfire && cd ~/doubtfire
git clone https://github.com/thoth-tech/doubtfire-deploy.git
cd doubtfire-deploy
git checkout 9.x
````

**Output (expected):**

```
Cloning into 'doubtfire-deploy'...
remote: Enumerating objects: 1488, done.
remote: Counting objects: 100% (703/703), done.
remote: Compressing objects: 100% (128/128), done.
remote: Total 1488 (delta 607), reused 575 (delta 575), pack-reused 785 (from 2)
Receiving objects: 100% (1488/1488), 307.82 KiB | 2.70 MiB/s, done.
Resolving deltas: 100% (771/771), done.
branch '9.x' set up to track 'origin/9.x'.
Switched to a new branch '9.x'
```

![Git Setup](https://raw.githubusercontent.com/0xFronto/doubtfire-setup-notes/main/docs/images/git-setup.png)

---

## Issue – Ruby Version Mismatch (Skip if irrelevant)

During build:

```
Your Ruby version is 3.1.7, but your Gemfile specified ~> 3.3.0
```

**Fix:** Edit `doubtfire-api/Dockerfile` → change:

```dockerfile
FROM ruby:3.1-bullseye
```

to:

```dockerfile
FROM ruby:3.3-bullseye
```

![Gemfile Fix](https://raw.githubusercontent.com/0xFronto/doubtfire-setup-notes/main/docs/images/gemfile-fix.png)
![Ruby Mismatch](https://raw.githubusercontent.com/0xFronto/doubtfire-setup-notes/main/docs/images/ruby-mismatch.png)

---

## Rebuild

```bash
docker-compose -f development/docker-compose.yml build --no-cache
docker-compose -f development/docker-compose.yml up
```

---

## Problem – Migration / DB State

API container fails with MySQL migration errors (indexes missing).

![Migration Error](https://raw.githubusercontent.com/0xFronto/doubtfire-setup-notes/main/docs/images/migration-error.png)

---

## Fix – Clean Reset

Inside `~/doubtfire/doubtfire-deploy`:

```bash
docker-compose -f development/docker-compose.yml down -v
docker-compose -f development/docker-compose.yml up --build
```

This wipes volumes (databases) and rebuilds fresh.

---

## Performance Issues – Extreme Latency

After starting successfully, the VM froze / lagged heavily. Cause:

* Angular + Rails + MySQL + Redis + Overseer containers are **very resource intensive**.
* Running with only **2 GB RAM** in Kali VM made the system unusable.

![VM Latency](https://raw.githubusercontent.com/0xFronto/doubtfire-setup-notes/main/docs/images/vm-latency.png)

---

## Fix – Increase VM Resources

* Increased VM memory from **2 GB → 8 GB**.
* 4 GB may work, but **2 GB is too low**.
* Recommended (Example if you have 64 GB of RAM):

  * **RAM:** 8–12 GB
  * **CPUs:** 4–6
  * **Disk:** 80 GB+

![Resource Upgrade](https://raw.githubusercontent.com/0xFronto/doubtfire-setup-notes/main/docs/images/resource-upgrade.png)

---

After this, system built and ran at `http://localhost:4200` (web) and `http://localhost:3000` (API).

---

![Final Success](https://raw.githubusercontent.com/0xFronto/doubtfire-setup-notes/main/docs/images/final-success.png)

```

---

