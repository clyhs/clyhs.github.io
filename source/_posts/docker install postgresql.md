---
title: docker安装postgresql
date: 2019-01-10 09:22:24
category: docker
tags: postgresql
---

# docker install postgresql

## image

```
docker pull postgres
```

## build container

```
docker run -it --name postgres --restart always -r POSTGRES_PASSWORD='123456' -e ALLOW_IP_RANGE=0.0.0.0/0 -V /home/postgres/data:/var/lib/postgresql -p 5432:5432 -d postgres
```

## enter container

```
docker exec -it postgres bash
```

## login postgres

change postgres

```
su postgres
psql -U postgres -W
password:
psql (12.2....)
Type "help" for help.
postgres=#
```

## setting remote access

change file: pg_hba.conf   postgresql.conf

### 1、change pg_hba.conf

copy pg_hba.conf to /home

```
docker cp postgres:/var/lib/postgresql/data/pg_hba.conf /home
```

example

```
local all all         trust
host all all 127.0.0.1/32 trust
*host all all 0.0.0.1/0 md5*
host all all ::1/128 trust
```

next copy to container change

```
docker cp /home/pg_hba.conf postgres:/var/lib/postgresql/data
```

### 2、change postgresql.conf

change listen_addresses

```
listen_addresses='*'
```

exit container

```
exit
```

## shutdown firewalld

```
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld
```

## download pgAdmin client

 ...