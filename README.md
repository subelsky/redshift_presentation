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

by [Mike Subelsky](http://subelsky.com)

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

    staq_account_id	staq_start_at_timestamp	staq_end_at_timestamp	staq_connection_id	staq_key	staq_time_zone	staq_collection_mission_id	staq_collected_at	date	last_modified_date_time	creative_id	creative_type	platform	tag_key	banner
    565	1970-01-01 00:00:00	1970-01-01 23:59:59	1637	36039976369	UTC	5962713	2014-10-09 18:33:31	2014-10-03	2014-10-03 21:58:55	36039976369	ThirdPartyCreative	staq_no_value_found	staq_no_value_found	staq_no_value_found

---
# Sample manifest file

    { "entries": [
        { "url": "s3://example-bucket/example_file.tsv.gz",
          "mandatory": true
        }
      ]
    }
---
# COPY from manifest file to Redshift

    COPY e1_performance_by_day ("staq_account_id", "staq_start_at_timestamp", "staq_end_at_timestamp", "staq_connection_id", "staq_key", "staq_time_zone", "staq_collection_mission_id", "staq_collected_at", "date", "ctr", "impressions")
    FROM 's3://example-bucket/example_file.tsv.gz'
    WITH CREDENTIALS AS 'aws_access_key_id=######;aws_secret_access_key=######'
    NULL AS 'staq_no_value_found'
    DELIMITER AS '	'
    MANIFEST
    ACCEPTINVCHARS
    GZIP
    IGNOREHEADER 1
    STATUPDATE OFF
    COMPUPDATE OFF;

---
# StaqETL (simplified)

    class StaqETL::Loader
      attr_writer :database

      def initialize(high_priority = false)
        @high_priority = high_priority
      end

      def load(load_operations)
        operations = [StaqETL::Operations::ConfigureSlotCount.new(high_priority)]

        load_operations.each do |load_op|
          command = if load_op.table_exists?
                      StaqETL::Operations::Merge.new(load_op)
                    else
                      StaqETL::Operations::FirstCopy.new(load_op)
                    end

          operations += command.operations
        end

        database.execute(operations,high_priority)
      end

      private

      attr_reader :high_priority

      def database
        @database ||= StaqRedshift::Database.new
      end
    end

---
# Why use Redshift?

---
# If you have big/complex data

* Is relational (needs JOIN)
* You don't want to roll your own reporting primitives
* Big enough to benefit from NoSQL advantages
* Don't need full RDBMs guarantees

---
# If you want cloud flexibility or AWS interoperability

* Resizing / restoring on the fly
* Copy to/from S3 and other AWS products like DynamoDB

---
# If you already are a Postgres shop

* can use Postgres 8.3 DB drivers
* can use Postgres to simulate Redshift locally

---
# Disadvantages / Considerations

* High latency, may need to cache/proxy
  * Short-duration queries can take several seconds to spin up

* Requires careful tuning for max benefit
  * Distribution keys
  * Sort keys

* Constraints are only used for hinting the query planner
* Leader node is a bottleneck

---
# Disadvantages / Considerations

* No UPSERT
* Big cost jump to larger nodes
* Loading kind of slow for large numbers of small files
* Consider other competitors
  * Snowflake, Splout, AtomicDB
* TSV/CSV/JSON not great for typed data

---
# Cost

* Amazon claims you can get it down to cost $999/TB/year
  * (reserved for 3 years)

---
# Further reading

* http://docs.aws.amazon.com/redshift/latest/dg/welcome.html
* http://www.slideshare.net/AmazonWebServices/redshift-best-practices-part-1dp2
* http://www.slideshare.net/AmazonWebServices/uses-and-best-practices-for-amazon-redshift

---
# Questions? Want a job?

* mike@staq.com
* http://subelsky.com
