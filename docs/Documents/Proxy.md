
---

 Proxy Network – Basics Guide

This guide explains **what a proxy network is**, **why it is used**, and **how to use it**, starting from the very basics.  
It is intended for students, developers, and anyone working in institutional or server environments.

---

## 1. What Is a Proxy?

A **proxy** is an intermediate server between your computer and the internet.

### Normal Internet Connection
```

Your Computer → Website

```

### Internet Connection Using a Proxy


Your Computer → Proxy Server → Website


Instead of connecting directly to a website, your request is first sent to the **proxy server**, which then communicates with the website on your behalf.

---

## 2. What Is a Proxy Network?

A **proxy network** is a setup where:
- Multiple users access the internet
- Through one or more **proxy servers**

The proxy server:
- Receives requests from users  
- Forwards them to the internet  
- Receives responses  
- Sends the responses back to users  

Proxy networks are commonly used in **companies, universities, data centers, and HPC clusters**.

---

## 3. Why Use a Proxy?

Common reasons include:

### 1. Internet Access Control
- Restrict or allow access to specific websites
- Widely used in offices, colleges, and research labs

### 2. Security
- Hides the user’s real IP address
- Protects internal networks from direct exposure

### 3. Monitoring
- Allows administrators to monitor internet usage

### 4. Caching
- Frequently accessed content loads faster
- Reduces bandwidth usage

---

## 4. Types of Proxies (Basic)

### HTTP Proxy
- Used for standard web traffic
- Example: browsing normal websites

### HTTPS Proxy
- Used for encrypted web traffic
- Required for secure websites

---

## 5. How a Proxy Works (Step by Step)

1. You request a website  
2. The request goes to the proxy server  
3. The proxy checks access permissions  
4. The proxy forwards the request to the website  
5. The website responds to the proxy  
6. The proxy sends the response back to you  

---

## 6. Using a Proxy on Linux / macOS

Most command-line tools read proxy settings from **environment variables**.

Common variables:
- `http_proxy`
- `https_proxy`

---

## 7. What Does `export` Mean?

In Linux/macOS:
- `export` sets an **environment variable**
- Makes it available to all programs started from that terminal session

---

## 8. Setting Proxy Using `export`

### Example Commands

```bash
export http_proxy="http://username:password@proxy_ip:port/"
export https_proxy="https://username:password@proxy_ip:port/"
```
### Real Example

```bash
export http_proxy="http://application_user:Abc%60123456@100.100.100.100:100/"
export https_proxy="https://application_user:Abc%60123456@100.100.100.100:100/"
```

---

## 9. Breaking Down the Proxy URL

```
http://application_user:Abc%60123456@100.100.100.100:100/
```

| Part               | Meaning                      |
| ------------------ | ---------------------------- |
| `http://`          | Proxy protocol               |
| `application_user` | Proxy username               |
| `Abc%60123456`     | Proxy password (URL-encoded) |
| `100.100.100.100`  | Proxy server IP              |
| `100`              | Proxy port                   |

### Note on URL Encoding

* `%40` is URL encoding for `@`
* Password `Abc@123456` becomes `Abc%60123456`

---

## 10. What Happens After Exporting the Proxy?

Once exported:

* Tools like `curl`, `wget`, `pip`, `apt`, and `git`
* Automatically route traffic through the proxy

### Example

```bash
curl google.com
pip install numpy
```

---

## 11. Temporary vs Permanent Proxy Settings

### Temporary

* Works only for the current terminal session
* Lost after closing the terminal

### Permanent

Add the export commands to:

* `~/.bashrc`
* `~/.profile`

---

## 12. Security Warning ⚠️

Proxy credentials are stored in **plain text**.

Recommendations:

* Avoid sharing terminal screenshots
* Clear shell history if required
* Use secure credential management when possible

---

## 13. Installing Python Libraries Using `pip` Through a Proxy

### Method 1: Proxy in the `pip install` Command (One-Time)

```bash
pip install pandas --proxy http://application_user:Abc%60123456@100.100.100.100:100
```

✅ Uses the proxy only for this command.

---

### Method 2: Export Proxy Once (Recommended)

#### Step 1: Export Proxy

```bash
export http_proxy="http://application_user:Abc%60123456@100.100.100.100:100/"
export https_proxy="https://application_user:Abc%60123456@100.100.100.100:100/"
```

#### Step 2: Install Package

```bash
pip install pandas
```

✅ `pip` automatically uses the proxy.

---

### Method 3: Permanent Proxy for `pip` (Best for Servers / HPC)

#### Step 1: Create Config Directory

```bash
mkdir -p ~/.pip
```

#### Step 2: Create Config File

```bash
nano ~/.pip/pip.conf
```

#### Step 3: Add Configuration

```ini
[global]
proxy = http://application_user:Abc%60123456@100.100.100.100:100
```

#### Step 4: Install Normally

```bash
pip install pandas
```

---

## 14. Common Mistakes ❌

### Wrong Package Name

```bash
pip install pands   # ❌
pip install pandas  # ✅
```

### Missing URL Encoding

* Use `Abc%60123456`
* Not `Abc@123456`

---

## 15. How to Verify Proxy Is Working

```bash
pip install numpy -v
```

If you see:

```
Connecting to 100.100.100.100:100
```

➡️ The proxy is being used.

---

## 16. SSL / Certificate Errors

Try:

```bash
pip install pandas \
  --proxy http://application_user:Abc%60123456@100.100.100.100:100 \
  --trusted-host pypi.org \
  --trusted-host files.pythonhosted.org
```

---

## 17. Quick Summary 🧠

| Method     | When to Use            |
| ---------- | ---------------------- |
| `--proxy`  | One-time install       |
| `export`   | Session-wide usage     |
| `pip.conf` | Permanent server setup |

---

**In short:**
A proxy is a middleman between you and the internet.
Proxy networks are widely used in institutions for security, control, and performance.

```

---

