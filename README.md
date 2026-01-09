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
Initial exploration was performed to confirm that the BOTSv3 dataset was successfully ingested into Splunk and to identify available sourcetypes.
Evidence: See screenshots/q1_index_overview.png


### Question 2
AWS CloudTrail logs were analysed to identify AWS API activity performed without multi-factor authentication (MFA). The field used was userIdentity.sessionContext.attributes.mfaAuthenticated.
Evidence: See screenshots/q2_mfa_field.png


### Question 3
**Method**  
Hardware inventory logs were analysed using the `hardware` sourcetype in Splunk, which contains system-level information such as CPU, memory, and disk details. As the question focuses on web servers, only hardware records associated with web server hosts were examined.

**Evidence**  
The CPU information is recorded in the `CPU_TYPE` field.

Following the extraction method demonstrated in the question example (e.g. *Intel Core i7-8650U → i7-8650U*), the vendor name and clock speed were removed to extract the processor number.

**Answer**  
**E5-2676**



Evidence: See screenshots/q3_cpu_type.png


### Question 4
**Bud accidentally makes an S3 bucket publicly accessible. What is the event ID of the API call that enabled public access?**

**Method**  
AWS CloudTrail logs were analysed using the `aws:cloudtrail` sourcetype, which records AWS API activity. To identify changes to S3 bucket permissions, the logs were filtered for the `PutBucketAcl` API call, as this operation is used to modify S3 bucket access control lists, including making a bucket publicly accessible.

**Evidence**  
The search returned two `PutBucketAcl` events associated with the same user. As both events relate to ACL configuration, the event corresponding to the API call that applied the access control change was selected as the action that enabled public access to the bucket. The event ID extracted from this API call is shown below:
See screenshots/q4_putbucketacl_event.png
<img width="1920" height="920" alt="q4_putbucketacl_event" src="https://github.com/user-attachments/assets/9bafa5b5-fd52-4887-9343-624e53703417" />


<img width="1920" height="920" alt="q4_putbucketacl_event2" src="https://github.com/user-attachments/assets/86e2fff8-1944-408a-aa1c-4c926444082f" />

### Question 5
From the same CloudTrail event, the IAM username responsible for the action was identified as bstoll.
Evidence: See screenshots/q5_username.png
<img width="1920" height="920" alt="q5_username" src="https://github.com/user-attachments/assets/a096f5b2-c7f6-44b4-ab0e-f6ef0ef1adad" />



### Question 6
The name of the S3 bucket that was made publicly accessible was frothlywebcode.
Evidence: See screenshots/q6_bucket_name.png


### Question 7
S3 access logs were analysed to identify files uploaded after the bucket became publicly accessible. The uploaded text file was OPEN_BUCKET_PLEASE_FIX.txt.
Evidence: See screenshots/q7_s3_put_textfile.png
<img width="1920" height="920" alt="q7_s3_put_textfile" src="https://github.com/user-attachments/assets/06bf9022-1dfe-4b6f-966c-d371c95db7fb" />



### Question 8
Windows endpoint monitoring logs were analysed to compare operating system editions across hosts. One endpoint was identified as running a different Windows edition. The FQDN of this endpoint is bstoll-l.froth.ly.
Evidence: See screenshots/q8_winhostmon_os.png
<img width="1920" height="920" alt="q8_winhostmon_os3" src="https://github.com/user-attachments/assets/53583bd2-8f01-4933-a6fb-8ba3fff1a553" /><img width="1920" height="920" alt="q8_winhostmon_os5" src="https://github.com/user-attachments/assets/5b71c5ee-5be8-4aac-a405-be9d5304d848" />





## Conclusion and Lessons Learned
This investigation demonstrates how cloud misconfigurations and endpoint anomalies can be identified through effective log analysis. The findings highlight the importance of access controls, monitoring, and baseline analysis within a Security Operations Centre.

## References
- AWS CloudTrail Documentation
- AWS S3 Access Control List Documentation
- Splunk Documentation
