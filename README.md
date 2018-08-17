## swaggerhub地址

- https://app.swaggerhub.com/apis/ccthiot/NGSI/1.0.0

## Fiware-Orion

- https://fiware-orion.readthedocs.io/en/latest/index.html

## smartsdk/ngsi-timeseries-api: QuantumLeap: a Timeseries NGSI backend
- https://github.com/smartsdk/ngsi-timeseries-api

### docker-compose-dev.yml
```yaml
version: '3'

services:

  orion:
    image: fiware/orion:1.13.0
    ports:
      - "11026:1026"
    command: -logLevel DEBUG -noCache -dbhost mongo
    depends_on:
      - mongo
    healthcheck:
      test: ["CMD", "curl", "-f", "http://0.0.0.0:1026/version"]
      interval: 1m
      timeout: 10s
      retries: 3

  mongo:
    image: mongo:3.2
    ports:
      - "27018:27017"
    volumes:
      - mongodata:/data/db

  quantumleap:
    image: ${QL_IMAGE:-smartsdk/quantumleap}
    ports:
      - "8668:8668"
    depends_on:
      - mongo
      - orion
      - crate
    environment:
      - CRATE_HOST=${CRATE_HOST:-crate}
      - USE_GEOCODING=True
      - REDIS_HOST=redis
      - REDIS_PORT=6379

  crate:
    image: crate:1.0.5
    ports:
      # Admin UI
      - "4200:4200"
      # Transport protocol
      - "4300:4300"
    command: -Ccluster.name=democluster -Chttp.cors.enabled=true -Chttp.cors.allow-origin="*"
    volumes:
      - cratedata:/data

  grafana:
    image: grafana/grafana
    ports:
      - "4000:3000"
    environment:
      - GF_INSTALL_PLUGINS=crate-datasource,grafana-clock-panel,grafana-worldmap-panel
    depends_on:
      - crate

  redis:
    image: redis
    deploy:
      # Scaling Redis requires some extra work.
      # See https://get-reddie.com/blog/redis4-cluster-docker-compose/
      replicas: 1
    ports:
      - "7379:6379"
    volumes:
      - redisdata:/data

volumes:
  mongodata:
  cratedata:
  redisdata:

networks:
    ngsi:
        driver_opts:
            com.docker.network.driver.mtu: ${DOCKER_MTU:-1400}

```


### 使用postman测试API的范例

- https://github.com/ten2net/ccthAPI/blob/master/NGSI.postman_collection.json

> 下载[postman](https://www.getpostman.com/apps)，打开postman工具，文件/导入……

