---
title: monitoring
date: 2020-02-26 21:52:07
tags:
---
# Frontend Monitoring
## Two approaches to frontend monitoring
- real user monitoring (RUM)
- synthetic
  - tools: webpagetest.org
**Browsers expose page performance metrics visa the Navigation Timing API**

# Application Monitoring
**Application Performance Monitoring(APM) Tools**
- StatsD is a tool used to add metrics inside of your code.

# Telegraf, InfluxDB + Grafana
- Grafana: Grafana is "The open platform for beautiful analytics and monitoring." It makes it easy to create dashboards for displaying data from many sources, particularly time-series data. It works with several different data sources such as Graphite, Elasticsearch, InfluxDB, and OpenTSDB. We’re going to use this as our main front end for visualizing our network statistics.

- InfluxDB: InfluxDB is "…a data store for any use case involving large amounts of timestamped data." This is where we’re going to store our network statistics. It is designed for exactly this use-case, where metrics are collected over time.

- Telegraf: Telegraf is "…a plugin-driven server agent for collecting and reporting metrics." This can collect data from a wide variety of sources, e.g. system statistics, API calls, DB queries, and SNMP. It can then send those metrics to a variety of datastores, e.g. Graphite, OpenTSDB, Datadog, Librato. Telegraf is maintained by InfluxData, the people behind InfluxDB. So it has very good support for writing data to InfluxDB.
