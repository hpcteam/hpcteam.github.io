# Hyper-Threading Theory and HPC Implications

---

##  What is Hyper-Threading?

- Hyper-Threading (HT) is **Intel’s implementation of Simultaneous Multithreading (SMT)**.
- It allows **one physical CPU core** to appear as **two logical cores** to the operating system.
- Each logical core can handle **independent threads** simultaneously.
- Purpose: **better utilization of CPU resources** that would otherwise remain idle.

**Analogy:**  
Think of a chef (CPU core) who can cook **one dish at a time**. Normally, if the chef is waiting for water to boil (idle), nothing happens. With Hyper-Threading, the chef can start preparing a second dish while waiting — making the overall kitchen (CPU) more efficient.

---

##  How it works (simplified)

- Each physical core has:
  - Execution units (ALUs, FPUs)
  - Cache memory
- When one thread stalls (waiting for memory, I/O, etc.), the **other thread can use the execution units**.
- The operating system sees **twice as many cores**, but performance **does not double**.

---

##  Advantages of Hyper-Threading

| Advantage | Explanation |
|-----------|------------|
| Better CPU utilization | Idle execution units can be used by a second thread. |
| Improved throughput | More threads can run concurrently. |
| Multi-tasking efficiency | Useful for running multiple applications simultaneously. |
| Cost-effective performance | Acts like adding cores **without buying more physical cores**. |

---

##  Disadvantages / Limitations

| Disadvantage | Explanation |
|-------------|------------|
| Performance not doubled | Two threads share the **same physical resources**. |
| Can increase latency | Threads may compete for CPU resources. |
| Security concerns | Some side-channel attacks exploit HT (e.g., Spectre/Meltdown). |
| Not always beneficial for HPC | Certain HPC applications need **dedicated cores** for consistent performance. |

---

##  Hyper-Threading in HPC Environments

HPC workloads are **compute-intensive** and often **parallelized** using MPI/OpenMP. They usually:

- Use **floating-point-heavy calculations**
- Require **predictable, consistent performance**
- Run on clusters with **many nodes/cores**

### Effect of Hyper-Threading in HPC

| Scenario | Effect |
|----------|--------|
| CPU-bound, floating-point heavy | HT may **not help**; can slightly reduce performance because threads compete for execution units. |
| Memory-bound workloads | HT can help utilize idle CPU cycles while waiting for memory, giving **small performance gains**. |
| Latency-sensitive applications | HT can **increase variability** and **slow down tight communication loops**. |
| Large MPI jobs | Often better to **disable HT** for consistent core-to-core performance. |

---

## General HPC Best Practices

1. **Check your workload type**
   - CPU-intensive → HT off
   - I/O or memory-bound → HT can be left on
2. **Benchmark your applications**
   - Run with HT enabled and disabled to see the effect.
3. **Cluster scheduling**
   - Schedulers (Slurm, PBS) treat logical cores differently.
   - HT-enabled CPUs may require configuring **threads per core** carefully.
4. **Energy considerations**
   - HT may slightly increase power consumption.
5. **Security**
   - In multi-tenant environments, **disabling HT can improve isolation**.

---

##  When to Enable / Disable HT

| Use Case | Recommendation |
|----------|----------------|
| HPC scientific computing (CPU-heavy) | **Disable** Hyper-Threading |
| Multi-tasking / server virtualization | **Enable** Hyper-Threading |
| Mixed workloads | Test & benchmark; enable only if performance improves |

---

##  Summary

- HT is **not magic**; it doesn’t double performance.
- It’s **good for throughput**, not latency-critical HPC tasks.
- In HPC clusters, **most admins disable HT** to ensure:
  - Predictable performance
  - Consistent benchmarking
  - Efficient resource scheduling

### **Enable and Disable Hyper-Threading (Linux)**
---

## Enable / Disable Hyper-Threading Manually (BIOS / iDRAC)

> ⚠️ **IMPORTANT**
>
> * The server **will reboot**
> * Ensure the node is **isolated**
> * Make sure **no jobs are running**

---


## Check Hyper-Threading Status

Run the following command:

```
lscpu | grep -E "Thread|Core|Socket"


### Interpretation

* **Thread(s) per core: 2** → Hyper-Threading is **enabled**
* **Thread(s) per core: 1** → Hyper-Threading is **disabled**
```
---

### Using `nproc`

If the server has **192 physical cores**, run:

```bash
nproc
```

* Output **greater than 192** → Hyper-Threading is **enabled**
* Output **equal to 192** → Hyper-Threading is **disabled**

---

## Accessing iDRAC

### Option 1: Network Access

If the server is accessible on the local network:

* Open a browser
* Access iDRAC using:

  ```
  https://<server-ip>
  example:
  https://168.192.1.1
  ```

### Option 2: Direct USB Access

If the server is not reachable on the network:

* Connect the server to a laptop using a **USB-B cable**
* Access the iDRAC console locally

---

## Steps to Enable / Disable Hyper-Threading

1. Log in to the iDRAC console
2. Navigate to:

   ```
   Configuration → BIOS → Processor Settings
   ```
3. Locate the following options:

   * **Logical Processor**
   * **Virtualization Technology**

### Configuration Meaning

* **Logical Processor = Enabled** → Hyper-Threading **enabled**
* **Logical Processor = Disabled** → Hyper-Threading **disabled**

4. Save the changes
5. Reboot the server to apply the configuration

---

## Notes

* BIOS-level configuration is **persistent**
* Recommended for **production HPC environments**
* OS-level SMT changes are temporary and reset on reboot
