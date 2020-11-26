---
title: Big Data Realtime Project
subtitle: GULI MALL REALTIME USER BEHAVIOR ANALYSIS PROJECT
date: 2020-10-01T12:45:00.000Z
draft: false
featured: false
external_link: https://github.com/lixin940207/gmail1205parent
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---
Project Structure: nginx+Springboot+Kafka+SparkStraming+HBase+Mysql+Canal+Elasticsearch

Collected real-time user behavior data and sent it to **Kafka**.

Calculated daily and timesharing DAU and transaction amount using **SparkStreaming** and saved data to **Hbase**.

Joined data streams to merge order information and user together, then sent to **ElasticSearch**for further query.

Analyzed user purchase behavior from **Kibana**.