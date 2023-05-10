# full-cycle-2.0-kafka-connect

Files I produced during the Apache Kafka Connect classes of my [Microservices Full Cycle 3.0 course](https://drive.google.com/file/d/1bJnFxQPKgSsI30sCvW-KzYK4V5JWzgSs/view?usp=share_link).

# Theory

## Kafka Connect basic functioning

With Kafka Connect you can get data from one place, store it on a topic, and from the topic, store that data in another place. For example, get data from MySQL, store on a topic, and then, send that data to Elasticsearch.

![Data Sources, Connectors, Sinks and Kafka](./images/kafka-connect-functioning.png)

## Standalone worker

Standalone workers have connectors inside them and inside the connectors there are tasks running to get data from one source and store it on another.

![](./images/kafka-connect-standalone-worker.png)

## Distributed Workers

With a cluster of distributed workers, you can divide the same task over multiple workers, each one reading from different partitions for example.

![](./images/kafka-connect-distributed-workers.png)

## Converters

Converters help you get data from one source, change it's format and send it to another source.

![MySQL -> ResultSet -> JDBC Connector -> API format -> AvroConverter -> byte array -> Kafka -> byte array -> AvroConverter -> API format -> HDFS Sink Connector -> HDFS](./images/kafka-connect-converters.png)

## Configuring a connector to import data from MySQL and export it to MongoDB

### Inserting data into MySQL

```sh
docker-compose up -d

# When the containers are ready, run:
docker-compose exec mysql bash

# Inside mysql container, run:
mysql -u root -p fullcycle  # Password is root
```

Inside MySQL shell, run:

```sql
create table categories(id int auto_increment primary key, name varchar(255));
show tables;
insert into categories (name) values ('Electronics');
select * from categories;
```

Now, go to Control Center (http://localhost:9021), Connect menu, upload mysql.properties and launch this connector.

Then you can go to the mysql-server.fullcycle.categories topic, in messages tab, and set the offset field to 0. You will see all messages in that topic. For now, the only one should be the insertion of the 'Electronics' category.

Experiment inserting more data into MySQL:

```sql
insert into categories (name) values ('Kitchen');
select * from categories;
```

![](./images/kafka-connect-mysql-topic.png)

### Configuring MongoDB Connector

Now, go to Control Center (http://localhost:9021), Connect menu, upload mongodb.properties and launch this connector.

Then you can go to Mongo Express (http://localhost:8085), fullcycle database, View button.

![Printscreen of the fullcycle database with a document with properties _id, before, after and source](./images/kafka-connect-mongo-express.png)

Explaining the document properties:

- **_id**: MongoDB generated value
- **before**: Old value of this row in MySQL
- **after**: New value of this row in MySQL
- **source**: Metadata about MySQL, the database and the table of this row

### Learning about Kafka Connect Transformations

As we only want the new value of the MySQL rows, we will extract only the **after** property.

[Documentation of the extract field/value transformation](https://docs.confluent.io/platform/current/connect/transforms/extractfield.html)

Steps to do that:

- Delete fullcycle database in MongoDB
- Delete MongoDB connector in Control Center
- Uncomment the lines in mongodb.properties
- In Control Center, Connect menu, upload mongodb.properties and launch this connector again.
- Insert more data into MySQL

```sql
insert into categories (name) values ('Toys');
select * from categories;
```

![Printscreen of the fullcycle database with a document with properties _id, id and name](./images/kafka-connect-mongo-express-extractfield.png)
