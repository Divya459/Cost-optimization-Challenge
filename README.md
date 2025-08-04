# Cost-optimization-Challenge
Managing Billing records in Azure Serverless Architecture

**Query/Scenario:**

we have a serverless architecture in azure where one of tour services stores billing records ina zure cosmos db. the system is read heavy, but records older than three months are rarely accessed. Over the past few years the data base size hads signiicantly grown, eading to increased costs. we need an efficient way to reduce costs while maintaining the sata avaialbility 

**Considering the below Requirements:**
1. simplicity & ease of implemetation -th esolution should be straitforward to deploy and maintain
2. no data loss & no downtime the transaction should be seamless, with out losing any recods or requiring 2 service downtime.
3. no changes to API contracts- the exisitng read/write api for billing records must remain unchabged
   
**also** 

1. record size : each billing record can be large as 300 kb.
2. total records: the data base curently holds over 2million records 
3. access latency :When an old record is required, it should still be served, with a response time in th eorder if seconds

**Solutions can be considered:**

Tiered Storage with Seamless Access

🔹 Active Tier (Cosmos DB)
Stores recent billing records (≤ 3 months)
Continues to serve read/write APIs without any change

🔹 Archive Tier (**Azure Blob Storage or Data Lake**)
Stores older records (> 3 months) as JSON files
Accessed only when needed, with latency in seconds

🔄 Data Migration Strategy

 Below strategy can be used for automating the process of storing the records older than 3 months in Blob
 
✅ **_Azure Data Factory_** (**Preferred for Simplicity**)

**Scheduled pipeline** runs weekly

**Filters records older than 3 months**

Copies data to **Blob Storage**

Optionally deletes or flags records in Cosmos DB


**Azure Data Factory** – **Step-by-Step Pipeline**

🎯 Goal:

Copy records older than 3 months from Cosmos DB to Azure Blob Storage.


🛠️ Steps:

**1. Create Linked Services**

Cosmos DB: Use your account key or connection string.
Azure Blob Storage: Use your storage account credentials.

**2. Create Datasets**

Source Dataset: Cosmos DB collection (Billing Records)
Sink Dataset: Blob Storage (JSON or CSV format)

**3. Create a Pipeline**

Add a Lookup Activity (optional):  To fetch the current date or parameters.

Add a Filter Activity:

Use a query like:

SELECT * FROM c WHERE c.timestamp < GetCurrentDate() - 90

**Add a Copy Data Activity:**

Source: Cosmos DB dataset with the above query
Sink: Blob Storage dataset
Configure file naming with dynamic content (e.g., archive_@{utcnow()}.json)

**4. Add a Trigger**

Schedule the pipeline to run weekly or monthly.

**5. Monitor**
Use the Monitor tab to track pipeline runs and troubleshoot failures.

✅ **_Azure Function_** (**Optional for Custom Logic**)
Can be used for **real-time or event-driven** migration
Handles **edge cases or custom transformations**

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/bee3f24f-76a6-4502-9442-c164274b20e9" />


🔍 Seamless Access Strategy
To maintain API contracts:
No changes to existing read/write APIs

Introduce a middleware or service layer that:
First queries Cosmos DB
If not found, queries Blob Storage
Returns the result transparently
This ensures no downtime, no data loss, and no disruption to consumers.

<img width="900" height="400" alt="image" src="https://github.com/user-attachments/assets/7898bf33-f3d9-490d-8933-24e046df344f" />
