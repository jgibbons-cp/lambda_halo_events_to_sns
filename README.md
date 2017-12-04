# Event Connector with Modules for Jira, SNOW, Quarantine and SNS


## Purpose
This lambda application monitors the /v1/events endpoint in the Halo API,
looking for specific events.  If a targeted event is matched, the tool will
move the workload into the configured quarantine group or create a JIRA or ServiceNow ticket or send a messages to 
AWS SNS with event information.

## How it works
Targeted events are listed, one per line, in `/conf/target-events`.  Feel free
to alter the file and re-zip the archive.  Event types produced by Halo can be found here:
https://api-doc.cloudpassage.com/help#event-types

When the end of the events stream is reached, this tool will continue to query
until more events arrive.  If you do not set the `HALO_EVENTS_START`
environment variable, the tool will start at the beginning of the current day.

The quarantine group is defined with the `$QUARANTINE_GROUP_NAME` environment
variable.  If you don't define this environment variable, it is assumed to be
"Quarantine". You should configure the group in your Halo account before you run
this tool.  We recommend applying a firewall policy to the group that restricts
all outbound traffic, and only allows inbound traffic from Ghostports users.

## Prerequisites

* You'll need an account with CloudPassage Halo.
* Make sure that your policies are configured to create events on failure.
* You'll need an auditor API key for your Halo account.
* You'll need a JIRA or ServiceNow account to use those modules
* Create a quarantine group in your Halo account, with the appropriately
restrictive firewall rules if you are using that module.


## Using the tool
        
If working with Jira, configure your JIRA information in `configs/config.yml`  
If working with ServiceNow configure your ServiceNow instance in service_now_test.py by populating servicenow_instance = ""  For example, if your ServiceNow instance is https://devxxxxx.service-now.com then set the variable to "devxxxxx"

If working with SNS, configure an SNS topic and a FIFO SQS (set a message that says None or put a timestamp to start from in ISO 8601 format like 2017-12-04T04:32:40.241Z)and configure a Lambda function from scratch:  

In the Lambda function:  
1) For Runtime choose Python 2.7  
2) Create a custom role and add the JSON in iam_role.json as an inline policy.  
3) Choose the role just created for the function.
4) Click "Create function"  
5) For "Environment Variables" enter the following:  
&nbsp;&nbsp;HALO_API_KEY (auditor)  
&nbsp;&nbsp;HALO_API_SECRET_KEY (auditor)    
&nbsp;&nbsp;SNS_ARN  
&nbsp;&nbsp;SQS_URL  
&nbsp;&nbsp;HALO_EVENTS_START (in ISO 8601 format if you seeded the queue with None above, e.g. 2017-12-04T04:32:40.241Z unless you want to start at the beginning of the current day)  
9) Set "Timeout" in "Basic Settings" to 2 minutes or above  
10) Click "Save"
11) For "Code entry type" choose "Upload a .ZIP file" from the dropdown.  
12) Upload sns_lambda.zip  
13) For "Handler" enter runner.handler    
14) Click "Test", name it and save the default 
15) Clic "Test" to test.  The last_event_timestamp will be stored in the SQS to save state for the next run.   

