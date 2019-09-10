---
title: "A Better CMDB Integration"

---


Over the past 5 years, I’ve continued to hear people say the same thing about their organizations CMDB - “No one uses it because all the data in there is junk”.  As I’ve dug deeper into the underlying cause of failed CMDB implementations, one of the common trends I’ve seen is a third-party application dumping bad data into ServiceNow.

When I started a new CMDB integration several months ago, one of my top priorities was to prevent this from happening and creating thousands of unwanted CI’s.  I wanted to share how I solutioned this to ensure that bad data was kept out and we maintained the out of the box functionality in ServiceNow.  I should also add the part of the scope of this integration was to eBond to a third-party application as well.

The first piece of this solution was to find a way to limit what data was coming in and ensure the external application could not create CI classes/product models outside of the scope of the integration.  To achieve this, I created what I called a “Class Mapping table”.  This provided a data driven way to determine what classes and product models would appear on the cmdb_ci table.

This table contained the following fields:
-	Make 
-	Model
-	Resource type
-	Resource type
-	Class code
-	OS
-	Class
-	Product Model


This way, each time a new record was created on the staging table, an onBefore script is run to check if there is matching class mapping record to the staged record.  If not matching record is found, the record is ignored: 

```javascript
(function runTransformScript(source, map, log, target /*undefined onStart*/ ) {


	// Add your code here

	var gr = new GlideRecord("u _class_mapping");
	gr.addQuery("u_make", source.u_make);
	gr.addQuery("u_model", source.u_model);
	gr.addQuery("u_class_code", source.u_class_code);
	gr.addQuery("u_resource_type", source.u_resource_type);
	gr.addQuery("u_os", source.u_os);
	gr.query();
	if (!gr.next()) {
		log.info('A matching class mapping was not found for this record');
		ignore = true;
	}
	


})(source, map, log, target);
```

Using a data driven solution like this can prevent you from having to update the code going forward and make it easier to make changes.  The other advantage to taking this approach is not only are you preventing junk records from being created on the cmdb_ci table, but preventing new CI classes and product models from being created on the fly.  Allowing for integrations to create these can lead to a massive clean-up effort down the line. 

The other major issue I’ve seen with third party CMDB integrations is completely altering a ServiceNow instance to match what a third party application is doing.  To work around having to create dozens of new priorities, status’, categories and subcategories, I used what I call a “Value Conversion table” to get around this.  The concept of this table is to lookup values sent from a third-party system and convert them into “ServiceNow language.”

I created this table with 4 main fields;
-	Direction (Outbound, Inbound)
-	Type (Priority, Department, Category, Subcategory, Status, Reason)
-	Key
-	Value

Similar to the class mapping table, this provides a data driven way to translate values and prevent code updates when things change in the future.  This also makes it easier to visualize how values are being converted in the integration.  When transforming data or sending outbound calls to the third-party system, a simple script as below can be used to do the conversion:

```javascript
	getValueConversion: function(utype,udirection,ukey){
			var value;
			var conversionValues = new GlideRecord("u_value_conversions");
			conversionValues.addQuery("u_type", utype);
			conversionValues.addQuery('u_direction', udirection);
			conversionValues.addQuery('u_key', ukey);
			conversionValues.query();
			
			if (conversionValues.next()) {
				value = conversionValues.u_value.toString();
			}
			
			return value;
		},
```

Although I would not recommend over utilizing data driven solutions, it can definitely make integrations easier to maintain and update for the next develop to come along.
