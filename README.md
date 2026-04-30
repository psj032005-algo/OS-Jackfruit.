# 🧠 Mini Container Runtime with Kernel Monitoring

## 📌 Overview
This project implements a lightweight container runtime in C with a supervisor process and a kernel module for memory monitoring.

---

## ⚙️ Features

- Multi-container support (alpha, beta)
- CLI commands (`run`, `ps`, `logs`)
- Supervisor using UNIX sockets (IPC)
- Container logging
- Kernel module for memory monitoring
- Soft and Hard memory limits
- CPU scheduling experiment

---

## 🏗️ System Design

- **engine.c** → CLI + Supervisor
- **monitor.c** → Kernel module
- **IPC** → UNIX domain socket (`/tmp/mini_runtime.sock`)
- **Containers** → Isolated processes using rootfs

---

## 🛠️ Setup

### 1. Clone Repository
```bash
git clone https://github.com/YOUR-USERNAME/OS-Jackfruit.git
cd OS-Jackfruit/boilerplate
```

## 🚀 Build

```
make clean
make
```
Setup Root Filesystem

Download Alpine rootfs:
```
wget https://dl-cdn.alpinelinux.org/alpine/latest-stable/releases/x86_64/alpine-minirootfs.tar.gz
tar -xvf alpine-minirootfs.tar.gz
```

Run Supervisor

commands
```
sudo ./engine supervisor ~/rootfs-base
```
in terminal 1
```
sudo ./engine run alpha ~/rootfs-alpha /bin/sh
sudo ./engine run beta ~/rootfs-beta /bin/sh
```
in terminal 1
```
sudo ./engine ps
```
Screenshot 1: Multi-container
Two containers running (alpha & beta)
Command: engine ps
<img width="1156" height="285" alt="WhatsApp Image 2026-04-18 at 6 38 29 PM" src="https://github.com/user-attachments/assets/72fe3f7c-ba3e-42fd-8f9f-360364ecdf8a" />

📸 Screenshot 2: Metadata tracking (PID and status of running containers)
<img width="892" height="142" alt="2" src="https://github.com/user-attachments/assets/0dd49092-8dd4-4f6a-bcf4-36115f2e0cb3" />
commands

```
sudo ./engine supervisor ~/rootfs-base
```
in terminal 1
```
sudo ./engine run alpha ~/rootfs-alpha /bin/sh
sudo ./engine run beta ~/rootfs-beta /bin/sh
```
in terminal 1
```
sudo ./engine ps
```
📸 Screenshot 3: Logging output for container alpha
<img width="1186" height="280" alt="3" src="https://github.com/user-attachments/assets/3cc53b78-46d8-43cb-b0ab-4daac0782bc9" />

commands
```
sudo pkill engine
```
```
sudo rm -f /tmp/mini_runtime.sock
```
```
sudo ./engine supervisor ~/rootfs-base
```
```
sudo ./engine run alpha ~/rootfs-alpha /bin/sh
```
```
sudo ./engine logs alpha
```
📸 Screenshot 4: CLI interacting with supervisor via IPC (UNIX socket)
<img width="1217" height="371" alt="4" src="https://github.com/user-attachments/assets/ac0cbfa8-f7a9-440a-a010-f6bdb33e0dc8" />
commands
```
sudo ./engine supervisor ~/rootfs-base
```
```
sudo ./engine run alpha ~/rootfs-alpha /bin/sh
sudo ./engine run alpha ~/rootfs-alpha /bin/sh
```
```
sudo ./engine ps
```
📸 Screenshot 5: Kernel module detecting SOFT memory limit breach
<img width="1219" height="350" alt="5" src="https://github.com/user-attachments/assets/8747f92d-a188-4dbb-bbb5-76ae7d1ef858" />
commands
```
make clean
make
```
```
sudo rmmod monitor   # ignore error if not loaded
sudo insmod monitor.ko
```
```
ls /dev | grep monitor
```
```
sudo ./engine supervisor ~/rootfs-base
```
```
cp memory_hog ~/rootfs-alpha/
```
```
sudo ./engine run alpha ~/rootfs-alpha ./memory_hog
```
```
sudo dmesg | tail
```
📸 Screenshot 6: Kernel enforcing HARD memory limit (process terminated)
<img width="1223" height="313" alt="6" src="https://github.com/user-attachments/assets/62f7591c-1893-4926-963e-04e78f686d8e" />

```
clear
sudo dmesg | tail
```
📸 Screenshot 7: CPU-bound vs I/O-bound scheduling using top

<img width="1113" height="666" alt="71" src="https://github.com/user-attachments/assets/d3d9185b-c945-4030-9ca0-b579eae15919" />
<img width="1113" height="666" alt="72" src="https://github.com/user-attachments/assets/7b8f0c7e-8fce-478b-967d-a9bca3e62f14" />

```
gcc -O2 -o cpu_hog cpu_hog.c
```
```
./cpu_hog 100 &
./io_pulse &
```
```
top -o %CPU
```
📸 Screenshot 8: Clean teardown (no running processes or zombies)
<img width="1115" height="285" alt="WhatsApp Image 2026-04-18 at 9 34 05 PM" src="https://github.com/user-attachments/assets/293f2410-1300-4888-a5d0-0cc03d652a77" />

```
sudo pkill -9 engine
```
```
pkill cpu_hog
pkill io_pulse
```
```
sudo rm -f /tmp/mini_runtime.sock
```
```
ps aux | grep engine
```

---

## 🧪 Observations

- Containers are implemented as isolated processes using separate root filesystems.
- The supervisor manages container lifecycle using IPC via UNIX domain sockets.
- The `engine ps` command correctly tracks container metadata such as PID and status.
- Logging functionality captures container activity in real time.
- The kernel module successfully monitors memory usage of processes.

---

## ⚠️ Memory Limit Behavior

- **Soft Limit**:  
  When memory usage exceeds the soft threshold, a warning is logged:
  [container_monitor] SOFT LIMIT container=alpha ...
  
- **Hard Limit**:  
When memory usage exceeds the hard threshold, the process is terminated:
[container_monitor] HARD LIMIT container=alpha ...
Out of memory: killed process


---

## ⚡ Scheduling Experiment Analysis

- `cpu_hog` is a CPU-bound process that continuously performs computations.
- `io_pulse` is an I/O-bound process that performs periodic operations and sleeps.

### Observed Behavior:
- `cpu_hog` consumes nearly 100% CPU.
- `io_pulse` consumes minimal CPU.

### Reason:
CPU-bound processes utilize CPU continuously, while I/O-bound processes frequently yield CPU during I/O waits. The scheduler allocates more CPU time to CPU-bound processes.

---

## 🧹 Cleanup Verification

- All running containers and supervisor processes were terminated.
- No residual or zombie processes were found in the system.

---

## 🧠 Key Learnings

- Understanding of containerization using Linux process isolation
- Use of IPC (UNIX sockets) for communication
- Kernel module development and integration
- Memory monitoring at kernel level
- Difference between CPU-bound and I/O-bound scheduling

---

## 📌 Conclusion

This project successfully demonstrates the design and implementation of a lightweight container runtime along with kernel-level monitoring. It provides practical insights into operating system concepts such as process management, scheduling, and memory control.

---




