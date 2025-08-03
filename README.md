
# Cloud-Based SOC & Honeynet using Azure Sentinel

This project aims to build a Cloud-based Security Operations Center (SOC) integrated with a Honeynet on Microsoft Azure to detect, analyze, and respond to real-world cyber threats. By deploying decoy assets like VMs, SQL databases, and storage services, it attracts attackers to controlled environments. All activities are monitored through Log Analytics Workspace and visualized using Azure Sentinel (SIEM). The system enables real-time incident tracking, threat mapping, and alerting. It provides security teams with deep insights into attacker behavior and helps strengthen the overall security posture through proactive defense.


### Key Objectives:

->Deploy honeypots (VMs, databases) to attract attackers and study their behavior.

->Collect logs and telemetry from various resources.

->Use Log Analytics Workspace to centralize data.

->Analyze threats in Azure Sentinel for real-time incident detection and response.




## Architecture

<img width="1027" height="574" alt="image" src="https://github.com/user-attachments/assets/56219f6c-6d31-4a45-afee-d082f6cbdd71" />


## Deploying and Configuration of Components 

### 1)Windows & Linux VMs

Open VMs option in azure and dive into configuration of creating  VMs .

In  basic detail , we need to create new resource and make sure to include rest other components in same resource group , let say it is RG-Lab.

Also we make sure they come under same zone and region.

Then create  credentials  and move further for managing networking to create subnet and virtual IP. Finally deploy VMs.

### 2)Create Nsg

Select windows or Linux nsg after deploying and move to configure inbound security rules .

Remove default rule of RDP  to make these NSG as  vulnerable as possible .

Add rule of any soruce , any destionation , any port action are allowed so we can lure traffic here . 

Create nsg flow log by selecting resource as linux & windows vm

### 3)Configuration of windows VMs

Use of remote desktop connection to boot in virtual machine with credintail earlier created.

Aim to create windows VM as vulnerable as possible , after booting , move to configure Firewall .

Go to windows defender firewall and turn off Domain profile , Private profile , Public profile Firewall state 

Now Install SQL database  and SQL server Management studio(SSMS) in Windows VMs  and remember credentials to log into  SSMS later . 

Now set full permission of SQL server through registory editior . 

Go to  computer → HKEY_LOCAL_MACHINE→ SYSTEM → CurrentControlSet → Services → Eventlog → Security 

In properties of Security add user /group as network service .

Go to secpol.msc turn enable application generated log for success and failure both . 

Finally run up SSMS to connect to SQL server with credentials and after connecting , go to properties and turn on login auditing for both success and failure .

Thus we make sure to create vulnerable windows machine and turn on all possible types of log we can create .
### 4) Attack-VM

Create windows based attack VM with same process as earlier but for these VM we create different resource group let say RG-attack , but make sure they are in same region and zone.

These is created to simply demonstrate basic RDP attack with credential we know for Windows-VM we created , but  we make mulitple failed login attempt to generate log event in windows-VM . 

We can do same for SQL server , by installing SSMS in attacker VM and trying to login in SQL server with credentials . To generate log with attempt multiple failed login and also successfull login so we can generate good bunch of log from them .

### 5) Log Analytics workspace
Centralized data collector that aggregates logs from all above components.

Create Log analytics in same group as RG-Lab and deploy the instance.

Go to Sentinel and add log analytics worksapce to it .

Go to watch list and add geo-ip csv file which is curated file of locationthat  map with ip address to locate different attacking vector location .

Go to Agents and check computers connected to it .If they are not connected via azure model rather it is log analyatics , then create data collection rule.

Create instance of data collectionrule  and in resources section  add Linux & windows VM . In collection & deliver section specify the different types of log required to analysis for both VMs . and in custom sub-cateogory add the following xpath query .

After successfully connecting the flow of log of VMs to log analytics workspace , you can check via writing KQL query in log section to get desired log like Syslog or SecurityEvent.

### 6)Defender for Cloud

Go to environment setting → LAW-RG-lab 

Enable defenders plan for Servers and SQL server on machines and 

Enable Data collection for All security events on windows 

Go to environment setting → azure subscriptions

Enable defenders plan for Servers , Databases , Storage , Key-Vault and 

Within Servers , configure the Log analystics agent as to direct auto-provisioning configuration to select LAW-RG lab workspace for all events 

Enable continous export for log analytics workspace  to export all data types mentioned and directed to LAW-RG lab
### 7)Storage Accounts
Azure object storage solution that might store sensitive or luring data for honeypots (e.g., fake credentials, logs).

Create storage account in same as RG-Lab and deploy the instance .

Then go to diagnostic setting of instance created and select blob storage account  and add  diagnostic setting where you specify all cateogry log you want to collect  and send it to log analystics workspace .

For testing purporse , we can add container to storage account to genrated logs 

### 8) Active Directory
Active Directory manages user identities and controls access to Azure resources. It acts as the identity provider.

Go to Microsoft Entra ID then go to Diagnostic setting . 

Add diagnostic setting for AuditLog & SigninLogs to destination Log analytics workspace.

Then go to users section to create new user as global administrator  for testing.

we can test these new user by trying to login with credentails we created the user , but  we make mulitple failed login attempt to generate log event in Log analytics workspace

### 9) Monitor

Go to Micosoft montior these basically log all activty on mircosoft azure you do . 

Go to activty log  then export these logs by adding diagnostic setting and specify all cateogry log you want to collect  and send it to log analystics workspace.

### 10)Key Vault
Key Vault safeguards secrets like API keys, passwords, certificates. Can be monitored for unauthorized access attempts.

Go to Micrsoft key vault  and create new instance of it .

Then go to Key section of instance created and create a new key if type like RSA or EC . 

Then go to Secret section of instance created and create new secret value.

### 11)Sentinel
Sentinel is Security Information and Event Management system(SIEM) tools.It uses data from Log Analytics to:

Detect threats

Generate alerts

Visualize incidents

Automate responses

Go to Microsoft Sentinel then go to workbook 

Here we will create workbook of different section from where we could have generated Log from all endpoint placed in public network.
Those section are  :

Ms-SQ-auth-fail 

Linus-ssh-auth-fail

nsg-malicious-allowed-in

windows-rdp-auth-fail

All of these section has  query to generate map of ip location from geo-ip we have donwload and these query are uploaded in files as json formate.


## Hardening of components

#### VMs
-> Turn back on firewall in windows

-> Limit access using NSG rules, thus remove inbound rule we created and add required inbound rule only

-> Enable endpoint protection/antivirus.

#### Key Vault
->Grant least privilege access.

->Enable logging and alerting on secret access.

->Use managed identities securely.

#### Blob Storage
->Enable private access only (no public blobs).

->Use encryption at rest and in transit.

->Enable access logging.



## Attack Visualization 

<img width="904" height="443" alt="image" src="https://github.com/user-attachments/assets/d064983c-c6bd-4fe9-b9e8-317e0c114102" />

<img width="728" height="464" alt="image" src="https://github.com/user-attachments/assets/2d6eba90-82c8-447f-afda-db7c0371266d" />

<img width="735" height="460" alt="image" src="https://github.com/user-attachments/assets/2c7972f1-a0b2-46f5-b76f-a1c72f8261bb" />


## Metrics Before Hardening / Security Controls

The following table shows the metrics we measured in our insecure environment for 24 hours:

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 6371
| Syslog                   | 533
| SecurityAlert            | 4
| SecurityIncident         | 43
| AzureNetworkAnalytics_CL | 831

## Metrics After Hardening / Security Controls

The following table shows the metrics we measured in our environment for another 24 hours, but after we have applied security controls:


| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 2968
| Syslog                   | 6
| SecurityAlert            | 0
| SecurityIncident         | 0
| AzureNetworkAnalytics_CL | 0

