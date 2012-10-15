ixDbEz
======

IndexedDB EZ is a js wrapper for IndexedDB providing rapid client-side development of IndexedDB databases with the possibility of easy data synchronization hooks.  
Version2 adds hooks for a new new javascript library called ixDbEzSync.  This library is not publically available just yet.  For now, when you create a ixDbEz database,
just set the useIxDbSync variable to false, and this functionality will be ignored. 

ixDbEz Examples
===============
All examples are written using Version 2 of ixDbEz.  

Create an ixDbEz Database
=========================

This code can be added to the window.onload event.

    //Define the local IndexedDB structure in a function
    //This code ONLY executes anytime: 
    //    1.  The database does not exist.
    //    2.  The version number passed to StartDB is greater than the 
    //         current version number.
    //    3   The version number passed to StartDB is not a valid integer.
      var ixdbDDL = (
        function () {
          //Create IndexedDB ObjectStore and Indexes via ixDbEz 
          ixDbEz.createObjStore("demoOS", "keyField", false);
          ixDbEz.createIndex("demoOS", "ixField1", "Field1", true);
        }
      );

    //Create or Open the local IndexedDB database via ixDbEz
    ixDbConnection = ixDbEz.startDB("demoDb", 2, ixdbDDL, undefined, undefined, true);

    
Add a record to the database created above
==========================================

    //define new record with users input 
    var newRecord = {};
    newRecord.keyField = "1";
    newRecord.Field1 = "This is";
    newRecord.Field2 = "a";
    newRecord.Field3 = "test";
                
    //add new record to the local database
    ixDbEz.add("demoOS", newRecord);	


Update a record in the database created above
=============================================
    ixDbEz.put('demoOS', JSON.parse("{ "keyField":"1" , "Field1" : "This is" , "Field2" : "an update" , "Field3" : "test" }");
	
	
Delete a record
===============

ixDbEz.delete('demoOS','1');


Get a single record in the database created above using the index created above and delete it
=============================================================================================
    
	//Create a callback function that runs when / if the cursor comes back.
    //ixDbCursorReq is the cursor request object passed back from the getCursor request.
	//Use ixDbCursorReq.onsuccess event to trigger any logic you need to run against the data that 
    //comes back in the cursor.	
    function myDeleteRecordCallback(ixDbCursorReq) {

        if (typeof ixDbCursorReq === "undefined") {
            //we have no cursor returned in the provided value for ixDbCursorReq.
        }
        else {
            
            ixDbCursorReq.onsuccess = function (e) {
                var cursor = ixDbCursorReq.result || e.result;    
                
                if (cursor) {
                    
                    cursor.delete();
                }
            };
        }
    }
	
	//The key range can be used to target 1 record or a range of records in your cursor query
    var keyRangeObj = IDBKeyRange.only("This is");
	
	//Notice I have used the "true" in the "readWrite" argument of getCursor to make sure we have 
	//write access on the table for the delete.
    ixDbEz.getCursor("ixDbSync", myDeleteRecordCallback, undefined, keyRangeObj, true, "ixField1");

	
Add a new record to the database table and then rebuild a div based on it's contents using ixDbEz.getCursor()
=============================================================================================================

    //define new record with users input 
    var newRecord = {};
    newRecord.keyField = "1";
    newRecord.Field1 = "This is";
    newRecord.Field2 = "a";
    newRecord.Field3 = "test";
                
    //add new record to the local database
    ixDbEz.add("demoOS", newRecord , function() { ixDbEz.getCursor("demoOS",getIxDbRecords); });	
	
	
	//1.  This function assumes you have a div on your webpage with id="clientRecords"
	//2.  When the a new record is added to the table (like above),  you clear out the div and 
    //    rebuild it to display the new data.	
	function getIxDbRecords(ixDbCursorReq) {
            var html = []; 
            var clientRecords = document.getElementById('clientRecords');

            //Clear out any old content
            clientRecords.innerHTML = "";

            if(typeof ixDbCursorReq === "undefined"){
                clientRecords.innerHTML = '<table id="clientDb-Results"><tr border="1"><th>Key</th><th>Value (JSON)</th><th></th></tr></table>';
                clientRecords.style.visibility = "visible";
            }
            else{
                //Also fires once each time cursor.continue() is called.
                ixDbCursorReq.onsuccess = function (e) {
                    var cursor = ixDbCursorReq.result || e.result;  // FF4 requires e.result.    
                    if (cursor) {
                          var cursorObjString = JSON.stringify(cursor.value);
                          html.push('<tr> ',
                                      '<td>', '<span class="dataValue" contenteditable onblur="ixDbEz.updateKey(\'demoOS\',\'' ,cursor.key ,'\', this.textContent, call_getIxDbRecords )">',cursor.key,'</span> </td>', 
                                      '<td class="dataCell" >', '<span class="dataValue" contenteditable onblur="ixDbEz.put(\'demoOS\', JSON.parse(this.textContent), undefined, call_getIxDbRecords )"> ', cursorObjString, ' </span> </td>', 
                                      '<td>', '<a href="javascript:void(0)" onclick="ixDbEz.delete(\'demoOS\',\'' , cursor.key ,'\', call_getIxDbRecords )">[Delete]</a>','</td>',
                                    '</tr>');
                          cursor.continue();
                    }
                    else
                    {
                        clientRecords.innerHTML = '<table id="clientDb-Results"><tr border="1"><th>Key</th><th>Value (JSON)</th><th></th></tr>' + 
                                                    html.join('') +
                                                  '</table>';
                        clientRecords.style.visibility = "visible";
                    }
                }

                ixDbCursorReq.onerror = function (e) {
                    //Do some error handling stuff here, if you want to...
                }
            }
            return false;
        }
	
	
	