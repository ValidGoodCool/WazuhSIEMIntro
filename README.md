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
- As per Wazuh documentation, to enable vulernability detection a setting in the wazuh settings file on the server must be changed. This file is located at `/var/ossec/etc`.

<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/VulnScanOnConfig.png"/>
</p>

- After Vulnerability Scan setting has been enabled. Confirm funcionality is now available.

<p align="centre">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/ConfirmVulnDetect.png"/>
</p>

## Step 5: Install Invoke Atomics
- Invoke-Atomic is a automated cybersecurity red team test framework. It is utilised to find vulnerabilities on a system. I will employ this tool to test the vulnerabilties of my linuxagent vm and the abilties of Wazuh to display these in a meanful way.
- Install powershell classic on linux vm:
`sudo snap install powershell --classic`
- Open powershell by entering `pwsh` in terminal.
- Copy and run the following script from the Invoke-Atomics github to import the test framework

<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/InstallAtomics.png"/>
</p>

## Step 6: Create security events with Invoke Atomics.
- Create security events with Invoke-Atomics. Here I used a mitre test technique common on linux environments.

<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/InvokeAtomicTestCompleted.png"/>
</p>

- I then confirmed the events were detected by the Wazuh EPD agent in the Wazuh dashboard. This confirms success.
<p align="centre">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/EventsDetected.png"/>
</p>

## Step 7: Set up Automated Response with VirusTotal

- The Wazuh documentation includes a guide to set up detection and removal of amlware using the VirusTotal API. The following follows this guide.
- Enable File Integrity Monitoring by accessing the Wazuh Agent's `ossec.conf` file in the `/var/ossec/etc` folder.
<p align="left">
    <img src="PIC HERE"/>
</p>

- Add `/home/nick/Downloads` folder path to the File Integrity Monitoring section
<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/AddFolderActiveFileMonitoring.png"/>
</p>
- Install `jq`, a utility that processes JSON input from the active response script:

```
sudo apt update
sudo apt -y install jq
```
- Create `remove-threat.sh` file in the `/var/ossec/active-response/bin` folder. Paste in the following bash script provided by Wazuh and save file:
```
#!/bin/bash

LOCAL=`dirname $0`;
cd $LOCAL
cd ../

PWD=`pwd`

read INPUT_JSON
FILENAME=$(echo $INPUT_JSON | jq -r .parameters.alert.data.virustotal.source.file)
COMMAND=$(echo $INPUT_JSON | jq -r .command)
LOG_FILE="${PWD}/../logs/active-responses.log"

#------------------------ Analyze command -------------------------#
if [ ${COMMAND} = "add" ]
then
 # Send control message to execd
 printf '{"version":1,"origin":{"name":"remove-threat","module":"active-response"},"command":"check_keys", "parameters":{"keys":[]}}\n'

 read RESPONSE
 COMMAND2=$(echo $RESPONSE | jq -r .command)
 if [ ${COMMAND2} != "continue" ]
 then
  echo "`date '+%Y/%m/%d %H:%M:%S'` $0: $INPUT_JSON Remove threat active response aborted" >> ${LOG_FILE}
  exit 0;
 fi
fi

# Removing file
rm -f $FILENAME
if [ $? -eq 0 ]; then
 echo "`date '+%Y/%m/%d %H:%M:%S'` $0: $INPUT_JSON Successfully removed threat" >> ${LOG_FILE}
else
 echo "`date '+%Y/%m/%d %H:%M:%S'` $0: $INPUT_JSON Error removing threat" >> ${LOG_FILE}
fi

exit 0;
```
- Change the ownership and permissions of this bash script:
```
sudo chmod 750 /var/ossec/active-response/bin/remove-threat.sh
sudo chown root:wazuh /var/ossec/active-response/bin/remove-threat.sh
```
- Restart Wazuh Agent to apply changes:
`sudo systemctl restart wazuh-agent`

- Now that the Wazuh ubuntu agent is set up, we need to configure the Wazuh Server.
- Add the following rules from the Wazuh Documents to the `/var/ossec/etc/rules/local_rules.xml` rules file. These rules alert about changes in the `/Downloads` directory that are detected by FIM scans.

<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/WazuhServerFIMServerRules.png"/>
</p>

- Add the following rules to the `/var/ossec/etc/ossec.conf` which includes my api key for VirusTotal:

<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/WazuhServerVirusTotalRules.png"/>
</p>

- Append the following block rules to the `/var/ossec/etc/ossec.conf`. This enables Active Response and triggers the remove-threat executable when the VirusTotal query returns positive matches for threats
<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/WazuhServerRemoveThreatRules.png"/>
</p>

- Add the following rules to the Wazuh server `/var/ossec/etc/rules/local_rules.xml` file to alert about the Active Response results:

<p align="left">
    <img src="https://github.com/ValidGoodCool/WazuhSIEMIntro/blob/main/WazuhServerActiveResponseResultsRules.png"/>
</p>

- Restart the Wazuh Server service:
`sudo systemctl restart wazuh-manager`
