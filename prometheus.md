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

## commands

```
cfg_file=/home/lakshman/host_c/Data/work/prometheus_try/prometheus.yml
data_dir=/mnt/other_disk/prometheus
docker run -p 9090:9090 -v $cfg_file:/etc/prometheus/prometheus.yml -v $data_dir:/prometheus prom/prometheus

dirclean() {
    data_dir=/mnt/other_disk/prometheus
    sudo rm -rf $data_dir
    mkdir -p $data_dir
    chmod 777 $data_dir
}

exdc() {
    docker ps -a | awk '/Exited/ { print $1 } ' | xargs -n 1 -r docker rm
    cont_name=$(docker ps | awk '/prom\/prometheus/ {print $1}')
    docker exec -it $cont_name /bin/sh
}

stdc () {
    docker ps -a | awk '/Exited/ { print $1 } ' | xargs -n 1 -r docker rm
    docker run -p 9090:9090 -v $cfg_file:/etc/prometheus/prometheus.yml -v $data_dir:/prometheus prom/prometheus
}

promtool tsdb create-blocks-from openmetrics values ./

echo $data_dir
bash dump_data.sh > $data_dir/values

```

# Links

https://devopscube.com/install-configure-prometheus-linux/
https://medium.com/tlvince/prometheus-backfilling-a92573eb712c
