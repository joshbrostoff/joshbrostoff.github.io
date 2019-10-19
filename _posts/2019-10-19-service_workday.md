---
title: "Integrating ServiceNow and Workday"

---

Before the London release of ServiceNow, an out of box SOAP integration with Workday was offered.  However, with this pre-built integration now deprecated, a lot of customers are still leveraging the old SOAP integration, and new ones stuck with figuring out how to build a new one.  Truth be told, the days of SOAP integrations are over.  In various ways which I wont dive into here, REST is a far preferred method for building modern SaaS integrations.

As many ServiceNow customers still look to integrate ServiceNow and Workday for various use cases, I wanted to share my recommended way of building this integration.

The first piece of connecting ServiceNow to Workday is to build what is known as a Workday Report-as-a-Service (RaaS).  This is a feature that exposes reports as web services. These reports must be configured as Advanced type reports to be web service enabled.
Building a RaaS is an extremely easy process that involves no coding and can be built in the matter of minutes.  To start building a RaaS in Workday, navigate to

1.	Find URL in Actions > Web Service > View URLs.

2.	Right-click JSON and choose "Copy URL"

3.	JSON endpoint example:

https://wd2-impl-services1.workday.com/ccx/service/customreport2/workato/workato/Investors?format=json&Worker_Type!WID=d588c41a446c11de98360015c5e6daf6&Base_Pay=0

4. Filter parameters
Prompts behave like request parameters. In the UI, it is presented as input fields before generating the actual report.  As a REST endpoint, these prompts are passed as request parameters. To do so, you have to set the report type and configure prompts. You can also define filters for your prompts.

5. Report type
Switch to advanced type if not already. Only Advanced custom reports can be used in RaaS.

6. Add prompts
Add all default prompts that are required of web-service-enabled reports. Add additional prompts as desired.

Once you have built out your RaaS, you are now ready to start consuming data from Workday and ingesting it into ServiceNow.  Here is how to do this:

1.	In the HR Core scope, create a new table that extends the import set row table called “Workday Workers Staging” (or whichever table from Workday you are retrieving).

2.	Create the fields on the new staging table which you will be pulling back.

3.	Create a transform map that maps the data from the new table you created (u_workday_workers_staging) to the HR Profiles table (sn_hr_core_profile) table.

4.	Create a new scheduled job that runs a script to call our to the RaaS, parse the JSON, and map the retrieved payload data to your fields on your staging table.  An example of this can be seen in the code snippet below:

 
```javascript
var method = "get";
//update with your RaaS endpoint
var endpoint = '{RaaS endpoint}’;

//create system properties with Workday username and password
var user = gs.getProperty('sn_hr_core.workday.user');
var password = gs.getProperty('sn_hr_core.workday.password');

var rm = new sn_ws.RESTMessageV2();
rm.setBasicAuth(user, password);
rm.setHttpMethod(method);
rm.setEndpoint(endpoint);
var body = '';
rm.setRequestHeader("Content-type", "application/json");
rm.setRequestBody(body);
var response = rm.execute();

var responseBody = response.getBody();
var jsonData = JSON.parse(responseBody);
for (var i = 0; i < jsonData.Report_Entry.length; i++) {
	var property = jsonData.Report_Entry[i];

	var gr = new GlideRecord('u_workday_users');
	gr.initialize();
	gr.setValue('u_name', property.Full_Name);
	gr.setValue('u_mgr _id', property.manager_id);
	gr.setValue('u_employee_type', property.emp_Type);
	gr.setValue('u_education', property.education);
	gr.setValue('u_ee_ethnicity', property.ethnicity);
	gr.setValue('u _home_address_state', property.Home_Address_State);
	gr.setValue('u_gender', property.Gender);
	gr.setValue('u_manager', property.Manager);
	gr.setValue('u_business_title', property.businessTitle);
	gr.insert();
  
}
```
