# WazuhSIEMIntro
Following John Hammonds Wazuh SIEM intro Lab.
## Goals
## Steps

## Step 1: Create VM for Wazuh Server
- Use Virtual Machine Manager (virt-manager/QEMU/KVM) to create an Ubuntu 22.04 VM on local linux machine. Used system requirements as per Wazuh docucmentation.
<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/WazuhVMReqs.png" alt="VM reqs" />
</p>

## Step 2: Install Wazuh Manager/Server onto Ubuntu Server VM
- Install Wazuh Server onto VM Server with following bash script from Wazuh documentation:
```
curl -sO https://packages.wazuh.com/4.11/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
- During installation Wazuh admin login details and password are printed to shell. These were saved to a text file for reference.
- Use linux command to retrieve the ip address of the VM Wazuh server :
```
ip a
 ```

## Step 3: Create VM for Wazuh Agent
-  Clone previous Ubuntu VM in virtmanager for use as Agent VM

<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/ClonedVMAgent.png"/>
</p>

-  Use Wazuh guide to add new agent details
<p align="left">
    <img src=""/>
</p>
-   to Install Wazuh Agent onto Agent VM
-  

## Step 4: Enable vulnerability detection
## Step 5: Install Invoke Atomics
## Step 6: Create security events with Invoke Atomics.

