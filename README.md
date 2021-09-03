
DevOps Assessment
===

<!--ts-->
   * [Scenario](#Scenario)
   * [Serverless Solution Design](#Serverless-Solution-Design)
   * [Design Explanation](#Design-Explanation)
   * [CICD Solution](#CICD-Solution)
   * [IaC Tool](#IaC-Tool)
   * [Go-Live Plan](#Go-Live-Plan)
   * [Monitoring Tool](#Monitoring-Tool)
<!--te-->


Scenario
---

- It has been decided to build out a version 2 of successful retail web application to meet the increasing demands of the customers. The current application currently has a mobile app, a monolithic web app which also exposes an API layer used by the mobile app. The database engine used is Microsoft SQL Server. The application is hosted in house on a windows server. For the new design, various modules will be split into microservices. and in addition to MSSQL, a NoSQL db will also be used to store some datasets, as well as redis for caching.
- Development is currently done in-house but the process is slow and tedious, the code is built on the local machine of the developer(s) and deployed manually using FTP. There’s a production environment and all testing is done on the local machine of the respective developer.

- When the new serverless platform is ready there are some tasks that need to be taken care of before the old environment can be switched off. For example, the DNS needs to be switched over to the new environment, and load balancing implemented.
- A go-live plan (runbook) needs to be created to document all steps that will have to be taken in preparation for going live on the new platform with their website.

**Tasks**
- [x] * Diagram a serverless architecture (SaaS) on a cloud platform preferably Azure. Keep in mind that the codebase is C# (.NET framework), elaborate on the diagram.
- [x] * Create a “go live plan” and document the steps that would have to be taken prior to going live on the new environment. Try to be as detailed as possible.
- [x] * Come up with a CI/CD strategy. Elaborate on the chosen tool.
- [x] * With IaC (Infra as Code) in mind, propose a tool that can easily be integrated with your proposed environment.
- [x] * Diagram a serverless architecture with redundancy in mind (multi-region, clustered, etc).

- [x] **Aftercare**: 
What monitoring tool would you use for the new environment and what metrics would you use? Explain why.
***
>You have 48hours to provide a github link to your solution, which will contain your well commented scripts for automation deployment etc. It’s important to submit your assigned by the deadline, regardless of the completeness of your solution. No submissions will be accepted after 48 hours.
May the spirit of Wakanda guide you.




Serverless Solution Design
---
![](https://i.imgur.com/BAPpA5J.png)


Design Explanation 
---


> "...For the new design, ***various modules*** will be split into ***microservices***. and in addition to ***MSSQL***, a ***NoSQL db*** will also be used to store some datasets, as well as ***redis for caching***.." 

From the requirement statement above, I gathered the following:  
- Various Modules (not all) would be split off. Meaning some parts of the Monolithic would be left as is. I used an **Azure App Service** to host this. This is a scalable Azure solution that is recommended for Lift-and-Shift migration of Monolithic web apps. This requires an **App Service Plan** where the compute requirements are configured.
- **Azure Functions** provides serverless compute on Azure. *Logic Apps* does a similar function but doesn't support C# as needed. 
- Azure hosts a robust **SQL service** which can be used. The **Azure CosmosDB** supports NoSQL as requested by the reachitecture.
- **Azure Cache for Redis** is used to persist semi-static data contents so as to reduce load/traffic to the DB. 
-  **Azure Front Door** is used to provide a scalable *multi-regional* routing. It will route requests to the fastest and most available application backend. Front Door has  backend health monitoring functions enabling it to perform automatic failover switches and can withstand failures to an entire *Azure region*.
- **Azure DNS** is a hosting service for DNS domains, providing name resolution.
- **Azure CDN** is used to cache site's static content for lower latency and faster delivery of content. These static contents are stored in **Azure Blob** storage.   
- **Azure Active Directory** provides the needed authentication for users.


CICD Solution
---
1. Before the CICD, a Source Code Management is needed, as exposed by this:

> "...Development is currently done in-house but the process is slow and tedious, the code is built on the local machine of the developer(s) and deployed manually using FTP." 

Azure Repos can be leveraged to solve this as it fit snugly to the Azure ecosystem. Alternatively Git/Github can be used but this solution doesn't tie into the choosen toolset, and would required more configuration to fit better.

2. The testing approach should be addressed. Here's the problem:

> "...There’s a production environment and all testing is done on the local machine of the respective developer."

For a unified/centralized testing, Azure provides Azure Test Plans which is baked into its Azure DevOps offering. This can be utilized to provide a more reliable and robust testing.

3. We can now face the CICD challenge stated here:
> "...The current application currently has a mobile app, a ***monolithic web app*** which also exposes an API layer used by the mobile app. The database engine used is Microsoft SQL Server. The application is ***hosted in house*** on a ***windows server***.."

Azure Pipelines should be use for the CICD. This solution is native to the platform and reduces tooling requirements and unnecessary complexity. However, in implementing it, these should be considered for efficiency:

- When the monolithic web app is migrated from the on-prem host to a Windows App Service Plan, multiple slots should be created. These can be test, staging, and production. The deployable artefact from the Continuous Delivery can be piped to the staging environment where some other vetting process can take place to ensure quality of deployed software.  

IaC Tool
---
Many IaC tools can be used but the reason I recommend **Terraform** is as follows:
- It is platform agnostic.
- It is robust and widely supported
- It can be used to maintain all the hosted infrastructure. This includes creation, deletion,and  modification
- It is Idempotent. Regardless of the number of times it is ran, Terraform would only ensure a platform is set to its target state, nothing more.
- It supports immutable infrastructure. It avoids the inconsistencies that can come from mutable infrastructure solutions like Ansible, such as non-descreet versioning. 
- It is descriptive in nature. You only set a target/desired state without explaining how it should achieve this.
-- Most importantly Terrafrom ties well into the choosen platform and toolset.

Go-Live Plan
---


> When the new serverless platform is ready there are some tasks that need to be taken care of before the old environment can be switched off. For example, the DNS needs to be switched over to the new environment, and load balancing implemented.

> A go-live plan (runbook) needs to be created to document all steps that will have to be taken in preparation for going live on the new platform with their website.


```markdown
1. Replicate configurations:
- Set the IAM policies, networking, firewall rules, and user/admin Accounts. 
- Ensure logging and monitoring is working in new environment 
- Make sure they are configured appropriately.

2. Minimize downtime:
- Test the new system to ensure it can hold traffic.
- Test the applications on the new infrastructure's configurations, ensuring that they have access to their databases, file shares, web servers, load balancers, Active Directory servers, etc.

3. Cautious release:
- Release the new environment to a smaller circle of trusted users in a pilot roll-over event. 
- Operate the two environment in parrallel while gradually switching more traffic to the new environment. 
- Rely heavily on App monitoring solutions and pilot user feedback.


4. Have a rollback plan in place:
- In the event of a catastrophic failure or unpredicted/undesirable behavior of the new event, redirect traffic back to old system. 
- If the go-live takes much longer than expected, initiate the roll-back plan. 

# In all, it is crucial to have a rollback plan in place so you can roll back to your old environment. This flexibility during go-live can help you prevent costly downtimes and breaches of service agreements.



---

If all goes well:
1. Perform a data freeze: Halt changes or developemnt going into the old environment. You might need to perform a final data sync of the last changes. 
2. Avoid Disruption: forward all traffic to the destination servers, ensuring that all users are immediately directed to the new environment. This avoids a potentially lengthy wait for DNS propagation.
3. Perform a DNS propagation to new environment 
4. Perform security checks and vulnerability scans regularly. 
```

Monitoring Tool
---
Azure Monitor should be used to provide insight on the performance of your applications and services. It ties best with the platform and has a lot of features that makes it ideal. 
- Metrics are point-in-time information of a system based on a numerical value
- Metrics in Azure Monitor are capable of supporting near real-time scenarios making them important for alerting and fast detection of issues.
- Metrics such as FunctionExecutionUnits and FunctionExecutionCount for Azure Functions are necessary for you to know the usage and the cost you're expecting. Others like FailedRequests enables you to know what might be failing and to remedy it quickly.
