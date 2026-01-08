

 ** How to Configure Network and Proxy Settings on RHEL**

---

 **Step 1: Confirm Network Connectivity**

Before configuring network settings, ensure that the network is reachable.

1. **Check Network Connectivity:**
   Open a terminal and use the `ping` command to verify that the network or a remote server is reachable.

   ping <destination-ip>   # Replace with a reachable IP or domain, e.g., 8.8.8.8 or google.com
   

   If the ping is successful, the network is reachable. If not, troubleshoot the network connection.

**Step 2: Assign IP Address, Subnet Mask, Gateway, and DNS**

To assign a static IP on RHEL, you will need to modify the network configuration files or use `nmcli` (NetworkManager's command-line tool).

 **Option 1: Using `nmcli` Command**

1. **Identify the Network Interface:**
   List all the network interfaces to identify the one you want to configure (e.g., `eth0`, `enp0s3`).
  
   nmcli device

   Find the interface you want to configure (e.g., `eth0` or `enp0s3`).

2. **Configure the IP Address:**
   Run the following command to assign a static IP address, subnet mask, gateway, and DNS server.

   nmcli con mod "System eth0" ipv4.addresses 192.168.1.100/24   # Replace with your desired IP
   nmcli con mod "System eth0" ipv4.gateway 192.168.1.1          # Replace with your gateway
   nmcli con mod "System eth0" ipv4.dns "8.8.8.8 8.8.4.4"        # Replace with your DNS
   nmcli con mod "System eth0" ipv4.method manual

3. **Restart the Network Connection:**

   After configuring the settings, restart the network connection for the changes to take effect.

   nmcli con up "System eth0"

 **Option 2: Manually Editing the Network Script (for older RHEL versions)**

1. **Navigate to Network Scripts Directory:**
   Network configuration files are located in `/etc/sysconfig/network-scripts/`.
                                                                                                       
   cd /etc/sysconfig/network-scripts/

2. **Edit the Configuration File:**
   Find the configuration file for your network interface (e.g., `ifcfg-eth0` or `ifcfg-enp0s3`), and open it in a text editor.

   sudo nano ifcfg-eth0  # Replace eth0 with your interface name

3. **Modify the File:**
   Change or add the following lines to assign a static IP, subnet mask, gateway, and DNS server.

   Example of a static IP configuration:

   DEVICE=eth0
   BOOTPROTO=static
   IPADDR=192.168.1.100   # Replace with your desired IP address
   NETMASK=255.255.255.0  # Replace with your subnet mask
   GATEWAY=192.168.1.1    # Replace with your gateway
   DNS1=8.8.8.8          # Replace with your preferred DNS server
   DNS2=8.8.4.4          # Optional: additional DNS server
   ONBOOT=yes

4. **Restart the Network Service:**

   After editing the configuration file, restart the network service to apply the changes:

   sudo systemctl restart network
  
 **Step 3: Configure Proxy Settings**

 **Option 1: Using `NetworkManager` GUI (if available)**

1. **Open Network Settings:**

   * On RHEL, open the **Settings** window (you can search for it in the Activities or Applications menu).
   * Go to **Network**.

2. **Select the Active Network Connection:**

   * Choose either your **Wired** connection from the list.
   * Click the **Settings** gear icon next to the active connection.

3. **Configure Proxy Settings:**

   * In the network settings window, navigate to the **Proxy** tab.
   * Select **Manual** for setting the proxy.
   * Enter the proxy information:

     * **HTTP Proxy**: `http://proxy.example.com:8080`
     * **HTTPS Proxy**: `http://proxy.example.com:8080`
     * **FTP Proxy** (optional): `http://proxy.example.com:8080`
   * You can also set **No Proxy** for specific domains like `localhost, 127.0.0.1, .example.com`.

4. **Apply the Changes:**
   Click **Apply** to save and apply the proxy settings.

 **Option 2: Using Environment Variables (CLI)**

1. **Set Proxy Environment Variables Globally:**
   Open the terminal and modify the `~/.bashrc` (or `~/.bash_profile`) file to set environment variables for the proxy.

   nano ~/.bashrc
  

2. **Add the Following Lines to Set Proxy:**

   Add the following lines to define the HTTP and HTTPS proxy:

   export http_proxy="http://proxy.example.com:8080"
   export https_proxy="http://proxy.example.com:8080"
   export ftp_proxy="http://proxy.example.com:8080"   # Optional, for FTP
   export no_proxy="localhost,127.0.0.1,.example.com"  # Optional, to exclude certain domains from proxy
   Temparary in terminal
   export http_proxy="http://username:Password%60123456@1.1.1.1:0000/"
   export https_proxy="https://username:Password%60123456@1.1.1.1:0000/" replay the your username and password (here password =Password@123456)
3. **Reload the Shell Configuration:**

   After modifying the file, reload the shell configuration to apply the changes:

   source ~/.bashrc   # For bash
   

**Option 3: Using `dnf` for Proxy Settings**

If you want to set the proxy for `dnf` (package manager), you can create or modify the `dnf.conf` file:

1. **Edit the `dnf.conf` File:**

   sudo nano /etc/dnf/dnf.conf
   

2. **Add the Proxy Settings:**

   Add the following lines to configure the proxy for `dnf`:

    proxy=http://1.1.1.1:0000
    proxy_username=username
    proxy_password=Password%60123456


---
4. Firefox Proxy Configuration

Open Firefox:

Launch Firefox from the Applications menu or by typing firefox in the terminal.

Access the Proxy Settings:

Click the hamburger menu (three horizontal lines) in the top-right corner of the browser.

Select Preferences.

Go to Network Settings:

Scroll down and click on Settings under the Network Settings section.

Configure Proxy Settings:

In the Connection Settings window, select Manual proxy configuration.

Enter your proxy server's HTTP Proxy and Port (e.g., http://proxy.example.com:8080).

Check the box Use this proxy server for all protocols if the same proxy is used for all types of traffic.

Apply and Save:

Click OK to apply the settings.

5. Google Chrome / Chromium Proxy Configuration
Google Chrome

Open Google Chrome:

Launch Google Chrome from the Applications menu or by typing google-chrome in the terminal.

Access the Proxy Settings:

In the address bar, type chrome://settings and press Enter.

Scroll to the Bottom:

Scroll to the bottom of the settings page and click on Advanced.

Open System Proxy Settings:

Under the System section, click on Open your computer’s proxy settings. This will open the system’s default network settings for proxies.

