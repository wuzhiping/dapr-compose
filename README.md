# dapr-compose
https://dapr.io/

<pre>
  app01:
    image: shawoo/openresty
    restart: always
    working_dir: /usr/local/openresty/
    volumes:
      - ./app01/nginx.conf:/usr/local/openresty/nginx/conf/nginx.conf
      - ./app01/conf.d:/etc/nginx/conf.d
      - ./app01/html:/usr/local/openresty/nginx/html
      - ./app01/lua:/usr/local/openresty/nginx/lua
    depends_on:
      - redisdb
      - placement
      - zipkin
    ports:
      - "3501:3500"
  app01-dapr:
    image: "daprio/daprd:edge"
    command: ["./daprd",
     "-app-id", "app01",
     "-app-port", "80",
     "-placement-host-address", "placement:50005",
     "-components-path", "/dapr/components",
     "-config", "/dapr/config.yaml"]
    volumes:
        - "./dapr/:/dapr/"
    depends_on:
      - app01
    network_mode: "service:app01"

  placement:
    image: "daprio/dapr"
    command: ["./placement", "-port", "50005"]
    ports:
      - "50005:50005"
  zipkin:
    image: "openzipkin/zipkin"
    ports:
      - "9411:9411"
  redisdb:
    image: redis
    restart: always
    #ports:
    #  - "6379:6379"
    volumes:
      - ./redis.conf:/etc/redis/redis.conf
    command: redis-server /etc/redis/redis.conf

</pre>
