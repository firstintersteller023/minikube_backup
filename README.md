---

## 🧠 Problem

When you run

```bash
minikube delete
```

it **removes everything** — cluster, volumes, PVCs, and configs.
So if you don’t take a backup, **you lose all pods, services, and persistent data.**

---

## 🛟 Solutions (Safe Backup Options)

### 🧩 Option 1 — Backup your workloads (YAML)

Before deleting or restarting:

```bash
kubectl get all --all-namespaces -o yaml > all-resources-backup.yaml
kubectl get pvc,pv -A -o yaml >> all-resources-backup.yaml
kubectl get configmaps,secrets -A -o yaml >> all-resources-backup.yaml
```

✅ This saves **deployment manifests, PVC definitions, ConfigMaps, Secrets**, etc.

To restore later:

```bash
kubectl apply -f all-resources-backup.yaml
```

---

### 💾 Option 2 — Backup persistent volumes (data)

If your apps store data (e.g. MySQL, MongoDB), copy the data from Minikube’s Docker volume:

#### List Minikube containers:

```bash
docker ps | grep minikube
```

#### Access the container:

```bash
docker exec -it minikube bash
```

Inside, your persistent volumes live under:

```
/var/lib/minikube/volumes/
```

You can copy data out:

```bash
docker cp minikube:/var/lib/minikube/volumes/ ./minikube-volumes-backup/
```

To restore later:

```bash
docker cp ./minikube-volumes-backup/ minikube:/var/lib/minikube/volumes/
```

---

### 🗂️ Option 3 — Save an entire Minikube profile (snapshot)

Minikube uses Docker, so you can “snapshot” the entire VM/container:

```bash
docker commit minikube minikube-backup:latest
```

Later, restore it:

```bash
docker run -d --name minikube-restored minikube-backup:latest
```

⚠️ This keeps everything exactly as-is (stateful backup), but is **large in size**.

---

### 🛠️ Option 4 — Use Velero (pro-grade backup tool)

For a more production-like workflow:

```bash
velero install --provider aws --plugins velero/velero-plugin-for-aws:v1.6.0 --bucket <bucket-name> --backup-location-config region=minikube
```

Then back up and restore your cluster declaratively:

```bash
velero backup create minikube-backup
velero restore create --from-backup minikube-backup
```

---

## ⚡ Recommended combo for Dev use

For Minikube/local work:

1. **Before delete →** `kubectl get all -A -o yaml > backup.yaml`
2. **For DBs →** `docker cp` the volume directory.
3. **Optional full snapshot →** `docker commit minikube minikube-backup:latest`

---

