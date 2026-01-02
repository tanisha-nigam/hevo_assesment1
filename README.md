Hevo Assessment – PostgreSQL to Snowflake Pipeline
This repository contains my complete solution for Hevo Exercise – Assessment I (CXE).
The goal of this exercise was to set up an end-to-end data pipeline using Hevo Data, ingest data from a local PostgreSQL database, apply transformations, and load the processed data into Snowflake.
Tech Stack Used
•	PostgreSQL 14 – Local source database
•	Snowflake (Free Trial) – Destination data warehouse
•	Hevo Data (Trial via Snowflake Partner Connect) – Data ingestion & transformation
•	ngrok – To securely expose local PostgreSQL to Hevo
•	pgAdmin 4 – PostgreSQL management
•	GitHub – Version control & documentation
Step-by-Step Reproduction Guide
Step 1: Snowflake Setup
1.	Signed up for a Snowflake free trial.
2.	Created a virtual warehouse (COMPUTE_WH).
3.	Verified access to Snowflake Worksheets.
Snowflake is used as the final destination for all transformed data.
Step 2: Hevo Setup via Snowflake Partner Connect
1.	Logged into Snowflake.
2.	Navigated to Partner Connect.
3.	Selected Hevo Data and created a free trial account.
4.	Verified Hevo account activation and region (Asia – Singapore).
This ensures seamless integration between Snowflake and Hevo.
Step 3: PostgreSQL Setup (Local)
•	Installed PostgreSQL 14 locally.
•	Used pgAdmin 4 for database management.
•	Created a database named hevo_db.
PostgreSQL acts as the source system.
Step 4: Schema Creation & Data Loading
Created the following tables in PostgreSQL as per the provided schema:
customers
•	id (PK)
•	first_name
•	last_name
•	email
•	address (JSON)
orders
•	id (PK)
•	customer_id (FK → customers.id)
•	status (varchar)
feedback
•	id (PK)
•	order_id (FK → orders.id)
•	feedback_comment
•	rating
Loaded data into these tables using the provided CSV files.
Step 5: Connecting PostgreSQL to Hevo
Since PostgreSQL was running locally, it was not directly accessible from Hevo’s cloud environment. To solve this:
Networking Solution Used
•	ngrok TCP tunnel
Steps:
1.	Enabled remote connections in PostgreSQL:
o	listen_addresses = '*'
o	Added rule in pg_hba.conf: host all all 0.0.0.0/0 md5
2.	Restarted PostgreSQL service.
3.	Started ngrok: ngrok tcp 5432.
4.	Used the generated ngrok post and host in Hevo PostgreSQL source configuration.
This allowed Hevo to connect securely to the local database.
Step 6: Hevo Pipeline Configuration
•	Source: PostgreSQL
•	Destination: Snowflake
•	Ingestion Mode: Logical Replication
Logical replication was chosen to ensure:
•	Incremental data ingestion
•	Change data capture (CDC)
•	Near real-time updates
Transformations Applied (Step 6)
1. Orders → Event Table
From the orders table, an event-based table was generated in Snowflake.
Mapping logic:
•	placed → order_placed
•	shipped → order_shipped
•	delivered → order_delivered
•	cancelled → order_cancelled
Each status change generates a separate event row.
2. Customers → Username Derivation
A derived column username was added to customer data.
Logic:
•	Extracted text before @ from email.
•	Example:
•	jane.doe@gmail.com → jane.doe
This transformation was implemented inside Hevo’s transformation layer.
Step 7: Loading & Validation in Snowflake
Hevo automatically loaded the transformed data into Snowflake under:
PC_HEVODATA_DB.PUBLIC
Validation queries were executed to confirm:
•	Row counts match expectations
•	Events generated correctly
•	Usernames derived correctly
•	No data loss during ingestion
Issues Faced & Workarounds
1. Hevo Could Not Connect to PostgreSQL
•	Cause: Local DB not internet-accessible.
•	Fix: Used ngrok TCP tunneling.
2. Data Type Errors (e.g., P01 numeric error)
•	Cause: Implicit casting of alphanumeric IDs.
•	Fix: Ensured all IDs treated as strings.
3. Country Mapping Returned Unknown
•	Cause: Inconsistent formatting in source data.
•	Fix: Explicit standardization logic using CASE statements.
All issues were resolved without compromising data integrity.
What I Learned
Through this assignment, I gained hands-on experience in building an end-to-end data pipeline using PostgreSQL, Hevo, and Snowflake. I learned how to configure logical replication, expose local databases securely, and handle real-world data quality issues such as duplicates, null values, inconsistent country codes, and mixed currencies. The exercise reinforced the importance of keeping raw data immutable, applying transformations downstream, and validating data at every stage. Overall, it improved my understanding of reliable data ingestion, transformation design, and production-grade data workflows.



