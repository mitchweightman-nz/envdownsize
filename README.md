# envdownsize

Contents
1.	Document Overview	4
1.1.	Document Summary	4
2.	Overview	5
2.1.	Introduction	5
2.2.	Why is this an issue?	5
2.3.	How the CS Large Folder Move Utility gets around the issue	5
2.4.	IMPORTANT	5
3.	Installation and Configuration	6
3.1.	CS Large Folder Move Utility	6
3.2.	Temporary Folder Parent	6
3.3.	LiveReport	6
3.4.	Configuration	7
3.5.	Edit CSLargeFolderMove.exe.config	7
3.6.	Login Configuration Page	7
4.	User Guide	10
4.1.	Moving Folders	10
4.2.	The Buttons	12
4.3.	Log Files	13
4.4.	Log File Samples	13
5.	How It Works	15
6.	Manual Recovery	16


1.	Document Overview
1.1.	Document Summary
Item	Brief Details
Document Title:	Environment Downsizing Utility - Administration and User Guide
Date:	Nopvember 2019
Prepared by:	Rob Vanderpoel
Reviewed by:	
Prepared for:	Techtonics
Document Status:	Draft
2.	Overview
2.1.	Introduction
Considerable resources and, therefore, costs are involved in maintaining one or more non-production Content Server environments as full copies of a production environments, particularly with respect to document storage costs.
The Environment Downsizer provides a mechanism for an across-the-board reduction in environmental resources in a manner that maintains a representative cross-section of content across all aspects of the business. It does this by using LiveReports to select content to be removed based on the last digit of the item’s node id which results in a random selection of 10% of the content to be retained.
2.2.	Architecture
The Environment Downsizer has been designed to be flexible in terms of the criteria for removing or keeping documents. This is achieved by using LiveReports as the data sources to identify the criteria for removing documents and requiring folder dataid to be passed so that specific areas of the folder structure can be targeted and others left untouched.
The Downsizer can be executed in two modes, interactive or command line. The interactive mode is really only useful to test the Live Reports as, to achieve the desired result would require a very large single delete execution which would eventually time out.
The command line approach is designed to be executed repeatedly via the Windows Task Scheduler to repeatedly delete documents in smaller batches.
When undertaking an environment downsizing exercise it is likely you will want to delete millions of documents. Clearly, to do this is in a single execution will lead to extremely long time frames prone to crashing and instability. The recommend approach is to include a Record Limit and execute the Downsizer repeatedly via the Windows Task Scheduler
2.3.	Parameters
 Parameters common to both modes:
	User Id
	Password
	Folder Id of the folder to be targeted
Additional parameter for Interactive mode:
	Max DOP – maximum number of threads to be executed.
Configuration setting for interactive mode (in DocManage.exe.config)
<appSettings>
<add key="ReportForBulkDelete" value="457459" />
 </appSetting	
Where ReportForBulkDelete identifies the LiveReport with the selection criteria
Additional parameter for command line mode
	LiveReport Id that identifies the LiveReport with the selection criteria
Note there is currently no MaxDOP parameter for command line mode so this is fixed at 50.

2.4.	Live Report Example

This example deletes 500 documents at a time, selecting all documents whose last digit is not 0 and selecting the largest documents (based on total storage of all versions) first. 
select top 500 dta.dataid
from dtreecore dtc 
left join dtreeancestors dta on dtc.dataid = dta.dataid
  left outer join dversdata dv on dtc.dataid = dv.docid and dv.vertype is null
where
dta.ancestorid = %1
and dta.dataid-dta.dataid/10*10 not in (0)
and subtype in (144,749)
and deleted = 0
  group by dta.dataid
  order by sum(dv.datasize)/1024/1024 desc

Variation to exclude PDFs as the customer wanted these kept in certain folders for Content Crawler testing:
Select 500 dta.dataid
from dtreecore dtc 
left join dtreeancestors dta on dtc.dataid = dta.dataid
left outer join dversdata dv on dtc.dataid = dv.docid and dtc.versionnum = dv.Version and dv.vertype is null
where
dta.ancestorid = %1
and dta.dataid-dta.dataid/10*10 not in (0)
and subtype in (144,749)
and deleted = 0
and dv.mimetype not like '%pdf%'
  group by dta.dataid
  order by sum(dv.datasize)/1024/1024 desc

Other variations could be to select personal workspaces except those for selected users, or to select all documents with storage greater than 10 MB regardless of last digit. 

3.	Interactive Mode
3.1.	CS Large Folder Move Utility

The Downsizer will automatically use Interactive mode unless it identifies that there are four command line parameters.
 
4.	Command Line Mode and Task Scheduler

The Downsizer will automatically use command line mode if it identifies that there are exactly four command line parameters.
With the MaxDOP value fixed at 50 it is likely you could create several scheduled tasks targeting different folders.
Command line parameters must be provided in the following order:
	User Id
	Password
	Folder Id
	LiveReport Id
4.1.	Setting up Task Scheduler


Set the user account to NT AUTHORITY\SYSTEM so that the tasks can be run in the background without requiring a user session to be active.
 

 

 
Note that you could set specific durations if you can calculate how long is required to execute the required number of executions to delete all the documents meeting the selection criteria.
The task repeat interval can be shortened. Suggest about 1 minute more than expected task duration. 
 

Command line parameters where 219976 is the folder id being targeted and 457459 is the LiveReport id
 


 

 
You task is now scheduled.
You create additional tasks targeting different folders with, if appropriate, different LiveReports with different selection criteria.


 

Double-click on task to edit it.
 

