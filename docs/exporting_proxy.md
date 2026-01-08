# How to Configure Network and Proxy Settings on RHEL

---

## Step 1: Confirm Network Connectivity

Before configuring network settings, ensure that the network is reachable.

### Check Network Connectivity

Open a terminal and use the `ping` command to verify that the network or a remote server is reachable.

```bash
ping <destination-ip>   # Replace with a reachable IP or domain, e.g., 8.8.8.8 or google.com
````

If the ping is successful, the network is reachable. If not, troubleshoot the network connection.

---

## Step 2: Assign IP Address, Subnet Mask, Gateway, and DNS

To assign a static IP on RHEL, you can modify the network configuration files or use `nmcli` (NetworkManager's command-line tool).

### Option 1: Using `nmcli` Command

1. **Identify the Network Interface**

List all network interfaces to identify the one you want to configure (e.g., `eth0`, `enp0s3`):

```bash
nmcli device
```

2. **Configure the IP Address**

Assign a static IP address, subnet mask, gateway, and DNS server:

```bash
nmcli con mod "System eth0" ipv4.addresses 192.168.1.100/24   # Replace with your desired IP
nmcli con mod "System eth0" ipv4.gateway 192.168.1.1          # Replace with your gateway
nmcli con mod "System eth0" ipv4.dns "8.8.8.8 8.8.4.4"        # Replace with your DNS
nmcli con mod "System eth0" ipv4.method manual
```

3. **Restart the Network Connection**

```bash
nmcli con up "System eth0"
```

---

### Option 2: Manually Editing the Network Script (for older RHEL versions)

1. **Navigate to Network Scripts Directory**

```bash
cd /etc/sysconfig/network-scripts/
```

2. **Edit the Configuration File**

Open the configuration file for your network interface (e.g., `ifcfg-eth0`) in a text editor:

```bash
sudo nano ifcfg-eth0  # Replace eth0 with your interface name
```

3. **Modify the File**

Example of a static IP configuration:

```ini
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.100   # Replace with your desired IP address
NETMASK=255.255.255.0  # Replace with your subnet mask
GATEWAY=192.168.1.1    # Replace with your gateway
DNS1=8.8.8.8           # Replace with your preferred DNS server
DNS2=8.8.4.4           # Optional: additional DNS server
ONBOOT=yes
```

4. **Restart the Network Service**

```bash
sudo systemctl restart network
```

---

## Step 3: Configure Proxy Settings

### Option 1: Using `NetworkManager` GUI (if available)

1. **Open Network Settings**

* On RHEL, open the **Settings** window (search in Activities or Applications menu).
* Go to **Network**.

2. **Select the Active Network Connection**

* Choose your **Wired** connection.
* Click the **Settings** gear icon next to the active connection.

3. **Configure Proxy Settings**

* Navigate to the **Proxy** tab.
* Select **Manual**.
* Enter proxy information:

```
HTTP Proxy: http://proxy.example.com:8080
HTTPS Proxy: http://proxy.example.com:8080
FTP Proxy (optional): http://proxy.example.com:8080
No Proxy (optional): localhost, 127.0.0.1, .example.com
```

4. **Apply Changes**

Click **Apply** to save and apply the proxy settings.

---

### Option 2: Using Environment Variables (CLI)

1. **Set Proxy Environment Variables Globally**

Edit `~/.bashrc` (or `~/.bash_profile`):

```bash
nano ~/.bashrc
```

2. **Add the Following Lines**

```bash
export http_proxy="http://proxy.example.com:8080"
export https_proxy="http://proxy.example.com:8080"
export ftp_proxy="http://proxy.example.com:8080"   # Optional, for FTP
export no_proxy="localhost,127.0.0.1,.example.com"  # Optional, exclude domains

# Temporary proxy in terminal
export http_proxy="http://username:Password%60123456@1.1.1.1:0000/"
export https_proxy="https://username:Password%60123456@1.1.1.1:0000/"
```

> Replace `username` and `Password@123456` with your actual credentials.

3. **Reload the Shell Configuration**

```bash
source ~/.bashrc   # For bash
```

---

### Option 3: Using `dnf` for Proxy Settings

1. **Edit the `dnf.conf` File**

```bash
sudo nano /etc/dnf/dnf.conf
```

2. **Add the Proxy Settings**

```ini
proxy=http://1.1.1.1:0000
proxy_username=username
proxy_password=Password%60123456
```

---

## Step 4: Firefox Proxy Configuration

1. **Open Firefox**

Launch Firefox from Applications or terminal:

```bash
firefox
```

2. **Access Proxy Settings**

* Click the hamburger menu (three horizontal lines) → Preferences.
* Scroll down → **Network Settings** → Click **Settings**.

3. **Configure Proxy**

* Select **Manual proxy configuration**.
* Enter HTTP Proxy and Port (e.g., `http://proxy.example.com:8080`).
* Check **Use this proxy server for all protocols** if applicable.

4. **Apply and Save**

Click **OK**.

---

## Step 5: Google Chrome / Chromium Proxy Configuration

1. **Open Google Chrome**

```bash
google-chrome
```

2. **Access Proxy Settings**

* Type `chrome://settings` in the address bar → Press Enter.
* Scroll to bottom → Click **Advanced** → Under **System**, click **Open your computer’s proxy settings**.

This opens the system’s default proxy configuration where you can apply your proxy settings.

```

---

If you want, I can **also make it look visually like a proper HPC admin documentation page** with **green-colored headings, code blocks styled, and rounded navigation buttons** for your GitHub Pages.  

Do you want me to do that next?
```
