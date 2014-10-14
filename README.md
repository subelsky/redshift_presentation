    __  .__   __. .___________..______        ______
    |  | |  \ |  | |           ||   _  \      /  __  \
    |  | |   \|  | `---|  |----`|  |_)  |    |  |  |  |
    |  | |  . `  |     |  |     |      /     |  |  |  |
    |  | |  |\   |     |  |     |  |\  \----.|  `--'  |
    |__| |__| \__|     |__|     | _| `._____| \______/

    .___________.  ______
    |           | /  __  \
    `---|  |----`|  |  |  |
        |  |     |  |  |  |
        |  |     |  `--'  |
        |__|      \______/

    .______       _______  _______       _______. __    __   __   _______ .___________.
    |   _  \     |   ____||       \     /       ||  |  |  | |  | |   ____||           |
    |  |_)  |    |  |__   |  .--.  |   |   (----`|  |__|  | |  | |  |__   `---|  |----`
    |      /     |   __|  |  |  |  |    \   \    |   __   | |  | |   __|      |  |
    |  |\  \----.|  |____ |  '--'  |.----)   |   |  |  |  | |  | |  |         |  |
    | _| `._____||_______||_______/ |_______/    |__|  |__| |__| |__|         |__|

## bmoreonrails.org October 2014

by Mike Subelsky
http://subelsky.com
@subelsky

---
# What was our problem?

* Large volume of diverse relational data
* Need for responsive ad hoc & canned reporting
* Many joins
* Many disparate tables/schemas
* Small team / no DBAs
* Heavy writes once a day, heavy reads rest of day
* Wanted NoSQL speed/flexibility but needed SQL expressiveness
* Already using Postgres for main database

---
# What is Redshift?

* Data warehouse service
* RDBMS-lite; optimized for analytics
* Runs on optimized hardware w/ direct-attached storage
* Column-oriented
* Highly parallel, easy to add nodes*
* Compression
* Near-web latencies
* Can use Postgres clients (pg 8.3)
* Cheap to get started, not cheap to grow

---
# Sample TSV file

---
# Sample manifest file

---
# COPY from manifest file to Redshift

---
# Loader worker (simplified)

---
# StaqETL (simplified)

---
# Why use Redshift?

---
# If you truly have big/complex data and want to analyze it with SQL

* Has relations
* Big enough to benefit from NoSQL advantages

---
# If you want cloud flexibility or AWS interoperability

* Resizing on the fly
* Copy to/from S3 and other AWS products like DynamoDB

---
# If you already are a Postgres shop
* can use Postgres 8.3 DB drivers
* can use Postgres to simulate Redshift locally

## StaqRedshift code

---
# Disadvantages / Considerations

* Latency is not super fast, may need to cache/proxy
  * Short-duration queries can take several seconds to spin up
* High performance requires careful tuning
  * Distribution keys
  * Sort keys
* Constraints are only used for hinting the query planner
* Leader node is a bottleneck

---
# Disadvantages / Considerations

* No UPSERT
* Not cheap to upgrade, big cost jump to larger nodes
* Loading kind of slow for large numbers of small files
* May want to consider other competitors like Snowflake, Splout, AtomicDB
* TSV/CSV/JSON not great for typed data

---
# Cost

* Amazon claims you can get it down to cost $999/TB/year
  * (reserved for 3 years)

---
# Questions?
