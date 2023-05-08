# Prometheus


## Basics

query .. @ time
 -- Gives you one more more (for each label-combination) SINGLE-INSTANT value
counter {label-combination l1="a", l2="x", l3="0"}
counter {label-combination l1="b", l2="y", l3="0"}

query_range .. @start,end,step
 -- Gives you one more more (for each label-combination) INSTANT-VECTOR value
counter {label-combination l1="a", l2="x", l3="0"}
    (time,value) .. (time,value) ...
counter {label-combination l1="b", l2="y", l3="0"}
    (time,value) .. (time,value) ...

RANGE-VECTOR:
    adding a [timeinterval] to the above -- goives a range vector
counter {label-combination l1="a", l2="x", l3="0"} [$timeinterval]
    - [ (t1,v1),(t2,v2).... (tn,vn) ] , [   ] , [   ] , [   ] , [   ] , [   ]
        |                         |
        |                         |
        +------range values ------+
         (spread over the [..] interval)

SUB-QUERY
Same as above, but you give the resolution for samples in the range vectory
counter {label-combination l1="a", l2="x", l3="0"} [$timeinterval:$resolution]

FUNCTION that collapse a range-vector back to instant-vector:

increase(), rate(), irate() , idelta(), sum_over_time()
    -- are functions that work on each range-value and bring the range back to instant.

AGGREGATION:
sum()
    -- if you have multiple label-combinations, then sum collapses them into one time-series.

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

# Understanding promqa

Simple count for a given duration from a increasing_counter:
```
http://localhost:9090/api/v1/query?query=increase(my_study_counters{type=%22monotonic%22}[5m])&time=1657688400
```
If you need 'N' samples spread across time - eg: attaches count ever 1 hour for the last 24 hours:

```
http://localhost:9090/api/v1/query_range?query=increase(my_study_counters{type=%22monotonic%22}[1h])&start=1657602000&end=1657688400&step=3600
http://localhost:9090/api/v1/query_range?query=increase(my_study_counters{type="monotonic"}[1h])&start=1657602000&end=1657688400&step=3600
```



# Links

https://devopscube.com/install-configure-prometheus-linux/
https://medium.com/tlvince/prometheus-backfilling-a92573eb712c
https://stackoverflow.com/questions/47138461/get-total-requests-in-a-period-of-time

