# Power Supply Unit (PSU) Issue 
---

## What is a PSU?

A **Power Supply Unit (PSU)** gives power to the server.
If the PSU fails, the server may not start or may shut down.

PSU error messages usually look like:

* PSU0001
* PSU0002
* PSU0003

---

## Common PSU Problems

You may see:

* Server not turning ON
* Amber (orange) light on PSU
* Power redundancy warning
* No display (No POST)

---

## First checks

1. Make sure the **power cable is connected tightly**
2. Check the **power socket or PDU**
3. Press the **server power button**
4. Check if any **PSU warning** is shown in iDRAC

---

## Troubleshooting Steps

### Step 1: Check Cables

* Remove and reconnect the power cable
* Make sure PSU is fully inserted into the server

---

### Step 2: Swap the PSU

If the server has **two PSUs**:

* Swap the PSU with the other slot
* If the problem moves → PSU is bad
* If the problem stays → board or motherboard issue

> ⚠️ Remove **only one PSU at a time**

---

### Step 3: Check PSU Type

* Both PSUs must be **same model**
* Wattage must be enough for server hardware
* After CPU/GPU upgrade, old PSU may not be enough

---

### Step 4: Check PSU LED Light

| LED Color      | Meaning             |
| ---------------| ------------------- |
| Green          | PSU is working      |
| Blinking Green | PSU firmware update |
| Blinking Amber | PSU problem         |
| No Light       | No power connected  |

---

## Replacing a PSU 

If the server has **two PSUs (redundant)**, you do NOT need to shut down.

### Remove PSU

1. Unplug power cable
2. Press release latch
3. Pull PSU out slowly

### Insert New PSU

1. Slide new PSU into slot
2. Connect power cable
3. Wait a few seconds
4. Check LED → should be **green**

---

## Important Safety Notes

* Never remove **both PSUs at the same time**
* Use **same type PSU** only
* PSU firmware update causes downtime

---

## If Problem Still Exists

* PSU may be failed
* Or motherboard / power board issue
* Contact **technical support**

---

## Summary

✔ Check cables

✔ Check power source

✔ Swap PSU

✔ Check LED light

✔ Replace PSU if needed

---

