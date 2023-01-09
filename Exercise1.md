## **Lab 1: Getting started with Cosmos DB for PostgreSQL**

For a successful connection into Cosmos DB for PostgreSQL, we will be adding firewall rules to allow access to our Cosmos DB for PostgreSQL database.

### Task 1: Configure server-level firewall service

Follow the instructions given below to allow yourself access to the Cosmos DB for PostgreSQL cluster.
 
1.In the **Azure Portal** go to **Home**, then locate **Azure services** and select **Cosmos DB for PostgreSQL** on homepage.

![](images/azpostgresql.png)


2.Click on your Cosmos DB for PostgreSQL Database **postgreXXXXX**.

![](images/azpostgresql1.png)

3.We pre-provisioned a basic production grade Cosmos DB for PostgreSQL cluster with 1 coordinator with 4 vCores and 16GB RAM and 2 workers with each 4 vCores and 32GB RAM for this lab. You can review the cluster details in Azure Portal. 

![](images/azpostgresqlclusterinfo.png)

4.On the overview blade under **Security** click **Networking** and **select** the **"Allow public access from Azure services and resources within Azure to this server group"**.

![](https://github.com/Shivashant25/Building-Real-time-applications-using-Hyperscale-Citus/blob/master/images/l2%20s3.png?raw=true)

5.Now add **Firewall Rule**. Select **Add current client IP address**, this will add the client IP by creating a new rule. Then **save** the changes.

![](https://github.com/Shivashant25/Building-Real-time-applications-using-Hyperscale-Citus/blob/master/images/l2%20s4.png?raw=true)

6.Click **Next** on the bottom right of this page.
