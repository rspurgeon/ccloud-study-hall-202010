# ccloud-study-hall-202010

The tutorial we covered today can be found here:

  https://kafka-tutorials.confluent.io/kafka-connect-datagen-ccloud/kafka.html

The `ccloud-stack` library is found in the `examples` repository, here: 

  https://github.com/confluentinc/examples/tree/6.0.0-post/ccloud/ccloud-stack 

For a list of commands and security configurations created by `ccloud-stack`, see this:

  https://docs.confluent.io/current/tutorials/examples/ccloud/docs/ccloud-stack.html

Helpful document for ksqlDB ACLs for a topic:

  https://docs.confluent.io/current/cloud/cp-component/ksql-cloud-config.html#create-acls-for-ksql-to-access-a-specific-topic-in-ccloud

Various commands:

	* Alternate for wget: `curl https://raw.githubusercontent.com/confluentinc/examples/latest/utils/ccloud_library.sh --output ccloud_library.sh`

  * ACLs are required to give the ksqlDB app access to our topics
  	* first get the ksqlDB service account id
      ```
      ccloud service-account list 
      ```

		* Then assign ACLs
      ```
      ➜ ccloud kafka acl create --allow --service-account 128188  --operation READ --operation WRITE --topic 'mytopic'
        ServiceAccountId | Permission | Operation | Resource |  Name   |  Type
      +------------------+------------+-----------+----------+---------+---------+
        User:128188      | ALLOW      | READ      | TOPIC    | mytopic | LITERAL
        User:128188      | ALLOW      | WRITE     | TOPIC    | mytopic | LITERAL
      
      ➜ ccloud kafka acl create --allow --service-account 128188  --operation READ --operation WRITE --topic 'users'
        ServiceAccountId | Permission | Operation | Resource | Name  |  Type
      +------------------+------------+-----------+----------+-------+---------+
        User:128188      | ALLOW      | READ      | TOPIC    | users | LITERAL
        User:128188      | ALLOW      | WRITE     | TOPIC    | users | LITERAL
      ``` 

		* sample ksqlDB app, taken from ksqlDB docs
      ```
      CREATE STREAM pageviews (
          viewtime BIGINT,
          userid VARCHAR,
          pageid VARCHAR
        ) WITH (
          KAFKA_TOPIC='mytopic',
          VALUE_FORMAT='JSON'
        );
      CREATE TABLE users (
            userid VARCHAR PRIMARY KEY,
            registertime BIGINT,
            gender VARCHAR,
            regionid VARCHAR
          ) WITH (
            KAFKA_TOPIC = 'users',
            VALUE_FORMAT='JSON'
        );
      CREATE STREAM pageviews_enriched AS
          SELECT 
             users.userid AS userid, 
             pageid, 
             regionid, 
             gender 
          FROM pageviews
            LEFT JOIN users ON pageviews.userid = users.userid
          EMIT CHANGES;
      ```

# Links
 * [Confluent Cloud](https://www.confluent.io/confluent-cloud/)
 * [Kafka Tutorials](https://kafka-tutorials.confluent.io/)
 * [Confluent Examples](https://github.com/confluentinc/examples)
 * [Confluent cp-demo](https://github.com/confluentinc/cp-demo)
 * [developer.confluent.io](https://developer.confluent.io/)
 * [kafka-devops](https://github.com/confluentinc/kafka-devops)

