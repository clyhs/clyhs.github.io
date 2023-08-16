title: superset部署
date: 2023-08-11 22:45:25
category: BI
tags: superset

# superset

```text
docker pull apache/superset

docker run -d -p 8080:8088 --name superset apache/superset

docker exec -it superset superset fab create-admin \
              --username admin \
              --firstname Superset \
              --lastname Admin \
              --email admin@superset.com \
              --password admin

docker exec -it superset superset db upgrade

docker exec -it superset superset load_examples

docker exec -it superset superset init
```