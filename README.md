# 📦 GlusterFS Distributed Storage Setup for Oracle RMAN Backup

## 🧠 Project Overview
This project demonstrates how to build a **high-availability distributed storage system** using:

- LVM (Logical Volume Manager)
- XFS filesystem
- GlusterFS (Replica + Arbiter)

The storage is designed for:
- Oracle RMAN backup storage
- Data redundancy
- Split-brain prevention using arbiter node

---

## 🏗️ Architecture
```bash
                +----------------------+
                |   Gluster Volume     |
                |  gv_rmancc_rep_arb   |
                +----------+-----------+
                           |
      ---------------------------------------------
      |                 |                 |
Ytlcbak01          Ytlcbak02         Ytlcbak03
 (Data Brick)       (Data Brick)      (Arbiter)
 /rmancc1           /rmancc1          /rmancc1

```

- Replica 2 + Arbiter 1
- Prevents split-brain while saving storage space

---

## ⚙️ Implementation Steps

### 1️⃣ Create LVM Volume

```bash
sudo lvcreate -L 1.5T -n lvrmancc1 vg01
```
2️⃣ Format with XFS
```bash
sudo mkfs.xfs /dev/vg01/lvrmancc1
```
3️⃣ Mount Filesystem
```bash
sudo mkdir -p /rmancc1
sudo mount /dev/vg01/lvrmancc1 /rmancc1
```
4️⃣ Prepare Gluster Brick Directory
```bash
sudo mkdir -p /rmancc1/replica1
sudo chmod 777 /rmancc1/replica1
```
⚠️ Note: In production, avoid using 777. Use proper user/group permissions.

5️⃣ Add Bricks to Gluster Volume
```bash
sudo gluster volume add-brick gv_rmancc_rep_arb replica 2 arbiter 1 \
Ytlcbak01:/rmancc1/replica1 \
Ytlcbak02:/rmancc1/replica1 \
Ytlcbak03:/rmancc1/arbiter1
```
Replica 2 → Data redundancy

Arbiter 1 → Prevent split-brain (stores metadata only)

6️⃣ Rebalance Volume
```bash
sudo gluster volume rebalance gv_rmancc_rep_arb start
sudo gluster volume rebalance gv_rmancc_rep_arb status
```
Ensures data is properly distributed across nodes.

🚀 Key Skills Demonstrated
Linux Storage Management (LVM)
Filesystem Optimization (XFS)
Distributed Storage (GlusterFS)
High Availability Design (Replica + Arbiter)
Production Deployment Workflow

💡 Real-World Use Case
Oracle RMAN backup repository
Telecom billing system backup (OCS / ZSmart)
Large-scale archive storage
