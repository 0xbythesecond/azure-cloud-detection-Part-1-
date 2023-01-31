# Azure Cloud Detection 🕵️

<h2>In this lab, we will walk through following:</h2>
<ul>
<li>Configure and Deploy Azure Resources such as Log Analytics Workspace, Virtual Machines, and Azure Sentinel.</li>
<li>Implement Network and Virtual Machine Security Best Practices.</li>
<li>Utilize Data Connectors to bring data into Sentinel for Analysis.</li>
<li>Understand Windows Security Event logs.</li>
<li>Configure Windows Security Policy Settings.</li>
<li>Utilize KQL to query Logs.</li>
<li>Write Custom Analytic Rules to detect Microsoft Security Events.</li>
<li>Utilizing MITRE ATT&CK to map adversary tactics, techniques, detection, and mitigation procedures</li>
</ul>

#

<b>Step 1: Create an Azure Account:</b>
<br />
Use this <a href="https://azure.microsoft.com/en-us/free/"> link </a> to create your Account. This process will automatically set up your account and associated Azure Subscription.

<b>Step 2: Create a Resource Group:</b>
<br />
When working with Azure Resources, Resource groups are logical containers for all our resources. We will first create a resource group to group all the resources we will use for this lab. These resources will include a Windows 10 VM, Log Analytics Workspace, and an Azure Sentinel Resource.

Search “Resource Group” in the Azure Portal Search Bar and Follow the on-screen prompts for the basic tab. Tags will not be required for this lab, so we can move forward by selecting to 'Review + Create' to move forward. Once that's done, you can choose to 'Create' to finalize the creation of the Resource Group. 
<p align="center"> <img src="https://i.imgur.com/RQqOGqR.png" height="50%" width="50%" alt="Create Resource Group"/></p>

Select 'Create' here
<p align="center"> <img src="https://i.imgur.com/7vjp64F.png" height="50%" width="50%" alt="Create Final Step of Resource Group"/></p>

<b> Step 3: Deploy a Virtual Machine (VM) </b>

In this lab, we will be collecting our data from a Windows Virtual Machine. To deploy a Virtual Machine you can do a quick search in the Azure portal search bar for "Virtual Machine" and once Virtual Machine is selected, you will then choose 'Create' to begin the steps of creation. 

#### Click Create:
<p align="center"> <img src="https://i.imgur.com/tO1HB49.png" height="50%" width="50%" alt="Create Virtual Machine"/></p>


Use the resource group created in the first step and fill out the required field to create your virtual machine. In the image above, the (US) East US is selected as the region that will house the Virtual Machine. 

<h6> Note: When selecting your region, keep in mind that some virtual machines may not be available and cost will vary depending on the region selected. </h6> 

Use all the default settings on the Basics Tab and fill in the appropriate field.

<h6>*Please remember your admin username and password as this is how you will authenticate to the Virtual Machine.</h6>

For this lab the default settings in Disks, Networking, Management, Advanced, and Tags are sufficient. We will make the appropriate network changes later.

Click Review + create to start the creation of your virtual machine. After selecting to 'Review + Create', you will see a summary of the what has been selected in creating the VM. 

<h6>Important: Be sure to select the check box confirming 'I confirm I have an eligible Windows 10 license with multi-tenant hosting rights'.Without this selected, it will not allow the validation to process as "passed" successfully. </h6>

<p align="center"> <img src="https://i.imgur.com/FzvKNU1.png" height="50%" width="50%" alt="Select default settings for Virtual Machine"/></p>

<p align="center"> <img src="https://i.imgur.com/XNwwt6D.png" height="50%" width="50%" alt="Select checkbox to confirm windows License"/></p>

Once you have selected to 'Create' in the confirmation page, you will then be presented with messaging showing that 'Your deployment is complete.'

<p align="center"> <img src="https://i.imgur.com/vQVWHrq.png" height="50%" width="50%" alt="Create VM Final"/></p>
  
<p align="center"> <img src="https://i.imgur.com/4WWjAUg.png" height="50%" width="50%" alt="Confirm VM Creation"/></p>

When you deploy a virtual machine in Azure, that virtual machine is placed on a Virtual Network (vnet). Your Virtual Machine is assigned an IP address on that network as well as a network interface. Another Azure security feature that is implemented with the default settings we used are Network Security Groups (NSG). A NSG is used to filter network traffic to and from Azure resources. Similar to a firewall, filtering is based on rules that dictate source des andtination ports as well as the network protocols that are allowed or denied.

If we go to back to our resource group we created earlier we can see the Virtual Network and NSG listed as resources.

<p align="center"> <img src="https://i.imgur.com/hRNvlwM.png" height="50%" width="50%" alt="Displays the NSG in the RG"/></p>

If we select our NSG we can see the default rules.

<p align="center"> <img src="https://i.imgur.com/f76zN77.png" height="50%" width="50%" alt="Displays default NSG settings"/></p>

Previously when we were creating our virtual machine we enabled this setting:

<p align="center"> <img src="https://i.imgur.com/yAz3tYz.png" height="50%" width="50%" alt="Displays inbound port rules"/></p>

If you look at the first rule (image above), you will see that inbound RDP traffic is allowed from any source to any destination. RDP is necessary to access our VM. However, with this current setting, anyone who obtains our public IP (which can be possibly be obtained via network scan) can potentially connect to the VM as this is public facing. This presents a security risk as it makes the VM vulnerable to possibly a brute force or password spray attack.

In order to reduce our attack surface, we need to enable a security feature called 'Just in Time' access. You can read more about this at the following <a href="https://docs.microsoft.com/en-us/azure/defender-for-cloud/just-in-time-access-usage?tabs=jit-config-asc%2Cjit-request-asc">link</a>.

Essentially what this feature does is only provide access to our Virtual Machine when necessary via time-based restrictions as well as implements the principle of least privilege by giving the option to restrict access to certain IP’s as well as RBAC roles.

Anyone who wants access to the VM will need to request and based on their IP and assigned role they would be granted or denied access. By default when creating your Azure Account you are a Global Administrator so upon request you will be granted access to the VM. To set this up we will perform the following steps:

Search "Microsoft Defender for Cloud" in the search bar at the top of the Azure Portal and select the service. You will see a page similar to this. On the left pane select “environment settings”

<p align="center"> <img src="https://i.imgur.com/2xkkrn0.png" height="50%" width="50%" alt="Select Environment Settings"/></p>

Select your Azure Subscription from the list provided. Upon selection, the following services will be seen on the screen. By default, Enhanced security is off but you will want to select the Enable All Microsoft Defender for Cloud Plans. You will be given a 30 free trial so be sure to disable when finished with the lab to avoid any cost. You can select then “Enable all option” and hit save.

<p align="center"> <img src="https://i.imgur.com/3HgsPyd.png" height="50%" width="50%" alt="select subscription for defender for cloud"/></p>

<p align="center"> <img src="https://i.imgur.com/GGsWmBq.png" height="50%" width="50%" alt="Enable All Microsoft Defender for Cloud Plans"/></p>

After enabling the plan, navigate back to the homepage for Defender for Cloud and select 'Workload Protections' on the left pane. That will then present the following screen:

<p align="center"> <img src="https://i.imgur.com/8nVRGqD.png" height="50%" width="50%" alt="Enable All Microsoft Defender for Cloud Plans"/></p>

Now, we will go back to our VMs that we created previously and then select 'labvm'. Once labvm is selected, we will choose 'Connect' on the left panel. Upon reaching the connect configuration page, we will select the 'Enable Just-in-time' button. 

<p align="center"> <img src="https://i.imgur.com/8Axd4aF.png" height="50%" width="50%" alt="Enable Just-in-time button"/></p>

<p align="center"> <img src="https://i.imgur.com/v1tc2BX.png"  height="50%" width="50%" alt="Enable Just-in-time button confirm"/></p>


To confirm the change that we have just created, we will go to the 'Networking' option on the left panel to view the rules that have been set in place. At the top of the provided rules table, we can now see that the Just-In-Time 'Security-Center-JITRule' has been implemented and will be executed before the RDP due to rule prioritization.

<p align="center"> <img src="https://i.imgur.com/Sy35VVI.png" height="50%" width="50%" alt="Networking Rules Show JIT "/></p>

Now that your JIT has been enabled, we will go to your VM settings and click Connect on the left pane. Select “My IP” as Source IP Request Access. Select “Request access”.

<p align="center"> <img src="https://i.imgur.com/dUNYnXs.png" height="50%" width="50%" alt="Request Access"/></p>


If we go to the networking tab for our VM we can see our rules have been updated. Now RDP traffic is allowed for a certain amount of time only from the IP of your computer. Anyone else who attempts to establish and RDP connection will be blocked via our Just in Time Access rules.

<p align="center"> <img src="https://i.imgur.com/JovRj9g.png" height="50%" width="50%" alt="RDP is allowed for certain traffic"/></p>

Step 4: Create Log Analytics Workspace and Deploy Sentinel

When working with Log Data in Azure we need somewhere to store/operate that data. Log Analytics workspace is used to collect and store log data from Azure Resources.

To configure a Log Analytic workspace:

Search “Microsoft Sentinel” in Search Bar in Azure Portal. This will prompt you to create a Log Analytics Workspace.

Use the same resource group and region used for the Azure Virtual Machine you created in the previous step when filling out the necessary fields to create your Log Analytics workspace.


<p align="center"> <img src="https://i.imgur.com/jcBrBCe.png" height="50%" width="50%" alt="Create Log Analytics Workspace"/></p>

Click “review + create” to create the Log Analytics Workspace. Then choosing "Create" to finalize the creation of the workspace, you are presented with the following page. 

<p align="center"> <img src="https://i.imgur.com/SEvU6Ik.png" height="50%" width="50%" alt=" Review-Create Log Analytics Workspace"/></p>

After creating the Log Analytics Workspace search 'Sentinel' in the search bar.

<p align="center"> <img src="https://i.imgur.com/ggqb8X8.png" height="50%" width="50%" alt=" Review-Create Log Analytics Workspace"/></p>

Scroll to the bottom of the page and select Add.

<p align="center"> <img src="https://i.imgur.com/DMY2CLx.png" height="50%" width="50%" alt=" Select Add button"/></p>

<a href="https://github.com/0xbythesecond/getting-data-into-sentinel"> Part 2: Getting Data into Sentinel </a>






















  




