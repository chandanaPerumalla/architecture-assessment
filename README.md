# Architecture Document
## HighLevel Architecture
![Open Architecture_Diagram.excalidraw in excalidraw.com](https://raw.githubusercontent.com/chandanaPerumalla/architecture-assessment/main/architecture-diagram.png)

## Components
* **Kafka**: To host the streams Stream_A and Stream_B
* **Hadoop Cluster**: To run Spark applications and Hive queries
* **Notification Service**: Used send event notifications based on signal to the subscribers.
* **Object Storage**: To store data from both the streams, for example, S3. This also helps decouple compute and storage.
* **Hive**: Imposes schema on files written to Object Storage. Hive tables can be queried for analytic workloads.
* **NoSQL Store**: Has copy of data from streams to make look ups easier with appropriate indexing.
* **RDBMS**: Has summary tables with timely aggregates. For example, mysql.
* **Visualization**: Used to visualize analytics query results. For example, tableau.

## Architecture Flow
### Real time/near real time Notifications
1. The Spark consumers running on the Hadoop Cluster consume the events from both streams in parallel.
2. The Spark consumers look for the signal in each of the events. In case of a signal, the Spark consumers post the notifications to a Notification Service.
3. The Notification service then pushes the notifications to the subscribers.

### Enable rolling aggregates
1. The Spark consumers running on the Hadoop Cluster consume the events from both streams in parallel.
2. The Spark consumers then write the micro batch of events for both streams to respective locations on an Object Storage into hourly partitions. The files written are in Parquet format, for efficient querying.
3. A Spark Application running on the Hadoop Cluster read the data for Stream B for the latest hour and compute aggregates every *N* min.
4. The Spark Application can also compute aggregates based on conditions on Stream B.
5. The Spark Application then upserts the aggregates for the current hour to an RDBMS table(s).
6. These RDBMS table(s) can be queried for rolling aggregates.

### Enable lookups
1. The Spark consumers running on the Hadoop Cluster consume the events from both streams in parallel.
2. The Spark consumers then write the micro batch of events to a NoSQL store.
3. The table for Stream A is indexed by `unique_identifier`. The table for Stream B is indexed by `stream_A_unique_identifier`.
4. Enabling indexes make lookups easier.

### Support Analytics
1. The Spark consumers running on the Hadoop Cluster consume the events from both streams in parallel.
2. The Spark consumers then write the micro batch of events for both streams to respective locations on an Object Storage into hourly partitions. The files written are in Parquet format, for efficient querying.
3. Tables for both streams are created on top of stream data locations on the Object Storage.
4. Query Hive for analytic workloads as the processing is done in parallel on the Hadoop Cluster.

## Edit Architecture Diagram
* Open excalidraw.com
* Open the file Architecture_Diagram.excalidraw
