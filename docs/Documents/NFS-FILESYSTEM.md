**NFS-FILESYSTEM**

**NFS-SERVER**

Step :1

Install required nfs and rpcbind pacakages in server node

Check the status\
systemctl status nfs-server

Systemctl restart nfs-server\
\
Step :2

Check the firewall service

Systemctl status firewalled.service

Systemctl stop firewalled service

OR\
\
if firewalled service must enabled add the following services and
enabled port number\
\
![](media/image1.png){width="6.268055555555556in"
height="3.230200131233596in"}\
step :3

Create directory which is want share across network\
change owner ship for the directory\
if require change the directory permissions also

![](media/image2.png){width="6.268055555555556in"
height="0.5905325896762904in"}

Step 4:

Add the nfs shared values in /etc/exports\
\
/directory_which_is_want_share ip_address_of_network(permissions for
shared point)\
\
/directory_which_is_want_share ip_address_of_clint_machine(permissions
for shared point)\
\
![](media/image3.png){width="6.268055555555556in"
height="0.28366360454943135in"}

![](media/image4.png){width="6.268055555555556in"
height="1.0797769028871391in"}\
step 5:

Check the expoert values with\
\
exportfs --a

exportfs --r

exportfs --v

![](media/image5.png){width="6.268055555555556in"
height="1.4083158355205598in"}

![](media/image6.png){width="6.268055555555556in"
height="0.7236395450568679in"}

Everything is good we need to configure the NFS-CLINT server

NFS-CLINT

Step 6:

Check the network of the nfs-server machine

If network is reachable we can go farther configuration\
\
check the firewall rules and port enable\
\
step 7:

Create a mounting directory\
![](media/image7.png){width="6.268055555555556in"
height="0.33969706911636044in"}

Step 8:

Mount the nfs_mount path\
![](media/image8.png){width="6.268055555555556in"
height="0.29483923884514435in"}\
or\
\
![](media/image9.png){width="6.268055555555556in"
height="1.7061439195100612in"}\
step 9:

For perment mount of after reboot add the nfs server valves in
/etc/fstab conf file\
\
the values of nfs system, ip addres of server,mount path of server,mount
of the clint and defaults values\
![](media/image10.png){width="6.395166229221347in" height="1.64in"}\
step 10 :

Check and verify mount point mounted or not\
\
![](media/image11.png){width="6.268055555555556in"
height="1.6512248468941382in"}
