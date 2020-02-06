## **Lab 3: Create Application Tables**

The data we’re dealing with is an immutable stream of log data that we will be inserting directly into Hyperscale (Citus). It’s also common for log data to first be routed through something like Kafka. Kafka has many benefits like allowing you to pre-aggregate the data so high volumes of data are manageable.
On this page we will create a simple schema for ingesting HTTP event data, shard it, create load and then query.
Let's create the tables for http requests, per-minute aggregates and a table that maintains the position of our last rollup.

1.Expand the server group **postgresxxxxx**, then the server and under server expand **Database**. Right click on the database **citus** and select **New Query**.

<kbd>![](images/1query.png)</kbd>

2.In the **New Query** copy and paste the following to create the tables.

```
-- this is run on the coordinator
CREATE TABLE http_request (
site_id INT,
ingest_time TIMESTAMPTZ DEFAULT now(),

url TEXT,
request_country TEXT,
ip_address TEXT,

status_code INT,
response_time_msec INT
) PARTITION BY RANGE (ingest_time);

-- Configure pgpartman to create daily partitions
SELECT partman.create_parent('public.http_request', 'ingest_time', 'native', 'daily');
UPDATE partman.part_config SET infinite_time_partitions = true;

-- Automatically create daily partitions
SELECT partman.run_maintenance(p_analyze := false);

-- Schedule automatic creation of partions on a daily basis
SELECT cron.schedule('@daily', $$SELECT partman.run_maintenance(p_analyze := false)$$);


CREATE TABLE http_request_1min (
site_id INT,
ingest_time TIMESTAMPTZ, -- which minute this row represents
request_country TEXT,

error_count INT,
success_count INT,
request_count INT,
sum_response_time_msec INT,
CHECK (request_count = error_count + success_count),
CHECK (ingest_time = date_trunc('minute', ingest_time)),
PRIMARY KEY (site_id, ingest_time,request_country)
);

CREATE TABLE latest_rollup (
minute timestamptz PRIMARY KEY,

CHECK (minute = date_trunc('minute', minute))
);
```

3.Then click on **Run** to execute the query.

<kbd>![](images/queryrun1.png)</kbd>

4.Once query completes execution, you should see **Commands completed successfully** under **Messages** tab. 

<kbd>![](images/querymsg1.png)</kbd>

5.Verify the tables created by going to the blade on the left of the Azure Data Studio window, and clicking the arrows to open **Databases** > **citus** > **Tables**. Under Tables, you can review the tables. Along with tables you would also see daily partitions for the http_request table. We used pg_partman to create those partitions. pg_partman makes postgres native partitioning very simplified. We also scheduled a cron job to automatically create partitions on a daily basis.

<kbd>![](images/query1table.png)</kbd>

### Shard tables across nodes

A hyperscale (citus) deployment stores table rows on different nodes based on the value of a user-designated column. This "distribution column" marks how data is sharded across nodes. Let's set the distribution column to be site_id, the shard key.

> **Note**: You can replace the query with a new one instead of opening a new query everytime for all other subsequent steps.

6.Now, Right click on the database citus as done in step 1 and select New Query. Then copy and paste the following in the console and then click on **Run** to execute the query to see what you just created. As an alternative, you may also choose to replace the query in existing query window and run from there.

```
SELECT create_distributed_table('http_request', 'site_id'); 
SELECT create_distributed_table('http_request_1min', 'site_id'); 
```

<kbd>![](images/query2.png)</kbd>

The above commands create shards for both the tables across worker nodes. When you create_distributed_table, you take all the data that would have been in a big single table and you split it ('shard it') across multiple smaller tables, that in turn get distributed across the worker nodes.

Notice that both tables are sharded on site_id. Hence there’s a 1-to-1 correspondence between http_request shards and http_request_1min shards i.e shards of both tables holding same set of sites are on same worker nodes. This is called colocation. Colocation makes queries, such as joins, faster and our rollups possible. In the following image you will see an example of colocation where for both tables site_id 1 and 3 are on worker 1 while site_id 2 and 4 are on Worker 2.

<kbd>![](images/colocation.png)

7.Click **Next** on the bottom right of this page.
