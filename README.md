# Confluent platform monitoring stack


Provides a monitoring stack for CP Ansible clusters based on Prometheus and Grafana, runs on Docker.


## tl;dr


```sh
docker-compose up
```

## Requirements

### Host setup

* [Docker Engine][docker-install] version **18.06.0** or newer
* [Docker Compose][compose-install] version **1.28.0** or newer (including [Compose V2][compose-v2])
* 1.5 GB of RAM

> [!NOTE]
> Especially on Linux, make sure your user has the [required permissions][linux-postinstall] to interact with the Docker
> daemon.

By default, the stack exposes the following ports:

* 9090: Prometheus web service
* 3000: Grafana portal


### Clusters to be monitored

All the CP Ansible clusters need to be deployed with the following option turned on in the playbook configuration.

```yaml
all:
  vars:
     # ...
     jmxexporter_enabled: true
```


## Usage

Open the grafana dashboard by visiting `http://<host-ip>:3000` in the browser.


* user: *admin*
* password: *pfsecret*


## Initial setup

Configure `prometheus/prometheus.yml` and edit the following Yaml,

```yaml
scrape_configs:
  - job_name: 'kafka-broker'
    static_configs:
      - targets: ['139.59.61.11:8080','139.59.61.83:8080'] # <---- (1)
        labels:
          env: "dev" # <---- (2)
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        regex: '([^:]+)(:[0-9]+)?'
        replacement: '${1}'
```

1. The hosts/IPs of all the Kafka brokers you want to monitor. The JMX port for broker is `8080`.
2. The custom label you want to give for each cluster. Configure it to be the same for all the components of a cluster.

Add another scrape config entry for Zookeeper.

```yaml
  - job_name: 'zookeeper'
    static_configs:
      - targets: ['139.59.60.174:8079'] # <---- (1)
        labels:
          env: "dev" # <---- (2)
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        regex: '([^:]+)(:[0-9]+)?'
        replacement: '${1}'
```

1. The Zookeeper JMX port is `8079`.
2. This zookeeper is part of the dev cluster. So we add the same label `dev` as that of the broker.

Likewise, add any other component which needs to be monitored(schema registry, connect cluster, ksqldb).

Here's the JMX port for each component.

| Service         | JMX port  |
|-----------------|-----------|
| Kafka broker    | 8080      |
| Zookeeper       | 8079      |
| Schema registry | 8078      |
| Connect cluster | 8077      |
| Ksqldb          | 8076      |


### Dashboards

This setup is going to provide the following dashboards:


#### Confluent platform overview

#### Kafka cluster

#### Zookeeper cluster

#### Kafka consumers

#### Kafka producers

#### Kafka lag exporter

#### Kafka topics

#### Ksqldb cluster

#### Schema registry

#### Connect cluster

# TODO

- Add screenshots

