# Reids Exporter with Cluster settings

Recently start to use redis exporter as monitoring tools.

## Tools

[Redis Exporter](https://github.com/oliver006/redis_exporter)


## Configuration

### Setup Cluster

3 nodes and 3 slaves

we suppose your cluster password is `1234567`
```bash
redis-cli --askpass --cluster create 192.168.1.3:17001 192.168.1.4:17001 192.168.1.5:17001 192.168.1.6:17001 192.168.1.2:5 --cluster-replicas 1
redis-cli --askpass -h 192.168.1.3 -p 17001
192.168.1.3:17001> cluster nodes
```

### Install redis exporter

```bash
cd /usr/local/
tar -xvf redis_exporter-v1.24.0.linux-amd64.tar.gz

```

### Create Redis exporter password file 
You can refer with [this](https://github.com/oliver006/redis_exporter/blob/master/contrib/sample-pwd-file.json)

```bash
cat /etc/redis/pwd.json
{
    "redis://192.168.1.2:17001": "1234567"
}
```

Start exporter
```bash
 /usr/local/redis_exporter-v1.24.0.linux-amd64/redis_exporter --redis.addr="redis://192.168.1.2:17001" --redis.password-file=/etc/redis/pwd.json
```

### Configure exporter on promethues

```json
{
    "job_name": "redis_exporter_targets",
    "metrics_path": "/metrics",
    "scrape_interval": "15s",
    "scrape_timeout": "10s",
    "relabel_configs_attributes": [
        {
            "name": "redis",
            "source_labels_attributes": [
                {
                    "source_label": "__address__"
                }
            ],
            "target_label": "__param_target",
            "separator": ";",
            "regex": "(.*)",
            "replacement": "$1",
            "action": "replace"
        },
        {
            "name": "redis",
            "source_labels_attributes": [
                {
                    "source_label": "__param_target"
                }
            ],
            "target_label": "instance",
            "separator": ";",
            "regex": "(.*)",
            "replacement": "$1",
            "action": "replace"
        },
        {
            "name": "redis",
            "source_labels_attributes": [
                {
                    "source_label": "instance"
                }
            ],
            "target_label": "__address__",
            "separator": ";",
            "regex": "(.*)",
            "replacement": "192.168.1.2:9121",
            "action": "replace"
        }
    ],
    "k8s_sd_configs_attributes": [],
    "consul_sd_configs_attributes": [],
    "static_configs_attributes": [
        {
            "name": "redis",
            "labels_attributes": [],
            "static_targets_attributes": [
                {
                    "target": "redis://192.168.1.2:17001"
                }
            ]
        },
        {
            "name": "redis",
            "labels_attributes": [],
            "static_targets_attributes": [
                {
                    "target": "redis://192.168.1.1:17001"
                }
            ]
        },
        {
            "name": "redis",
            "labels_attributes": [],
            "static_targets_attributes": [
                {
                    "target": "redis://192.168.1.3:17001"
                }
            ]
        },
        {
            "name": "redis",
            "labels_attributes": [],
            "static_targets_attributes": [
                {
                    "target": "redis://192.168.1.4:17001"
                }
            ]
        },
        {
            "name": "redis",
            "labels_attributes": [],
            "static_targets_attributes": [
                {
                    "target": "redis://192.168.1.5:17001"
                }
            ]
        },
        {
            "name": "redis",
            "labels_attributes": [],
            "static_targets_attributes": [
                {
                    "target": "redis://192.168.1.6:17001"
                }
            ]
        }
    ]
}
```

```yaml
scrape_configs:
  ## config for the multiple Redis targets that the exporter will scrape
  - job_name: 'redis_exporter_targets'
    static_configs:
      - targets:
        - redis://192.168.1.1:17001
        - redis://192.168.1.2:17001
        - redis://192.168.1.3:17001
        - redis://192.168.1.4:17001
        - redis://192.168.1.5:17001
        - redis://192.168.1.6:17001
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.1.1:9121

  ## config for scraping the exporter itself
  - job_name: 'redis_exporter'
    static_configs:
      - targets:
        - 192.168.1.19121
```
