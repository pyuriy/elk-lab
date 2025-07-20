# ELK Stack Introduction and Lab

## Introduction to ELK Stack

The **ELK Stack** is a powerful set of tools for centralized logging, monitoring, and observability:

- **Elasticsearch**: A distributed search and analytics engine that stores and indexes log data for fast retrieval.
- **Logstash**: A data processing pipeline that collects, parses, and transforms logs before sending them to Elasticsearch.
- **Kibana**: A visualization tool that creates dashboards and charts to analyze log data stored in Elasticsearch.

In monitoring and observability, ELK is used to aggregate logs from applications and infrastructure, detect errors, and visualize performance metrics. It supports production support by enabling root cause analysis and integrates with tools like PagerDuty for alerting.

## Quick Lab: Setting Up a Basic ELK Stack Logging Pipeline

This lab sets up a local ELK Stack to collect, process, and visualize sample application logs. It’s designed for a beginner to intermediate user and takes \~30 minutes.

### Prerequisites

- A machine with Docker installed (Windows, macOS, or Linux).
- Basic familiarity with command-line interfaces.
- 4GB RAM and 10GB free disk space.

### Lab Steps

#### 1. **Set Up ELK Stack Using Docker**

Use Docker Compose to run Elasticsearch, Logstash, and Kibana as containers.

1. Create a file named `docker-compose.yml`:

   ```yaml
   version: '3'
   services:
     elasticsearch:
       image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
       environment:
         - discovery.type=single-node
         - xpack.security.enabled=false
       ports:
         - "9200:9200"
       volumes:
         - esdata:/usr/share/elasticsearch/data
     logstash:
       image: docker.elastic.co/logstash/logstash:8.15.0
       volumes:
         - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
       ports:
         - "5044:5044"
       depends_on:
         - elasticsearch
     kibana:
       image: docker.elastic.co/kibana/kibana:8.15.0
       environment:
         - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
       ports:
         - "5601:5601"
       depends_on:
         - elasticsearch
   volumes:
     esdata:
   ```

2. Create a Logstash configuration file named `logstash.conf` in the same directory:

   ```conf
   input {
     file {
       path => "/tmp/app.log"
       start_position => "beginning"
     }
   }
   filter {
     grok {
       match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" }
     }
   }
   output {
     elasticsearch {
       hosts => ["http://elasticsearch:9200"]
       index => "applogs-%{+YYYY.MM.dd}"
     }
     stdout { codec => rubydebug }
   }
   ```

   This configuration reads logs from `/tmp/app.log`, parses them using a grok filter, and sends them to Elasticsearch.

3. Start the ELK Stack:

   ```bash
   docker-compose up -d
   ```

   This launches Elasticsearch (port 9200), Logstash (port 5044), and Kibana (port 5601).

#### 2. **Generate Sample Log Data**

1. Create a sample log file `/tmp/app.log` on your machine:

   ```bash
   echo "2025-07-14T12:00:00Z ERROR Failed to process request" > /tmp/app.log
   echo "2025-07-14T12:01:00Z INFO Request processed successfully" >> /tmp/app.log
   ```
2. Logstash will automatically detect and process these logs.

#### 3. **Access Kibana and Visualize Logs**

1. Open a browser and navigate to `http://localhost:5601`.
2. In Kibana, go to **Management &gt; Stack Management &gt; Index Patterns**.
3. Create an index pattern for `applogs-*` and set `@timestamp` as the time field.
4. Go to **Discover** to view the logs. You should see the parsed log entries with fields like `timestamp`, `level`, and `message`.
5. Create a simple dashboard:
   - Go to **Visualize &gt; Create Visualization**.
   - Choose a **Pie Chart**.
   - Select the `applogs-*` index.
   - Add a bucket for `terms` on the `level` field to visualize the distribution of ERROR vs. INFO logs.
   - Save and add to a new dashboard under **Dashboard &gt; Create Dashboard**.

#### 4. **Test and Explore**

1. Add more log entries to `/tmp/app.log`:

   ```bash
   echo "2025-07-14T12:02:00Z ERROR Database timeout" >> /tmp/app.log
   ```
2. Refresh Kibana’s **Discover** page to see the new log. Check the pie chart for updated ERROR/INFO distribution.
3. Experiment with Kibana queries, e.g., search for `level:ERROR` to filter error logs.

#### 5. **Clean Up**

Stop and remove the containers:

```bash
docker-compose down
```

### Expected Outcome

- Elasticsearch stores and indexes the logs.
- Logstash processes the sample log file and sends parsed data to Elasticsearch.
- Kibana displays logs and a pie chart showing log level distribution.

### Relevance

- **Monitoring and Observability**: This lab demonstrates centralized log collection and visualization, key for monitoring distributed systems.
- **Production Support**: Parsing logs to identify errors (e.g., “ERROR Database timeout”) supports troubleshooting and RCA.
- **AIOps**: While this lab is basic, ELK’s data can feed into AIOps platforms for anomaly detection (e.g., integrating with Dynatrace).

### Next Steps

- Explore advanced Logstash filters (e.g., parsing JSON logs).
- Integrate with Filebeat to collect logs from multiple sources.
- Set up alerts in Kibana for specific error patterns, connectable to PagerDuty.