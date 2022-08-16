# dapr-compose
https://dapr.io/

# auth for api gateway
## sudo htpasswd -c htpasswd uid
### header Authorization = Basic dWlkOnB3ZA==

<pre>
    location /v1.0/ {
        auth_basic           "DAPR Service Area ";
        auth_basic_user_file conf/htpasswd;
        # ...
        # ...
        deny  192.168.1.2;
        allow 192.168.1.1/24;
        allow 127.0.0.1;
        deny  all;
    }
    location /public/ {
       auth_basic off;
    }
</pre>

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

# https://blog.hypriot.com/
<pre>
REPOSITORY                  TAG                   IMAGE ID            CREATED             SIZE
daprio/daprd                edge-linux-arm        85c7c18ec35b        2 days ago          110MB
daprio/dapr                 1.8.4-linux-arm       6e73a117ed73        3 days ago          225MB

postgres                    latest                3562ae2df2fb        3 days ago          312MB
postgrest/postgrest         v9.0.0                c132f4ce0634        8 months ago        281MB

hypriot/rpi-redis           latest                c06243ca8f1b        6 years ago         117MB
pabloromeo/zipkin-arm       zipkin-slim-arm32v7   f8c50f0d07bf        2 years ago         716MB

hypriot/rpi-consul          latest                879ac05d5353        6 years ago         19.7MB
hypriot/rpi-mysql           latest                4f3cbdbc3bdb        4 years ago         209MB
hypriot/rpi-node            latest                c7a24b17fa2e        4 years ago         494MB
hypriot/rpi-busybox-httpd   latest                fbd9685c5ffc        7 years ago         2.16MB
</pre>
<pre>
version: '2'
services:
  placement:
    image: "daprio/dapr:1.8.4-linux-arm"
    command: ["./placement", "-port", "50005"]
    ports:
      - "50005:50005"

  redisdb:
    image: hypriot/rpi-redis
    restart: always

  app01:
    image: hypriot/rpi-busybox-httpd
    restart: always
    depends_on:
      - redisdb
      - placement
    ports:
      - "81:80"
      - "3501:3500"
      - "50010:50000"
  app01-dapr:
    image: "daprio/daprd:edge-linux-arm"
    command: ["./daprd",
     "-app-id", "app01",
     "-app-port", "80",
     "-dapr-grpc-port", "50000",
     "-placement-host-address", "placement:50005",
     "-components-path", "/dapr/components",
     "-config", "/dapr/config.yaml"]
    volumes:
        - "./dapr/:/dapr/"
    depends_on:
      - app01
    network_mode: "service:app01"

  app02:
    image: hypriot/rpi-busybox-httpd
    restart: always
    depends_on:
      - redisdb
      - placement
    ports:
      - "82:80"
      - "3502:3500"
      - "50020:50000"
  app02-dapr:
    image: "daprio/daprd:edge-linux-arm"
    command: ["./daprd",
     "-app-id", "app02",
     "-app-port", "80",
     "-dapr-grpc-port", "50000",
     "-placement-host-address", "placement:50005",
     "-components-path", "/dapr/components",
     "-config", "/dapr/config.yaml"]
    volumes:
        - "./dapr/:/dapr/"
    depends_on:
      - app02
    network_mode: "service:app02"
</pre>
