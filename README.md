# Platform Request Monitor Tool (experimental)
Automatically import PPR stats into a Model Driven App for daily monitoring.
Today, Power Platform Admins can access Platform Request consumption reporting by submitting a request for report by Licensed Users, Unlicensed Users and Per Flow plan. 
Once the report is generated, Admins can download the CSV file. However due to privacy reasons, the report only stores the relevant User's AAD Object Id or the Flow Id. 

This tool automates the request for reports and extracts the data from CSV into a Dataverse table daily, upon which Admins can access and review from a Model Driven App. 
Additional metadata has been added to resolve the User's AAD Object ID to friendly name and user principal name, as well as automatically constructing a deep-link to the Flow for Per Flow report.
The solution uses DataFlow to import records from CSV file and resolving the User name for each record, and Power Automate to initiate the report request and orchestrate the DataFlow import.

Today, this tool only extracts report by Licensed User and for Per Flow plans. Unlicensed user report will be added in the future.

Caution: This tool is provided as-is and is unsupported. 
It is also using an undocumented API - this API may change, and therefore the solution will require changes in accordance to the changes to the API, or may be unusable in the future.

## Setup Guide
1. Download the solution file (PlatformRequestMonitor_1_x_x_x_managed.zip). https://github.com/fooshen/PlatformRequestMonitor/blob/main/PlatformRequestMonitor_1_0_0_1_managed.zip

2. Go to your Power Apps Maker portal and select your desired Environment for this solution. This solution is intended for Admins, so pick a suitable Enviroment, for example your CoE Environment. Copy your existing Tenant ID and your Instance Url values - we will need these later. While in the Power Apps Maker portal, click on the gear icon and select "Session details". Copy the tenant Id and Instance Url values from the session details dialog.
<img width="400px" src="https://user-images.githubusercontent.com/23041800/203438206-991f8772-8f25-4ca3-aef0-c4d2672ab300.png" />
<img width="400px" src="https://user-images.githubusercontent.com/23041800/203440728-885625a6-3baf-4c63-8599-ac30e02a5013.png" />

For the Instance Url, only copy the orgid with the CRM.dynamics.com domain name. Exclude the preceeding "https://" and the trailing "/" at the end.

3. Import the solution in your selected Environment.
<img width="400px" src="https://user-images.githubusercontent.com/23041800/203444192-8f194f1f-be57-4133-8fe2-7448f4548a55.png" />

4. You will be required to map or create 3 new connections.
<img width="450px" src="https://user-images.githubusercontent.com/23041800/203438509-7860185a-4017-4c24-99a3-b9093fdc6653.png" />

For the HTTP with Azure AD connector, you will need to use the following values for both the Base Resource URL and Azure AD Resource URI. Type in "https://licensing.powerplatform.microsoft.com".
You must sign in using your Power Platform Service Admin or Dynamics 365 service Admin or Global Admin role create the connector.

<img width="400px" src="https://user-images.githubusercontent.com/23041800/203438594-3ef6578b-9331-4d02-976d-9cedb974548e.png" />

5. Next, you will need to enter your Tenant ID copied from step 2 above to the Environment variable for "Tenant Id"
<img width="400px" src="https://user-images.githubusercontent.com/23041800/203438846-b6c1e3f3-f0a9-46c5-981d-2a1745aa70d1.png" />

6. Click import. Once import is completed, you should see the solution "Platform Request Monitor". Click into the solution. You will see the following components:
<img width="600px" src="https://user-images.githubusercontent.com/23041800/203446341-e6de457f-28af-4cee-a642-92d9a7565e5c.png" />

Be sure to turn on any cloud flows that is Off.

7. Today, DataFlows do not support connection references and Environment variables yet. We will need to remediate the connections in DataFlow. Before we do that, we need to initiate a "First time run" to populate the initial requests into Dataverse tables so that we can perform the subsequent DataFlow configuration steps.
Locate the Cloud Flow CFMgr|First Run. Turn on this Flow, then click "Run" to execute the first run. Wait until the run has completed successfully, then proceed to the next step.

8. Navigate away from the solution view, and head to Dataverse -> DataFlows in Power Apps Maker portal. You should see two DataFlows as depicted in the screen below.
<img width="500px" src="https://user-images.githubusercontent.com/23041800/203440360-6880403a-e36f-4f00-93f3-e5e42fb324f1.png" />

9. Edit CRMgr Import Flow Report. Locate the Parameter "OrgId". Enter your current Environment's Org instance Url copied from Step 2 above and click "Apply".
<img width="400px" src="https://user-images.githubusercontent.com/23041800/203441136-bbe195ce-ae45-46d3-ae8d-c892f86e7617.png" />

10. Now click on the Get User Consumption Report query and click "Configure connection"
<img width="500px" src="https://user-images.githubusercontent.com/23041800/203441253-1910e68b-d1ff-460e-8d3b-cf30e7152515.png" />

Select "Organizational account" and click Sign In. 

<img width="400px" src="https://user-images.githubusercontent.com/23041800/203441543-61ee74a0-b4b0-4762-b623-569279b183e5.png" />

Click Next to proceed, and finally "Publish".

11. Perform the same for the second Flow - CFMgr Import User Report. Update the OrgId parameter with your Instance Url copied from Step 2 above (remember not to include the preceeding "https://" and the trailing "/" at the end).
For the query "Get User Consumption Report", click Configure connection. You will just need to select the same connection created in Step 10 and click Next, and finally "Publish".

12. Lastly, edit the Cloud Flow CFMgr|Import Report. You will need to expand the Switch block and update the DataFlow actions "Import User Report" and "Import Flow Report" to resolve your current Environment and DataFlows that were published in steps 10 and 11 above.
![image](https://user-images.githubusercontent.com/23041800/203449941-b19a759b-6ee8-4321-9c22-bcc08e32df7b.png)


That's all - you have now set up the solution which will now refresh the Platform Request report every 24 hours. 
To modify the time when the daily report import runs, select the cloud flow CFMgr|Daily Run and change the time when this executes once a day. This is currently set to run at 12.30 AM.
A clean up Flow - CFMgr|Cleanup will delete data older than 7 days ago. This will run every 2 days.

Launch the Model Driven App "PPR Monitor". Select the report by date and view the Report Details (Licensed User view includes User Name and Email, while Flow view includes deep link to the Cloud Flow).
![image](https://user-images.githubusercontent.com/23041800/203449180-f6926193-4160-4972-9ae8-545c423f869a.png)

You can create custom Automated Cloud Flows to send email notification or other alerts on the Dataverse Table CFMgr Report Detail. There are calculated columns for Overage and Percentage Consumed that you may use - for example, send alert when a user or a flow exceeds 80% of consumed Platform Request entitlement. 


