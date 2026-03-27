# Download Ubuntu
https://ubuntu.com/download/desktop
Select Ubuntu 24.04.4 LTS
Setup:
![[Pasted image 20260321185108.png]]
Storage: 20gb
Memory: 4gb
Processors: 2
Replicate physical network connection state ON
# Windows Setup
![[Pasted image 20260323135421.png]]
Storage: 40gb
Memory: 2gb
Processors: 2
Replicate physical network connection state ON


# Setting Up Network
## Checking machines
Ubuntu
![[Pasted image 20260323140655.png]]
Subnet: `255.255.255.0`
IP: `192.168.1.107`

Windows
![[Pasted image 20260323140824.png]]
Subnet: `255.255.255.0`
IP: `192.168.1.113`

This tells use they are some same network and should be able to communicate with each other

## Ping Test
![[Pasted image 20260323140950.png]]

*If Ubuntu doesn't work try disabling firewall and setting network to private as firewall may block Traffic*
```cmd
netsh advfirewall set allprofiles state off
```

![[Pasted image 20260323141135.png]]

Windows and Ubuntu VM are able to communicate with each other.

# Installing Manager on Ubuntu
Install gnupg
```bash
sudo apt update
sudo apt install -y gnupg
```

Install GPG Key command
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg
```

Install Wazuh
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i
```
change `4.14` to latest version.
## Resizing VM
While installing I did run into issue where ubuntu vm didn't have enough space. To fix this:
1. Shut down Ubuntu 
2. Go to VM on vmware
3. Go to Hard disk 
4. Expand disk capacity on the right 
5. change to 60 gb
![[Pasted image 20260327131810.png]]

OS is only using original partition size so I will have to fix it in ubuntu.
Checking resize using `lsblk`
![[Pasted image 20260327132558.png]]
`sda` is 60GB so the resize did happen

Resize `sda2`:
```
sudo growpart /dev/sda 2
sudo resize2fs /dev/sda2
```

Check again
![[Pasted image 20260327132942.png]]

Notes:
`sda` represents whole disk
`sda1` and `sda2` are partitions
`lsblk`: list all block devices

Finished installation image:
![[Pasted image 20260327145900.png]]
## Accessing Wazuh Manager GUI
1. Use `ifconfig` or any other network command to find your device IP address 
2. Enter `https://<ipaddress>:443` 
3. Login with provide credentials
![[Pasted image 20260327150306.png]]

# Install and Onboard Windows Agent on Windows
1. Download windows installer from [here](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)
2. Open installer accept agreement then finish
3. Open Wazuh agent can be done via search bar
![[Pasted image 20260327173022.png]]
The ip entered should be the ubuntu machines IP address. For auth key we will need to open ubuntu machine.

## Onboarding Agent
1. Open Terminal on Ubuntu
2. Run the following command to add agents
```bash
sudo /var/ossec/bin/manage_agents
```
6. On menu enter `A` to choose adding agent
![[Pasted image 20260327173653.png]]
7. Enter any name for agent name and the windows machines IP address
![[Pasted image 20260327173738.png]]
We have successfully created a new agent
8. To extract a key enter `E` on menu
9. Enter agent ID number
![[Pasted image 20260327174050.png]]
10. Go back to Windows Machine to paste auth key
![[Pasted image 20260327175102.png]]
11. Save and restart agent by clicking on manage > restart
## Outcome
Going back to wazuh platform on Ubuntu we can see that one agent is active and it is the windows agent.
![[Pasted image 20260327175434.png]]
Clicking on `WINDOWS-AGENT` shows us all the logs from windows machine
![[Pasted image 20260327175631.png]]
