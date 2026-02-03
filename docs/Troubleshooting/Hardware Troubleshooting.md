# **Hardware Troubleshooting – HPC / Data Center Nodes**
---

## 1. Identify Scope and Impact (Using OME)

Before taking any action, clearly identify the **scope and impact** of the issue.

### Determine the Level of Impact

* **Node-level** issue
* **Rack-level** issue
* **Cluster-level** issue
* Check whether **running jobs** are impacted

⚠️ **Important:**
If jobs are running on the node, **do not touch the node** until jobs are completed.

---

## 2. Check Workload Manager Status (PBS Pro)

First, verify node and job status from the scheduler.

### Commands

```bash
qstat -aw
pbsnodes -aSj | grep <node-hostname>
```

### What You Can Identify

* Node state (free, busy, offline)
* Jobs running on the node
* Job owner and job ID

---

## 3. Handling Nodes with Running Jobs

If the node is hosting any jobs:

1. **Do not troubleshoot immediately**
2. Wait until job completion
3. Inform the **respective users / higher authorities**
4. After job completion, proceed with troubleshooting

### Put Node in Offline Mode

```bash
pbsnodes -o <node-hostname> -C "hardware issue"
```

✔ This prevents new jobs from being scheduled on the node.

---

## 4. Check Cluster Health Status (Bright Cluster Manager)

Use **Bright Cluster Manager (BCM)** to verify node health.

### Using `cmsh`

```bash
cmsh
device use <node-hostname>
status
```

Or:

```bash
cmsh
device list | grep <node-hostname>
```

### Purpose

* Identify node health status
* Detect cluster-wide issues

---

## 5. Centralized Hardware Alert Check (OME)

**OpenManage Enterprise (OME)** acts as the **single source of truth** for Dell server hardware issues.

### OME Provides Alerts For

* CPU
* DIMM (Memory)
* PSU
* FAN
* Thermal warnings
* Power supply redundancy
* Firmware mismatches
* Lifecycle controller logs

---

## 6. iDRAC Access for Node-Level Diagnosis

Using OME, access the **iDRAC console** of the affected node.

### Using iDRAC, You Can Check

* PSU status
* Fan status
* Inlet temperature
* Fan RPMs
* Node-level patterns
* Port status
* Power consumption status
* Sensor health
* System event logs (SEL)

📌 **Tip:**
If multiple nodes in the same rack show failures, immediately suspect:

* Power issues
* Cooling issues

---

## 7. Power Drain (Cold Power Reset) Procedure

A power drain is recommended after identifying hardware issues.

### Best Practice

Always **collect logs first** before performing a power drain.

---

## 8. Log Collection via iDRAC

### Steps

1. Login to iDRAC console
2. Navigate to **Maintenance → SupportAssist**
3. Click **Start Log Collection**
4. Save logs for analysis or Dell support

---

## 9. Power Drain Procedure (System Mute / OFF-RPM)

### Steps

1. Remove **network cables**

   * PowerSync cable
   * InfiniBand (IB) cable
   * iDRAC cable
2. Remove **power cables from both PSUs**
3. Remove any additional power cables (if connected)
4. Press and hold the **power button for 2–5 minutes**

   * This drains residual power completely

---

## 10. Power-On and Recheck

1. Reconnect **all cables**
2. Power ON the server
3. Re-collect logs
4. Observe iDRAC alerts
5. If issue persists, **raise a Dell support case** with logs

---

## 11. Common Hardware Troubleshooting Scenarios

### RAM Issue

* Pop-up error in iDRAC
* Scenario 1: Remove faulty DIMM
* Scenario 2: Swap DIMM position
* Replace DIMM if issue persists

---

### InfiniBand (IB) Cable Issue

* Try a different cable
* Perform power drain
* If issue persists, replace IB cable

---

### PSU Issue

* Swap PSU positions
* Check redundancy status
* Replace PSU if issue persists

---

### iDRAC Access Issue

* Reset iDRAC
* Check BIOS/iDRAC settings
* Replace iDRAC card if required

---

### CPU Issue

* Boot server with **1 CPU / 1 RAM**
* Identify faulty CPU
* Proceed with replacement

---

### Fan Issue

* Swap fan positions
* If issue follows the fan → replace fan

---

## 12. Final Step – Node Recovery

Once the issue is resolved:

```bash
pbsnodes -r <node-hostname>
```

✔ Node is now available for scheduling jobs.

---

## 13. Best Practices

* Always check **scheduler status first**
* Never disturb nodes with running jobs
* Use **OME as the authoritative source**
* Collect logs **before** power drain
* Follow systematic isolation (swap & test)
* Escalate to Dell support with logs if required

---
