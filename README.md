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
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/AddAgentDialogue.png"/>
</p>

-  This created the following bash install script which is copied and run in the terminal

```
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.11.2-1_amd64.deb && sudo WAZUH_MANAGER='192.168.122.228' WAZUH_AGENT_NAME='AgentPlusOne' dpkg -i ./wazuh-agent_4.11.2-1_amd64.deb
```
-  The following bash commands are run to add the wazuh agent service to the linux service manager

```
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
-  This step can be repeated for multiple agents.
-  Reload Wazuh dashboard. Navigate to Agents Management summary page to confirm addition of new agent.

<p align="centre">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/AgentAddedSuccess.png"/>
</p>

## Step 4: Enable vulnerability detection
- As per Wazuh documentation, to enable vulernability detection a setting in the wazuh settings file on the server must be changed. This file is located at '/var/ossec/etc'.

## Step 5: Install Invoke Atomics
## Step 6: Create security events with Invoke Atomics.

