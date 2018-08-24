# logspout-logstash-multiline

[![Docker Hub](https://img.shields.io/docker/pulls/bekt/logspout-logstash.svg?maxAge=2592000?style=plastic)](https://hub.docker.com/r/pablolibo/logspout-logstash-multiline/)
[![](https://img.shields.io/docker/automated/pablolibo/logspout-logstash-multiline.svg?maxAge=2592000)](https://hub.docker.com/r/pablolibo/logspout-logstash-multiline/builds/) [![](https://images.microbadger.com/badges/image/pablolibo/logspout-logstash-multiline.svg)](https://microbadger.com/images/pablolibo/logspout-logstash-multiline "Get your own image badge on microbadger.com")


Tiny [Logspout](https://github.com/gliderlabs/logspout) adapter to send Docker container logs to [Logstash](https://github.com/elastic/logstash) via UDP or TCP. This just the hosted working version of [looplab/logspout-logstash](https://github.com/looplab/logspout-logstash).


## Example

A sample `docker-compose.yaml` for swarm file:

```yaml
version: '3.6'
services:
  logspout:
    image: pablolibo/logspout-logstash-multiline
    environment:
      - LOGSPOUT=ignore
      - MULTILINE_ENABLE_DEFAULT=true
      - ROUTE_URIS=logstash://logstash.domain.io:5000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    logging:
      driver: json-file
      options:
        max-size: '12m'
        max-file: '5'
    deploy:
      mode: global
      resources:
        limits:
          cpus: '0.50'
          memory: 256M
        reservations:
          cpus: '0.25'
          memory: 128M
    healthcheck:
      test: "wget -q --spider http://localhost/health"
      interval: 3s
      timeout: 3s
      retries: 6
      start_period: 3s
```


A sample Logstash configuration `logstash/sample.conf`:

```conf
input {
  udp {
    port  => 5000
    codec => json
  }
}


filter {
  if [docker][image] =~ /^logstash/ {
    drop { }
  }
}


output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
  stdout { codec => rubydebug }
}
```
 
