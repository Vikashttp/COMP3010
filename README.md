# COMP3010 – BOTSv3 Incident Analysis

## Introduction

This report documents a security incident investigation conducted using the Boss of the SOC v3 (BOTSv3) dataset within Splunk. BOTSv3 simulates realistic enterprise log data across network, endpoint, and cloud environments, allowing analysts to practise real-world Security Operations Centre (SOC) workflows.

The purpose of this investigation is to analyse selected security events, identify indicators of suspicious or malicious activity, and demonstrate how a SOC analyst would approach detection, investigation, and response. The scope of this report focuses on one guided question set from the BOTSv3 dataset, supported by Splunk search queries, screenshots, and analysis.

## SOC Roles and Incident Handling Reflection
In a real-world Security Operations Centre (SOC), incident handling is typically organised across multiple tiers, each with distinct responsibilities. Tier 1 analysts are responsible for continuous monitoring and initial triage of alerts, identifying potential security incidents based on log data and predefined detection rules. Tier 2 analysts perform deeper investigation, correlating events across multiple data sources to confirm malicious activity and determine impact.

The BOTSv3 dataset reflects these SOC workflows by providing diverse log sources that require investigation, correlation, and contextual understanding. This investigation aligns primarily with Tier 1 and Tier 2 SOC responsibilities, focusing on detection, analysis, and initial response rather than long-term remediation or threat hunting.

## Installation and Data Preparation

## Guided Investigation and Findings
### Question 1  
**Which IAM users accessed an AWS service in Frothly’s AWS environment?**

**Method**  
AWS CloudTrail logs were analysed using the `aws:cloudtrail` sourcetype, which records AWS API activity. IAM user activity was identified by inspecting the `userIdentity.userName` field, which records the username associated with each API request. Both successful and unsuccessful requests were included.

**Evidence**  
The CloudTrail logs show multiple unique values in the `userIdentity.userName` field, indicating the IAM users that accessed AWS services within the environment.

**Answer**  
bstoll,btun,jstokes,mramirez



### Question 2
**What field would you use to alert that AWS API activity has occurred without MFA?**

**Method**  
AWS CloudTrail logs were analysed using the `aws:cloudtrail` sourcetype to identify AWS API activity performed without multi-factor authentication. Console login events were excluded, and MFA-related fields were inspected.

**Evidence**  
The screenshot shows the field `userIdentity.sessionContext.attributes.mfaAuthenticated`, which indicates whether MFA was used during the API request.

**Answer**  
`userIdentity.sessionContext.attributes.mfaAuthenticated`


### Question 3
**What is the processor number used on the web servers?**

**Method**  
Hardware inventory logs were analysed using the `hardware` sourcetype in Splunk, which contains system-level information such as CPU, memory, and disk details. As the question focuses on web servers, only hardware records associated with web server hosts were examined.

**Evidence**  
The CPU information is recorded in the `CPU_TYPE` field.

Following the extraction method demonstrated in the question example (e.g. *Intel Core i7-8650U → i7-8650U*), the vendor name and clock speed were removed to extract the processor number.
See screenshots/q3_cpu_type.png

**Answer**  
**E5-2676**



### Question 4
**Bud accidentally makes an S3 bucket publicly accessible. What is the event ID of the API call that enabled public access?**

**Method**  
AWS CloudTrail logs were analysed using the `aws:cloudtrail` sourcetype, which records AWS API activity. To identify changes to S3 bucket permissions, the logs were filtered for the `PutBucketAcl` API call, as this operation is used to modify S3 bucket access control lists, including making a bucket publicly accessible.

**Evidence**  
The search returned two `PutBucketAcl` events associated with the same user. As both events relate to ACL configuration, the event corresponding to the API call that applied the access control change was selected as the action that enabled public access to the bucket. The event ID extracted from this API call is shown below:
See screenshots/q4_putbucketacl_event.png

This event represents the API call responsible for the S3 bucket misconfiguration.

**Answer**  
**ab45689d-69cd-41e7-8705-5350402cf7ac**

<img width="1920" height="920" alt="q4_putbucketacl_event" src="https://github.com/user-attachments/assets/9bafa5b5-fd52-4887-9343-624e53703417" />


<img width="1920" height="920" alt="q4_putbucketacl_event2" src="https://github.com/user-attachments/assets/86e2fff8-1944-408a-aa1c-4c926444082f" />

### Question 5  
**What is the IAM username associated with Bud?**

**Method**  
The AWS CloudTrail event identified in Question 4 was examined in further detail. CloudTrail records the identity responsible for API actions within the `userIdentity` object.

**Evidence**  
Within the `PutBucketAcl` event, the IAM username associated with the API call is recorded in the `userIdentity.userName` field. As shown in the screenshot, this field contains the value:

See screenshots/q5_username.png

This username corresponds to Bud.

**Answer**  
**bstoll**

<img width="1920" height="920" alt="q5_username" src="https://github.com/user-attachments/assets/a096f5b2-c7f6-44b4-ab0e-f6ef0ef1adad" />


### Question 6  
**What is the name of the S3 bucket that was made publicly accessible?**

**Method**  
The AWS CloudTrail `PutBucketAcl` event identified in Question 4 was examined to determine which S3 bucket was affected by the access control change. CloudTrail records request-specific details within the `requestParameters` object.

**Evidence**  
Within the `PutBucketAcl` event, the name of the affected S3 bucket is recorded in the `requestParameters.bucketName` field. As shown in the screenshot, this field contains the value:
 See screenshots/q6_bucket_name.png
 
This confirms the name of the bucket that was made publicly accessible.

**Answer**  
**frothlywebcode**



### Question 7
**What is the name of the text file that was successfully uploaded to the publicly accessible S3 bucket?**

**Method**  
S3 access logs were analysed using the `aws:s3:accesslogs` sourcetype to identify activity performed after the bucket became publicly accessible. The logs were filtered for successful `PUT` operations with an HTTP status code of `200`, indicating successful uploads.

**Evidence**  
The search returned multiple successful `PUT` requests; however, only one of the uploaded objects was a plain text file. By examining the object names in the access logs, the following text file was identified as having been successfully uploaded:

This confirms the file that was uploaded to the publicly accessible bucket.

**Answer**  
**OPEN_BUCKET_PLEASE_FIX.txt**


<img width="1920" height="920" alt="q7_s3_put_textfile" src="https://github.com/user-attachments/assets/06bf9022-1dfe-4b6f-966c-d371c95db7fb" />



### Question 8
**What is the FQDN of the endpoint that is running a different Windows operating system edition than the others?**

**Method**  
Windows endpoint monitoring logs were analysed using the `winhostmon` sourcetype, which contains operating system information for endpoints. To establish a baseline, operating system data was aggregated across all hosts to identify any endpoint running a different Windows edition. A keyword-based search was then used to further inspect the identified endpoint.

**Evidence**  
By comparing operating system information across hosts, one endpoint was observed to be running a different Windows operating system edition than the others. A keyword search for this endpoint confirmed its fully qualified domain name (FQDN) as:


<img width="1920" height="920" alt="q8_winhostmon_os3" src="https://github.com/user-attachments/assets/53583bd2-8f01-4933-a6fb-8ba3fff1a553" /><img width="1920" height="920" alt="q8_winhostmon_os5" src="https://github.com/user-attachments/assets/5b71c5ee-5be8-4aac-a405-be9d5304d848" />

This identifies the endpoint running the different Windows edition.

**Answer**  
**bstoll-l.froth.ly**





## Conclusion and Lessons Learned
This investigation demonstrates how cloud misconfigurations and endpoint anomalies can be identified through effective log analysis. The findings highlight the importance of access controls, monitoring, and baseline analysis within a Security Operations Centre.

## References
- AWS CloudTrail Documentation
- AWS S3 Access Control List Documentation
- Splunk Documentation
