Information if you wish to run a ksqlDB application against data generated by the USERS and PAGEVIEWS quick start by the managed kafka-connect-datagen
 
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