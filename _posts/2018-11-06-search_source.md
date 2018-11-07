---
title: " Expanding Service Portal Search"

---


Out of the box, the ServiceNow Service Portal queries the service catalog and knowledge base.  While end user’s main objective on the portal is to submit incidents and requests, fulfillers need to access various other tables such as Change, Problem, CMDB, etc.

To give fulfillers a better Service Portal experience, we can expand this search to include these tables. This can be achieved through the addition of a simple scripted search source.  While the “no code” version of creating search sources is useful, it has some shortcomings.  For example, if you are to create a search source, set the table as incident, and set short description as a display field, is will also query the words in the short description field, which will degrade the ITIL user search experience when trying to order items from the service catalog.

The example below demonstrates how we can add a search source for change numbers, but this can be easily expanded to other tables as well.

1. Navigate to Service Portal and select the portal which you want to add a search source to.
2. Scroll to the bottom of the form and locate the “Search Sources” related list.
3. Click “New”
4. Name your new search source “Change Requests”.
5. Give your search source a higher order than the OOB search sources.
6. Make sure you add the necessary roles to the search source so it does not display to end users.
7. Scroll to the bottom of the form and select the “Is scripted source” checkbox.
8. In the “Data fetch script”, copy and paste the following code:
(function(query) {
  var results = [];

```javascript
(function(query) {
	var results = [];
	/* Calculate your results here. */

	//query change_request table to find changes containing the entered number in the query
	var gr = new GlideRecord("change_request");
	gr.addQuery("number", 'CONTAINS', query);
	gr.query();
	if (gr.next()) {
		var item = {};
		item.table = 'change_request';
		//primary field to be displayed
		item.primary = gr.getDisplayValue('number');
		item.sys_id = gr.sys_id;
		results.push(item);
	}


	return results;
})(query);
```

9. On the typeahead tab, check the “Enable Typeahead” and set the page field to “Form”.  This will leverage the OOB form widget.

<a href="/assets/images/ Screen Shot 2018-11-06 at 10.21.41 PM.png
"><img src="/assets/images/Screen Shot 2018-11-06 at 10.21.41 PM.png"></a>
