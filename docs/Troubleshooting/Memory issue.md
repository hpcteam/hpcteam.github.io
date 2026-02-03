# DIMM (Memory) Error Troubleshooting – Detailed SOP Using iDRAC
---

## Purpose

This document explains how to **identify, confirm, and resolve DIMM (memory) errors** using the **iDRAC console**, system logs, POST messages, and physical troubleshooting.

---

## Prerequisites

* iDRAC access
* Console access (Virtual Console or Physical KVM)
* Proper ESD precautions
* Server powered safely

---

## Step 1: Identify the Issue in iDRAC Console

1. Log in to the **iDRAC console**.
2. Navigate to:

   * **System Overview**
   * **Hardware / Memory Section**
3. Check system health status:

   * Look for **Memory / DIMM warnings or critical alerts**.
4. Note:

   * DIMM slot number (example: A1, B2, C1)
   * Error description
   * Timestamp of the error

**Purpose:**
To confirm whether the issue is related to memory and identify the affected DIMM slot.

---

## Step 2: Collect Logs from iDRAC

1. From iDRAC, collect the following logs:

   * Lifecycle Logs
   * System Event Logs (SEL)
2. Save the logs for reference.
3. Verify:

   * Any DIMM-related error entries
   * Repeated or persistent memory errors

 **Purpose:**
Logs help confirm whether the error is **intermittent or persistent** and are required for escalation.

---

## Step 3: Perform Power Drain

1. Gracefully power off the server.
2. Remove **all power cables** from the server.
3. Presee the power on button Wait for **10 to 30 seconds** (minimum).
4. Reconnect the power cables.
5. Power on the server.

 **Purpose:**
Power drain clears residual power and temporary hardware faults.

---

## Step 4: Observe POST Logs During Boot

1. Open the **Virtual Console** or connect to physical console.
2. Power on the server.
3. Carefully observe the **POST screen**:

   * Memory initialization messages
   * Any DIMM or memory-related errors
4. If:

   * Server fails to boot OR
   * Any error appears during POST
     Then:
   * Allow the server to complete power-up
   * Wait **2 additional minutes**
   * Recheck logs in iDRAC

 **Purpose:**
POST messages give **real-time confirmation** of hardware issues.

---

## Step 5: Confirm DIMM Error

1. Compare:

   * POST error messages
   * iDRAC alerts
   * System logs
2. Confirm that the issue is specifically a **DIMM error**.

**Purpose:**
Avoid unnecessary hardware replacement by confirming the root cause.

---

## Step 6: Power Off and Open the Server

1. Power off the server.
2. Remove **all power cables**.
3. Follow ESD safety guidelines.
4. Open the system cover.

 **Purpose:**
Prepare for physical inspection of memory modules.

---

## Step 7: Verify and Reseat DIMMs

1. Locate the DIMM slot showing the error (example: A1).
2. Check:

   * DIMM is fully inserted
   * Locking clips are properly engaged
3. Reseat the DIMM:

   * Remove the DIMM
   * Reinsert it firmly into the same slot
4. Verify **all DIMMs**:

   * Properly seated
   * Correct population order as per system guidelines

 **Purpose:**
Loose or improperly seated DIMMs are the most common cause of memory errors.

---

## Step 8: Power On and Observe Again

1. Close the system cover.
2. Reconnect power cables.
3. Power on the server.
4. Observe:

   * POST screen
   * Any memory error messages
5. Allow the server to boot fully.
6. Check iDRAC logs and console.

**Purpose:**
Confirm whether reseating resolved the issue.

---

## Step 9: Swap DIMM Positions (Isolation Test)

If the DIMM error still appears:

1. Power off the server.
2. Remove all power cables.
3. Open the system.
4. Move the DIMM to another slot:

   * Example:

     * Original error: **A1**
     * Move DIMM from **A1 → B1**
5. Close the system cover.
6. Power on the server.
7. Observe:

   * POST screen
   * Console messages
8. Wait until the server fully powers up.
9. Check iDRAC logs again.

 **Purpose:**
To determine whether the issue is with the  **DIMM slot / motherboard**.

---

## Step 10: Analyze the Result

### Case 1: Same DIMM Slot Shows Error Again

* Error remains on the **same slot** (example: A1)
* DIMM moved, but slot still reports error

✅ **Conclusion:**

* Slot or motherboard issue
  ➡️ Escalate to **support / vendor team**

---

### Case 2: Error Moves with the DIMM

* Error now appears in **new slot** (example: B1)
* Error follows the DIMM

✅ **Conclusion:**

* DIMM is faulty
  ➡️ Proceed with **DIMM replacement**

---

## Final Summary (Quick View)

* Identify error in **iDRAC**
* Collect logs
* Power drain (2 minutes)
* Observe POST logs
* Reseat DIMM
* Swap DIMM positions
* Decide:

  * Slot issue → Support team
  * DIMM issue → Replace DIMM

---
