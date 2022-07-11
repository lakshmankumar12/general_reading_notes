# Prometheus

# Run a prometheus docker

## Bare minimal config

```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

## command

```
cfg_file=/home/lakshman/host_c/Data/work/prometheus_try/prometheus.yml
data_dir=/mnt/other_disk/prometheus
docker run -p 9090:9090 -v $cfg_file:/etc/prometheus/prometheus.yml -v $data_dir:/prometheus prom/prometheus

```



# Links

https://devopscube.com/install-configure-prometheus-linux/
https://medium.com/tlvince/prometheus-backfilling-a92573eb712c
