---
title: "Monitoring Docker Containers with cAdvisor, Prometheus, and Grafana Using Docker Compose"
source: "https://medium.com/@varunjain2108/monitoring-docker-containers-with-cadvisor-prometheus-and-grafana-d101b4dbbc84"
author:
  - "[[Varun Jain]]"
published: 2023-12-28
created: 2026-04-23
description: "Introduction"
tags:
  - "clippings cadvisor"
---
## Introduction

This article introduces a trio — cAdvisor for real-time container metrics, Prometheus for data aggregation and storage, and Grafana for insightful visualization. We’ll guide you through setting up a monitoring system for your Docker containers using Docker Compose.

### Understanding the Tools

- [cAdvisor](https://github.com/google/cadvisor): cAdvisor (Container Advisor) is an open-source tool by Google that provides container users with an understanding of the resource usage and performance characteristics of their running containers, offering detailed real-time metrics about CPU, memory, file and network usage.
- [Prometheus](https://github.com/prometheus/prometheus): Prometheus is an open-source monitoring and alerting toolkit that collects and stores its metrics as time series data, allowing for powerful querying, alerting, and visualization of metrics from various sources including Docker containers.
- [Grafana](https://github.com/grafana/grafana): Grafana is a highly versatile open-source analytics and monitoring solution that allows you to query, visualize, alert on, and understand your metrics no matter where they are stored, making it an ideal tool for creating insightful and interactive dashboards based on data aggregated by Prometheus.

Note: For the following setup I am using Mac M1

Let’s start with cAdvisor ***docker-compose.yaml***

```c
version: '3'

services:
  cadvisor:
      image: gcr.io/cadvisor/cadvisor:v0.47.1
      hostname: cadvisor
      platform: linux/aarch64
      volumes:
        - "/:/rootfs:ro"
        - "/var/run:/var/run:ro"
        - "/sys:/sys:ro"
        - "/var/lib/docker/:/var/lib/docker:ro"
        - "/dev/disk/:/dev/disk:ro"
      ports:
        - "8080:8080"
```

Now, run the following command

> **docker compose -f docker-compose.yaml up**

Open the cAdvisot UI in your browser: [HTTP://0.0.0.0:8080](http://http//0.0.0.0:8080)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*BrrPFQdaJ1yzF5l_NXSoqA.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*rckjqNuRAkC033C7q8AylQ.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*9jzZwmxirm1xexAR4-nbbw.png)

Whenever cAdvisor detects the new container running on the host machine, it will automatically start capturing the metrics of the new container.

cAdvisor itself does not have built-in storage for the metrics it collects. Instead, cAdvisor is primarily used for real-time monitoring of container performance, gathering detailed information about resource usage (like CPU, memory, network, and storage) for running containers.

Metrics exposed by the cAdvisor: [HTTP://0.0.0.0:8080/metrics](http://http//0.0.0.0:8080/metrics)

While configuring the Prometheus we will use the above /metrics endpoint for the scraping purpose.

### Setup Prometheus

To store the metrics collected by cAdvisor over time, you would typically integrate it with a time-series database like Prometheus. Prometheus can scrape and store the data provided by cAdvisor, allowing for historical data analysis, querying, and alerting. This integration is a common setup for container monitoring, where cAdvisor collects the metrics, and Prometheus stores them.

Let’s set up the Prometheus using the same docker-compose.yaml

Update docker-compose.yaml with Prometheus

```c
version: '3'

services:
  cadvisor:
      image: gcr.io/cadvisor/cadvisor:v0.47.1
      hostname: cadvisor
      platform: linux/aarch64
      volumes:
        - "/:/rootfs:ro"
        - "/var/run:/var/run:ro"
        - "/sys:/sys:ro"
        - "/var/lib/docker/:/var/lib/docker:ro"
        - "/dev/disk/:/dev/disk:ro"
      ports:
        - "8080:8080"
  
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
```

> Run the **docker compose -f docker-compose.yaml up** again**.**

Open the Prometheus UI: [HTTP://0.0.0.0:9090](http://http//0.0.0.0:9090)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*gjmOpzYjTDa4XqSUa5NkqA.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*mJ1vLtAjJUh-D4S8DmgD3Q.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*-rlc4uMqiKdzKOVd2Asl5g.png)

Next step: Configure the Prometheus to scrape metrics exposed by the cAdvisor.

Create the prometheus-scrape-config.yaml like this in the same directory as docker-compose.yaml file.

```c
global:
  scrape_interval:     15s
scrape_configs:
  - job_name: 'cadvisor'
    scrape_interval: 5s
    static_configs:
      - targets: ['cadvisor:8080']
```

This scrape config file will pull the metrics from the advisor docker container exposed through port ***8080***, endpoint ***/metrics*** every ***5s***.

## Get Varun Jain’s stories in your inbox

Join Medium for free to get updates from this writer.

Update the ***docker-compose.yaml*** file. Mount the ***prometheus-scrape-config.yaml*** using the docker volume like this

```c
version: '3'

services:
  cadvisor:
      image: gcr.io/cadvisor/cadvisor:v0.47.1
      hostname: cadvisor
      platform: linux/aarch64
      volumes:
        - "/:/rootfs:ro"
        - "/var/run:/var/run:ro"
        - "/sys:/sys:ro"
        - "/var/lib/docker/:/var/lib/docker:ro"
        - "/dev/disk/:/dev/disk:ro"
      ports:
        - "8080:8080"
  
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus-scrape-config.yaml
    ports:
      - "9090:9090"
```

Restart the dockers services again and Open the Prometheus UI: [HTTP://0.0.0.0:9090](http://http//0.0.0.0:9090). Now we should be able to see cAdvisor in the target and service discovery section of the Prometheus UI. Refer to the images below

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*m4i09F9_iMwKjumZOlDoOQ.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*51on41dgTYoWbbf22aoL1Q.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*Zb2j2XaDdKRhpGO_-d_aog.png)

Go, and play with the Prometheus metrics query explorer.

Click here for [Sample Metrics Link](http://0.0.0.0:9090/graph?g0.expr=container_memory_usage_bytes&g0.tab=0&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h)

### What we have achieved so far?

- cAdvisor is running locally and capturing all the real-time metrics using the docker-compose.
- Prometheus is running locally and configured to scrape metrics from the cAdvisor /metrics endpoint and dump them into the Prometheus Time Series DB.

### Setup Grafana

Although Prometheus excels in data aggregation and storage, it offers limited visualization features. For advanced visualization, we’ll use Grafana, renowned for its comprehensive and interactive dashboards. Grafana transforms Prometheus metrics into detailed, easily interpretable visual formats, enhancing our monitoring capabilities.

Update the docker-compose.yaml file to add grafan container

```c
version: '3'

services:
  cadvisor:
      image: gcr.io/cadvisor/cadvisor:v0.47.1
      hostname: cadvisor
      platform: linux/aarch64
      volumes:
        - "/:/rootfs:ro"
        - "/var/run:/var/run:ro"
        - "/sys:/sys:ro"
        - "/var/lib/docker/:/var/lib/docker:ro"
        - "/dev/disk/:/dev/disk:ro"
      ports:
        - "8080:8080"
  
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus-scrape-config.yaml
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana
    environment: 
      GF_SECURITY_DISABLE_INITIAL_ADMIN_CREATION: "true"
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_ANONYMOUS_ORG_ROLE: "Admin"
      GF_AUTH_DISABLE_SIGNOUT_MENU: "true"
      GF_AUTH_DISABLE_LOGIN_FORM: "true"
    ports:
      - "9100:3000"
```

> Run the **docker compose -f docker-compose.yaml up** again**.**

Open the Grafana UI: [HTTP://0.0.0.0:9100](http://http//0.0.0.0:9100)

Now, we need to configure the Datasource as Prometheus in the Grafana.

Prometheus is our metrics storage layer and Grafana is the visualization tool. Grafana can configured with different types of data sources like Postgres, AWS Cloudwatch, Datadog, ElasticSearch, etc. Feel free to explore the Data source here: [***http://0.0.0.0:9100/connections/add-new-connection***](http://0.0.0.0:9100/connections/add-new-connection)

For integrating Prometheus with Grafana, we have two options: manually adding the Prometheus data source through Grafana’s UI, or automating the setup using a `grafana-datasource-config.yaml` file. This file can be configured to automatically establish the Prometheus data source when executing `docker-compose up`, streamlining the integration process.

Create the ***grafana-datasource-config.yaml*** in the same directory as docker-compose.yaml

```c
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

Now, we have to tell the Grafana docker service to use this file during the service boot-up.

Final updated docker-compose.yaml file

```c
version: '3'

services:
  cadvisor:
      image: gcr.io/cadvisor/cadvisor:v0.47.1
      hostname: cadvisor
      platform: linux/aarch64
      volumes:
        - "/:/rootfs:ro"
        - "/var/run:/var/run:ro"
        - "/sys:/sys:ro"
        - "/var/lib/docker/:/var/lib/docker:ro"
        - "/dev/disk/:/dev/disk:ro"
      ports:
        - "8080:8080"
  
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus-scrape-config.yaml
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana
    environment: 
      GF_SECURITY_DISABLE_INITIAL_ADMIN_CREATION: "true"
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_ANONYMOUS_ORG_ROLE: "Admin"
      GF_AUTH_DISABLE_SIGNOUT_MENU: "true"
      GF_AUTH_DISABLE_LOGIN_FORM: "true"
    volumes:
      - ./datasource.yaml:/etc/grafana/provisioning/datasources/datasource.yaml
    ports:
      - "9100:3000"
```

> Run the **docker compose -f docker-compose.yaml up** again**.**

We should be able to see the Prometheus DS here [http://0.0.0.0:9100/connections/datasources](http://0.0.0.0:9100/connections/datasources)

### Final Step

Let’s create the Dashboard on the Grafana.

- Open: [http://0.0.0.0:9100/dashboard/import](http://0.0.0.0:9100/dashboard/import)
- Load the Grafana Dashboard ID = 193, taken from here: [https://grafana.com/grafana/dashboards/193-docker-monitoring/](https://grafana.com/grafana/dashboards/193-docker-monitoring/)
- Select the DS as Prometheus. Hit Save.
- Voila, we have the Grafana dashboard showing the real-time metrics of the containers.
- Refer, to the images below.

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*CAFJC92lVMp_gQR9XTGF1Q.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*1DwfqYPtsUKADa64j5MvYA.png)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*qOzUMIJeW1zO9vgdliFswQ.png)