# DE_playground_hadoop_docker

## Description:
This project provides a ready-to-use solution for setting up a data engineering playground. It is ideal for data engineers who want to test their applications or for beginners who want to learn and experiment in a real-world-like environment.

The project is structured around Docker, and it includes several services such as Apache Hadoop, Apache Hive, Apache Spark, Livy, Hue, and PostgreSQL. All these services are orchestrated using Docker Compose, making it easy to manage the entire stack.

## Components:

- Apache Hadoop: A framework for distributed storage and processing of large datasets. The Hadoop services included are NameNode, DataNode, ResourceManager, and NodeManager.

- Apache Hive: A data warehouse infrastructure built on top of Hadoop for providing data query and analysis.

- Apache Spark: An open-source, distributed computing system used for big data processing and analytics.

- Livy: An open-source REST interface for interacting with Apache Spark from anywhere.

- Hue: A web interface for analyzing data with Apache Hadoop.

- PostgreSQL: An open-source relational database.

- MySQL: Another open-source relational database used here for Hue.

## Configuration:
The configuration files for each service are stored in their respective directories. For example, the configuration for Hadoop is in the hadoop-config directory, and the configuration for Spark is in the spark-config directory. These configuration files are mounted into the Docker containers at runtime.

The __'jars'__ directory contains the necessary Java libraries for the services to function correctly.

The __'logs'__ directory is used by Livy for logging.

The __'mysql_data'__ and __'postgres_data'__ directories are used for storing database data.

The __'spark-events'__ directory is used by Spark for logging events.

The __'test.sh'__ script is a utility script.

The __'warehouse'__ directory is used by Hive as a warehouse directory.

## Ports:

The services are exposed on the following ports:

- Hadoop NameNode: 9870
- Hadoop DataNode: 9864
- Hadoop ResourceManager: 8088 and 8032
- Hadoop NodeManager: 8042
- PostgreSQL: 5432
- Hive Metastore: 9083
- HiveServer2: 10000
- MySQL: 3306
 -Hue: 8888
- Spark Master: 8080 and 7077
- Spark Worker: 8081
- Livy: 8998
- Spark History Server: 18080

## Version Information:

- Livy: 0.7.1 incubating
- Apache Spark: 3.4.0
- Apache Hive: 4.0.0-alpha-2
- Apache Hadoop: 3
- PostgreSQL: latest
- MySQL: 5.7
- Hue: gethue/hue:20230711-140101

This project is a great starting point for anyone looking to dive into data engineering or for experienced engineers who want a convenient and comprehensive playground for testing and development. 
__Please note__ that the project is currently __unable__ to get Livy working correctly with Hue. 
However, all other components are functioning as expected.