## How to run PromQL compliance tests

1. Launch `zincobserve` (note that it listens at port 5080 by default).

2. Create Prometheus configuration file (e.g., `/tmp/prometheus.yml`).

```yaml
global:
  scrape_interval: 5s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'local_metrics'
    static_configs:
      - targets: ['host.docker.internal:5080']

remote_write:
  - url: "http://host.docker.internal:5080/api/default/prometheus/api/v1/write"
    basic_auth:
      username: "root@example.com"
      password: "Complexpass#123"
```

This configuration will make `prometheus` scrape metrics from `zincobserve`'s `/metrics` endpoint and remote-write them back to `zincobserve`.

3. Launch `prometheus`. Make sure to specify correct path to `prometheus.yml`.

```shell
docker run --rm -it -p 9090:9090 -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml:ro prom/prometheus:latest
```

4. Wait some time for metrics data to be ingested into both `prometheus` and `zincobserve`. "Some time" lies somewhere between several minutes and [several hours].

[several hours]: https://github.com/prometheus/compliance/blob/12cbdf92abf7737531871ab7620a2de965fc5382/promql/promql-test-queries.yml#L1-L3

5. Run `promql-compliance-tester`

```shell
 docker run --rm -it -v $PWD/zincobserve:/d:ro promql-compliance-tester:latest \
     --config-file /d/promql-test-queries.yml \
     --config-file /d/test-zincobserve-docker.yml
```
