# Active Directory | Threat Detection | Brute Force and Malware Injection on Home Lab with Splunk

This home lab is dedicated to simulating a real cyber attack and endpoint detection and response. I implemented 4 virtual machines to simulate the "office network" & victim machines. The attack machine will utilize "Crowbar" as its initial bruteforce attack, tacking advantage of the insecured Remote Desktop configuration on vimtim machines as well as 'Sliver' as a C&C framework to attack a Windows endpoint machine and attempt to move laterally across the network or obtain a reverse shell.

---

## Lab Diagram

![image](https://github.com/user-attachments/assets/67875cb6-c33b-4394-9cad-2edf650f1cf8)

The lab consists of:
- **Windows 11**: Target machine
- **Windows Server 2022**: Domain controller and Active Directory
- **Ubuntu Server 24.04.1**: Splunk server
- **Kali Linux 2024.3**: Attacker machine

The VMs are connected via a NAT Network with a common IPv4 prefix to ensure seamless communication.

---

## Setting Up the Virtual Machines

### Tools and ISOs
- VirtualBox was used to create and manage the VMs.
- Required ISOs:
  - **Windows 11**
  - **Windows Server 2022**
  - **Ubuntu Server 24.04.1**
  - **Kali Linux 2024.3**
![image](https://github.com/user-attachments/assets/15743937-fb2f-4951-83af-4f1c6858ce74)


### Network Configuration
- A NAT Network was configured for all VMs.
- The IPv4 prefix was set to match the lab design.
- Static IP addresses were assigned for specific roles:
  - **Splunk Server**: `192.168.10.10`
  - **Other devices**: Dynamically assigned as required

---

## Configuring Splunk

### Ubuntu Server Setup
1. **Set Static IP**:
   - Verified the current IP with `ip a`.
   - Edited `/etc/netplan/50-cloud-init.yaml` to:
     ```yaml
     network:
       version: 2
       ethernets:
         ens33:
           addresses: [192.168.10.10/24]
           gateway4: 192.168.10.1
           nameservers:
             addresses: [8.8.8.8]
           dhcp4: no
     ```
   - Applied the changes using `sudo netplan apply`.

2. **Install Splunk**:
   - Downloaded and installed Splunk Enterprise.
   - Configured the Splunk web interface for initial use.
![image](https://github.com/user-attachments/assets/bb4041a1-a2b5-4db5-9c4f-685e23d56af9)


### Splunk and Sysmon on Windows Machines
1. **Install Splunk Universal Forwarder**:
   - Downloaded Splunk Universal Forwarder.
   - Configured the `input.conf` file for log forwarding.

2. **Install Sysmon**:
   - Used [Olaf Hartong’s Sysmon Modular Config](https://github.com/olafhartong/sysmon-modular) to enhance visibility.

3. **Log Events**:
   - Set up an endpoint index in Splunk.
   - Configured forwarding and receiving to enable event population in Splunk.
![image](https://github.com/user-attachments/assets/16ed8498-18c9-4d3e-b22c-15c93e96a95f)

---

## Setting Up Active Directory

### Domain Controller
1. Installed the Active Directory Domain Services role via the **Server Manager**.
2. Promoted the Windows Server to a domain controller.
![image](https://github.com/user-attachments/assets/e30ff944-7041-4db7-8165-386f89d67093)


### Organizational Units and Users
- Created two OUs:
  - **SOC**
  - **HR**
- Added two users:
  - **Jessica Rabbit** (HR department)
  - **Will Matthews** (SOC department)
![image](https://github.com/user-attachments/assets/dcc84bc0-f863-4ade-b048-1735eed57533)


### Connecting the Target Machine
1. Set the DNS of the Windows 11 target machine to the server's IP.
2. Restarted and verified login with newly created domain users.

---

## Attacking the Home Lab

### Tools
- **Kali Linux 2024.3**: Used for penetration testing and attack simulation.
- **Crowbar**: Used for bruteforcing Will Matthews user password through RDP and using rockyou.txt password list
- **Sliver**: Used for generating and delivering payloads to vulnerable computer

### Attack Techniques
- Simulated brute-force attacks.<br>
  ![image](https://github.com/user-attachments/assets/585d5e5e-3062-44ef-a0d6-b59533209ebb)
- Generated and delivered payload using Sliver
![image](https://github.com/user-attachments/assets/af8ff633-4346-4265-b573-f9c7583e28d1)
![image](https://github.com/user-attachments/assets/ddcc756a-d9e5-4892-8fe1-3087c9f7a3c2)
![image](https://github.com/user-attachments/assets/51a6908e-a1b7-4f0c-8964-ed3a3790cd2c)
- Tested lateral movement within the network.<br>
![image](https://github.com/user-attachments/assets/76132c44-b3c6-48d1-a7a5-25a08d5dafa3)
![image](https://github.com/user-attachments/assets/730f1bfd-cf1e-4a86-8df6-c9f52405e217)
- After some C&C was able to obtain reverse shell
![image](https://github.com/user-attachments/assets/b31d225f-1898-44cd-97a1-1065754605b4)


---

## Defense and Threat Detection

- **Splunk Monitoring**:
  - Detected suspicious activities logged by Sysmon on both Windows devices
![image](https://github.com/user-attachments/assets/100a4d5c-3ac1-4a44-815f-6e65da974664)
![image](https://github.com/user-attachments/assets/1450940e-3681-45f3-9571-bfeb495a2f75)
  - Investigated indexed events for abnormal patterns and was able to identify source of attack through closer inspection of Event ID and Log Details.<br>
![image](https://github.com/user-attachments/assets/08c6f6f2-6958-4ace-9987-a0bb10f6a4cc)
![image](https://github.com/user-attachments/assets/f6240450-83c1-4023-be0b-4525c08d55f7)
![image](https://github.com/user-attachments/assets/daac9698-07cb-413a-aff0-a9e227c117a2)
---

